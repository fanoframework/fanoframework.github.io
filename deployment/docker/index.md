---
title: Deploy Fano application inside Docker container
description: Tutorial on how to deploy Fano application with Docker container.
---

<h1 class="major">Deploy Fano application inside Docker container</h1>

## Prerequisite

You have working Docker and Docker Compose installation.

It is assumed your current user is in group `docker`. If you are not in `docker` group, you will have to prefix all docker command with `sudo` or run it with elevated privilege (root).

## Deploy Fano CGI Application with Docker

### Create Dockerfile
Create file name `Dockerfile` in project root directory with content as shown below

```
FROM httpd:2.4
```

This tells Docker to create container image based Apache 2.4. This is Debian-based image with common Apache modules are installed.

Now append lines below. We make sure that `mod_cgi`, `mod_cgid` and `mod_rewrite` modules are loaded by modifying main Apache configuration like so.

```
RUN sed -i \
        -e 's/^#\(LoadModule .*mod_cgi.so\)/\1/' \
        -e 's/^#\(LoadModule .*mod_cgid.so\)/\1/' \
        -e 's/^#\(LoadModule .*mod_rewrite.so\)/\1/' \
        -e 's/^#\(Include .*httpd-vhosts.conf\)/\1/' \
        -e 's/^#ServerName.*/ServerName fano-app/' \
        /usr/local/apache2/conf/httpd.conf
```

In case unclear to you, regular expressions replace `#LoadModule mod_cgi` to become `LoadModule mod_cgi` effectively tells Apache  to load `mod_cgi`.

We also tells Apache to include any additional configurations in `httpd-vhosts.conf` file and set `ServerName` configuration.


Append line below to copy `vhost.example` file to `httpd-vhosts.conf` inside container. We will create `vhost.example` latter.

```
COPY ./vhost.example /usr/local/apache2/conf/extra/httpd-vhosts.conf
```

Remove `index.html` as we do not need it, but this is optional step.
```
RUN rm /usr/local/apache2/htdocs/index.html
```

Now add below it
```
COPY ./config/ /usr/local/apache2/config
COPY ./resources/ /usr/local/apache2/resources
COPY ./storages/ /usr/local/apache2/storages
COPY ./public/ /usr/local/apache2/htdocs/
COPY ./public/app.cgi /usr/local/apache2/htdocs/app.cgi

```
Which copy all runtime files needed by application such as configuration, HTML templates, CSS and JavaScript files.

So final Dockerfile will be

```
FROM httpd:2.4

RUN sed -i \
        -e 's/^#\(LoadModule .*mod_cgi.so\)/\1/' \
        -e 's/^#\(LoadModule .*mod_cgid.so\)/\1/' \
        -e 's/^#\(LoadModule .*mod_rewrite.so\)/\1/' \
        -e 's/^#\(Include .*httpd-vhosts.conf\)/\1/' \
        -e 's/^#ServerName.*/ServerName fano-app/' \
        /usr/local/apache2/conf/httpd.conf

COPY ./vhost.example /usr/local/apache2/conf/extra/httpd-vhosts.conf

RUN rm /usr/local/apache2/htdocs/index.html

COPY ./config/ /usr/local/apache2/config
COPY ./resources/ /usr/local/apache2/resources
COPY ./storages/ /usr/local/apache2/storages
COPY ./public/ /usr/local/apache2/htdocs/
COPY ./public/app.cgi /usr/local/apache2/htdocs/app.cgi
```

### Create virtual host configuration

Create a file name `vhost.example` in same directory as `Dockerfile` with content like so

```
<VirtualHost *:80>
    <Directory "/usr/local/apache2/htdocs">
        Options +ExecCGI
        AllowOverride FileInfo Indexes
        Require all granted
        DirectoryIndex app.cgi
        AddHandler cgi-script .cgi
        <IfModule mod_rewrite.c>
            RewriteEngine On

            RewriteCond %{REQUEST_FILENAME} !-d
            RewriteCond %{REQUEST_FILENAME} !-f
            RewriteRule ^(.*)$ app.cgi [L]
        </IfModule>
    </Directory>
</VirtualHost>
```

### Run application in Docker container

Run `build.sh` to compile application first and then from inside directory where Dockerfile resides, run

```
$ docker build -t fano-app .
```

This will build docker image with tag `fano-app` and may take a while.

After that run

```
$ docker run fano-app
```

Open Internet browser and go to `http://172.17.0.2` to access application.

### Run with docker-compose

We can take it further by using `docker-compose` to run application. Create new file name `docker-compose.yaml` in same directory as `Dockerfile`.

```
version: "2"
services:
    fano:
        build: .
```

Now we can run with

```
$ docker-compose up
```
To stop the application, run
```
$ docker-compose down
```

## Deploy Fano FastCGI Application with Docker

### Create Dockerfile

If you create FastCGI project using `--project-fcgid`

```
$ fanocli --project-fcgid=testfano
```

To deploy using Docker, we need to install `mod_fcgid` manually. This module is not installed by default in `httpd:2.4` image.

```
FROM:home:24

#-----------------------------------
# Install mod_fcgid
#-----------------------------------
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    libapache2-mod-fcgid
```
Following `mod_fcgid` installation, we tells Apache to load module.

```
#-----------------------------------
# Enable mod_fcgid mod_rewrite and
# additional virtual host config
#-----------------------------------
RUN sed -i \
        -e 's/^#\(LoadModule .*mod_rewrite.so\)/\1/' \
        -e 's/^#\(Include .*httpd-vhosts.conf\)/\1/' \
        -e '/LoadModule .*mod_rewrite.so/a LoadModule fcgid_module /usr/lib/apache2/modules/mod_fcgid.so' \
        -e 's/^#ServerName.*/ServerName fano-app/' \
        /usr/local/apache2/conf/httpd.conf

```

