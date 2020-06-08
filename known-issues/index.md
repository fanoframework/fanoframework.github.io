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
On FreeBSD 12, when linker `ld` is symlinked to LLVM linker `/usr/bin/ld.lld`, this cause segmentation fault when project is using `cthreads` unit.
Workaround is to modify `ld` to point to GNU linker `/usr/bin/ld.bfd`.

```
$ sudo ln -s -f /usr/bin/ld.bfd /usr/bin/ld
```
## Explore more

- [Getting Started](/getting-started)
