---
title: Known Issues
description: List of some of known issues
---

<h1 class="major">Known Issues</h1>

## <a name="issue-with-gnu-linker"></a>Issue with GNU Linker

When running `build.sh` script, you may encounter following warning:

```
/usr/bin/ld: warning: public/link.res contains output sections; did you forget -T?
```

This is known issue between Free Pascal and GNU Linker. See
[FAQ: link.res syntax error, or "did you forget -T?"](https://freepascal.org/faq.html#unix-ld219)

However, this warning is minor and can be ignored. It does not affect output executable.

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

## Issue on FreeBSD

On FreeBSD 12, if you compile Free Pascal from source, you may get error when you try to run Fano web application executable.

```
$ ./bin/app.cgi
Segmentation fault (core dumped)
```

Try to update to latest FreeBSD and install latest binutils

```
$ sudo pkg install binutils
```

and then rebuild the application, if the error does not go away, you may want to symlink `ld` to point to GNU linker `/usr/bin/ld.bfd` as a workaround.

```
$ sudo ln -s -f /usr/bin/ld.bfd /usr/bin/ld
```

When linker `ld` is symlinked to LLVM linker `/usr/bin/ld.lld` (default), sometime, this cause segmentation fault when project is using `cthreads` unit.

However, latest binutils on newer FreeBSD releases no longer ships with `ld.bfd`.
Other workaround you may try is to install Free Pascal from FreeBSD package manager

```
$ sudo pkg install fpc
```
## Issue with Free Pascal 3.2.0

When upgrading Free Pascal from 3.0.4 to 3.2.0, you may notice that Free Pascal 3.2.0 will generate several notes regarding inline methods it can not inline while in Free Pascal 3.0.4 there was no such notes generated.

When you compile examples program from [Examples page](/examples) with development build type, (i.e, you run build script with `BUILD_TYPE=dev`),

```
$ BUILD_TYPE=dev ./build.sh
```

You may find that compilation will stop with Free Pascal 3.2.0 compiler. This is because in development build, we use compiler options `Sewn` which will stop compilation when any error, warning or notes are generated.

To remedy the situation, just edit `build.dev.cfg` file in example root directory and replace `Sewn` to `Sew` so that compilation will not stop on notes.

## Explore more

- [Getting Started](/getting-started)
