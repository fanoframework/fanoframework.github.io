---
title: Sending Email
description: Tutorial on how to send email with some utilities provided by Fano Framework
---

<h1 class="major">Sending Email</h1>

## Send email message

Free Pascal users have some options when sending email. Some library such as Indy has TIdSMTP which connect directly to SMTP server.
As alternative, Fano Framework offers a way to send email by using sendmail binary.

## IMailer interface

To abstract away mail sender implementation, `IMailer` interface is provided. It is a contract for any class having capability to send email.

## Sending email with sendmail

Current implementation of `IMailer` interface supports sending email using sendmail binary, from [ssmtp library](https://linux.die.net/man/8/ssmtp) thorough class `TSendmailMailer`. To use it, register its factory class with [container](/dependency-container).

```
container.add('mailer', TSendmailMailerFactory.create());
```

```
var amailer : IMailer;
...
amailer := container['mailer'] as IMailer;
amailer.recipient := 'recipient@example.com';
amailer.sender := 'no-reply@example.com';
amailer.subject := 'Hello';
amailer.body := 'Hello world';
amailer.send();
```

## Setting up ssmtp library

To install ssmtp in Debian-based distribution,

```
$ sudo apt install ssmtp
```
or Fedora-based,

```
$ sudo yum install ssmtp
```

Edit `/etc/ssmtp/ssmtp.conf` and add following lines,

```
mailhub=smtp.yourownserver.com:25
AuthUser=YourUsername
AuthPass=YourOwnSecretPassword
FromLineOverride=YES
```

Replace `mailhub`, `AuthUser`, `AuthPass` with your own value. If you do not setup dedicated SMTP server, you can use external SMTP server such as gmail or mailtrap, or MailHog. Both mailtrap and MailHog is fake SMTP server which helps sending test email without fear of becoming email spammer during development.

For example, this is configuration for sending email with mailtrap.io on development machine. You should replace `YourUsername` and `YourOwnSecretPassword` with username and password of your mailtrap.io account.

```
mailhub=smtp.mailtrap.io:2525
AuthUser=YourUsername
AuthPass=YourOwnSecretPassword
FromLineOverride=YES
```
<a href="/assets/images/mailtrap.io.png">
<img src="/assets/images/mailtrap.io.png" alt="Mailtrap.io credential" width="50%">
</a>

To test if your configuration works, try to send an email from command line

```
$ echo -e 'Subject: test\n\nTesting ssmtp' | sendmail -v tousername@example.com
```

## Sending email with Indy

Current implementation of `IMailer` interface supports sending email using Indy, thorough class `TIndyMailer`. To use it, register its factory class with [container](/dependency-container).

```
container.add('mailer', TIndyMailerFactory.create());
```

```
var amailer : IMailer;
...
amailer := container['mailer'] as IMailer;
...
```
To be able to use `TIndyMailer` you need to add conditional compilation define `USE_INDY`
in your project.

```
{$DEFINE USE_INDY}
```
or add it via Free Pascal configuration file or commmand line.

```
-dUSE_INDY
```

When using Indy, you need to be aware of [intentional memory leak issue of Indy library](/known-issues#indy-memory-leak-issue).

You need to set `INDY_DIR` environment variable to point to Indy library base directory.

```
$ export INDY_DIR="/path/to/Indy"
```
## Sending email with Synapse

Current implementation of `IMailer` interface supports sending email using Synapse, thorough class `TSynapseMailer`. To use it, register its factory class with [container](/dependency-container).

```
container.add('mailer', TSynapseMailerFactory.create());
```
When you need its instance, query it from service container.
```
var amailer : IMailer;
...
amailer := container['mailer'] as IMailer;
...
```
To be able to use `TSynapseMailer` you need to add conditional compilation define `USE_SYNAPSE`
in your project.

```
{$DEFINE USE_SYNAPSE}
```
or add it via Free Pascal configuration file or commmand line.

```
-dUSE_SYNAPSE
```

And set `SYNAPSE_DIR` environment variable to point to Synapse library base directory.

```
$ export SYNAPSE_DIR="/path/to/synapse"
```

## Explore more

- [Utilities](/utilities)
- [Fano Email](https://github.com/fanoframework/fano-email) example project demonstrates how to send email.
