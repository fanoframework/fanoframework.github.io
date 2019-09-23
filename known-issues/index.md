---
title: Known Issues
description: List of some of known issues
---

<h1 class="major">Known Issues</h1>

## Issue with GNU Linker

When running `build.sh` script, you may encounter following warning:

```
/usr/bin/ld: warning: public/link.res contains output sections; did you forget -T?
```

This is known issue between Free Pascal and GNU Linker. See
[FAQ: link.res syntax error, or "did you forget -T?"](https://www.freepascal.org/faq.var#unix-ld219)

However, this warning is minor and can be ignored. It does not affect output executable.

## Explore more

- [Getting Started](/getting-started)

<ul class="actions">
    <li><a href="/documentation" class="button">Documentation</a></li>
</ul>
