---
title: Known Issues
description: List of some of known issues
---

<h1 class="major">Known Issues</h1>

## <a name="issue-with-gnu-linker"></a>Issue with GNU Linker

When running `build.sh` script with Free Pascal 3.0.4, you may encounter following warning:

```
/usr/bin/ld: warning: public/link.res contains output sections; did you forget -T?
```

This is known issue between Free Pascal and GNU Linker. See
[FAQ: link.res syntax error, or "did you forget -T?"](https://freepascal.org/faq.html#unix-ld219)

However, this warning is minor and can be ignored. It does not affect output executable.

To remedy, just upgrade Free Pascal and binutils.

## <a name="issue-with-gcc-library-search-path"></a>Issue with development tools library search path

When running `build.sh` script, you may encounter following minor warning:

```
Warning: "crtbegin.o" not found, this will probably cause a linking failure
Warning: "crtend.o" not found, this will probably cause a linking failure
```

This is mostly because you install Free Pascal compiler before install development tools library.

First, you need to locate development tools library path on your computer. Run

```
# find / -name crtbegin.o
```

For example, in my Centos 7, command above will output (yours may be different)

```
/usr/lib/gcc/x86_64-redhat-linux/8/32/crtbegin.o
/usr/lib/gcc/x86_64-redhat-linux/8/crtbegin.o
```

Edit Free Pascal main configuration, `/etc/fpc.cfg` file and add following lines

```
#ifdef cpui386
-Fl/usr/lib/gcc/x86_64-redhat-linux/8/32
#endif

#ifdef cpux86_64
-Fl/usr/lib/gcc/x86_64-redhat-linux/8
#endif
```

## <a name="issue-on-freebsd"></a>Issue on FreeBSD

On FreeBSD 12, if you compile Free Pascal from source or tar file, you may get error when you try to run Fano web application executable.

```
$ ./bin/app.cgi
Segmentation fault (core dumped)
```
This is Free Pascal issue.
Try to update to latest FreeBSD and install latest binutils

```
$ sudo pkg install binutils
```

and then rebuild the application, if the error does not go away, you may want to symlink `ld` to point to GNU linker `/usr/bin/ld.bfd` as a workaround.
If you can not find `ld.bfd` in `/usr/bin`, you may want to check other directories such as `/usr/local/bin`.

```
$ sudo ln -s -f /usr/bin/ld.bfd /usr/bin/ld
```

When linker `ld` is symlinked to LLVM linker `/usr/bin/ld.lld` (default), sometime, this cause segmentation fault when project is using `cthreads` unit.

However, latest binutils on newer FreeBSD releases no longer ships with `ld.bfd`.
Other workaround you may try is to install Free Pascal from FreeBSD package manager

```
$ sudo pkg install fpc
```
## <a name="issue-with-free-pascal-3.2.0"></a>Issue with Free Pascal 3.2.0

When upgrading Free Pascal from 3.0.4 to 3.2.0, you may notice that Free Pascal 3.2.0 will generate several notes regarding inline methods it can not inline while in Free Pascal 3.0.4 there was no such notes generated.

When you compile examples program from [Examples page](/examples) with development build type, (i.e, you run build script with `BUILD_TYPE=dev`),

```
$ BUILD_TYPE=dev ./build.sh
```

You may find that compilation will stop with Free Pascal 3.2.0 compiler. This is because in development build, we use compiler options `Sewn` which will stop compilation when any error, warning or notes are generated.

```
#----------------------------------------------
# halt compiler after error, warning and notes
#----------------------------------------------
-Sewn
```

To remedy the situation, just edit `build.dev.cfg` file in example root directory and replace `Sewn` to `Sew` so that compilation will not stop on notes.

## <a name="missing-libmicrohttpd-development-package"></a>Missing Libmicrohttpd development package

If you [create libmicrohttpd-based web application](/scaffolding-with-fano-cli/creating-project#scaffolding-libmicrohttpd-project) and get linking error

```
/usr/bin/ld : cannot find -lmicrohttpd
```
when running `build.sh` script.

You need to install libmicrohttpd development package. For Debian-based distribution,

```
$ sudo apt install libmicrohttpd-dev
```

For Fedora-based distribution,

```
$ sudo yum install libmicrohttpd-devel
```
## <a name="missing-etc-fpc-cfg"></a>Missing /etc/fpc.cfg

`build.sh` script that [Fano CLI](/scaffolding-with-fano-cli) generates for each project, needs `/etc/fpc.cfg`. If you install Free Pascal to non default directory or using tool such as *[fpcupdeluxe](https://github.com/LongDirtyAnimAlf/fpcupdeluxe)*, it may show error about missing `/etc/fpc.cfg` file or `Fatal: Can't find unit system used by ...` error. To remedy this situation, just create symbolic link in `/etc` directory to actual `fpc.cfg` file. For example,

```
$ sudo ln -s ~/fpcupdeluxe/fpc/bin/x86_64-linux/fpc.cfg /etc/fpc.cfg
```
## <a name="missing-mysql-client-library"></a>Missing MySQL client library

If you encounter error *Can't load default MySQL library ("libmysqlclient.so.20" or "libmysqlclient.so"). Check your installation*, you need to install MySQL client development library. For example on Debian,

```
$ sudo apt install libmysqlclient-dev
```
## <a name="shut-down-database-server-may-cause-memory-leak"></a>Memory leak due to database shutdown.

When Fano Framework web application is connected to a database server, and then database shuts down while application is still running. When application shuts down, [it leaks memory](https://github.com/fanoframework/fano/issues/14). This issue is related to Free Pascal [bug report 37993](https://bugs.freepascal.org/view.php?id=37993). Memory leak only happens when database shuts down and never restart. Memory leak does not happen when database server shuts down and then restarts or when database does not runs at all.

## Explore more

- [Getting Started](/getting-started)