The rest of Dockerfile is same as Dockerfile for CGI above.

## Deploy Fano FastCGI Application with Docker

### Create Dockerfile
For FastCGI that runs as separate server via reverse-proxy module, i.e, using `--project-fcgi`, you can run Fano application and Apache as separate docker container and run it with `docker-compose`.

```
$ fanocli --project-fcgi=testfano
```

For that we need to create two Dockerfiles for Fano application and Apache.

### Dockerfile for Fano application

Dockerfile for Fano application pulls Ubuntu image and copies binary executable, configuration file, HTML templates etc.

Last, we tells Docker to execute application and listen on `0.0.0.0:7704`. Parameter `--host=0.0.0.0` is required, otherwise our application will only listen on `127.0.0.1`. Because Apache will act as reverse-proxy server on different container, it will not be able to connect to Fano application without `--host=0.0.0.0`.

```
FROM ubuntu

COPY ./config/ /usr/local/fano/config
COPY ./resources/ /usr/local/fano/resources
COPY ./bin/app.cgi /usr/local/fano/app.cgi
CMD ["/usr/local/fano/app.cgi", "--host=0.0.0.0", "--port=7704"]
```
We name this Dockerfile as `fano_dockerfile`.

### Dockerfile for Apache reverse proxy server

Dockerfile for Apache mostly similar to above Dockerfile for CGI but with goal to setup Apache as FastCGI reverse proxy server that connect to our application. This is done by tells Apache to load `mod_proxy` and `mod_proxy_fcgi`.

```
FROM httpd:2.4

#-----------------------------------
# Enable mod_proxy_fcgi mod_rewrite and
# additional virtual host config
#-----------------------------------
RUN sed -i \
        -e 's/^#\(LoadModule .*mod_proxy.so\)/\1/' \
        -e 's/^#\(LoadModule .*mod_proxy_fcgi.so\)/\1/' \
        -e 's/^#\(LoadModule .*mod_rewrite.so\)/\1/' \
        -e 's/^#\(Include .*httpd-vhosts.conf\)/\1/' \
        -e 's/^#ServerName.*/ServerName fano-app/' \
        /usr/local/apache2/conf/httpd.conf && \
    rm /usr/local/apache2/htdocs/index.html

#-----------------------------------
# set default virtual host config
#-----------------------------------
COPY ./vhost.example /usr/local/apache2/conf/extra/httpd-vhosts.conf

#-----------------------------------
# copy public assets (images, css, js)
#-----------------------------------
COPY ./public/ /usr/local/apache2/htdocs

```

We copy `vhost.example` into container virtual host configuraton. We will create it latter.

We name this Dockerfile as `httpd_dockerfile`.

### Create virtual host configuration
We create new file `vhost.example` with content aim to setup FastCGI reverse proxy. `fcgi://fano:7704` line tells Apache to forward FastCGI data to our application container identified as `fano` (This is service we defined in `docker-compose.yaml` below).

```
<VirtualHost *:80>
    ProxyRequests Off
    ProxyPassMatch "/css|js|images|img|plugins|bower_components(.*)|webfonts" !
    ProxyPassMatch ^/(.*)$ "fcgi://fano:7704"
</VirtualHost>
```

### Create docker-compose configuration.

To simplify running both application, we use `docker-compose` with configuration like so

```
version: "2"
services:
    fano:
        build:
            context: .
            dockerfile: fano_dockerfile
        ports:
            - "7704:7704"
    apache:
        build:
            context: .
            dockerfile: httpd_dockerfile
        depends_on:
            - fano
```

Here we tells `docker-compose` to build two services from two dockerfiles we previously created. We also forward traffic on port `7704` so that Apache container can connect it.

Do not forget to save it as `docker-compose.yaml`.

### Run FastCGI application with docker-compose

To run application and Apache, run `build.sh` script first and then,

```
$ docker-compose up
```

To access application, we need to get IP address of Apache container image.
Run

```
$ docker network ls
```

It lists all Docker available networks. If our application is in `testfano` directory,
it is listed as `testfano_default` by default. To get IP address

```
$ docker network inspect testfano_default
```

Find IP address for Apache, for example if it prints `172.20.0.3/16`
Then open browser and visit `http://172.20.0.3` to access our application.

## Deploy Fano SCGI Application with Docker

This will mostly similar to FastCGI reverse proxy setup above, except that `httpd_dockerfile` will load `mod_proxy_scgi` and `vhost.example` contains `scgi://fano:7704` line

## Deploy Fano uwsgi Application with Docker

This will mostly similar to FastCGI reverse proxy setup above, except that `httpd_dockerfile` will load `mod_proxy_uwsgi` and `vhost.example` contains `uwsgi://fano:7704` line

## Deploy Fano CLI-generated Project with Docker

[Fano CLI](/scaffolding-with-fano-cli/) since `v1.13.0` generates `docker-compose.yaml` file during creation of [CGI](/scaffolding-with-fano-cli/creating-project/#scaffolding-cgi-project), [FastCGI](/scaffolding-with-fano-cli/creating-project/#scaffolding-fcgid-project), [SCGI](/scaffolding-with-fano-cli/creating-project/#scaffolding-scgi-project) and [uwsgi](/scaffolding-with-fano-cli/creating-project/#scaffolding-uwsgi-project) project.

Which means you can skip steps above and just compile application and then run with `docker-compose up`.

## Explore more

- [Scaffolding with Fano CLI](/scaffolding-with-fano-cli/)
- [Deployment](/deployment)
