---
title: "低版本内核编译Qt时报错修复"
date: 2020-02-02T16:23:51+08:00
draft: false
---

``` shell
error while loading shared libraries: libQt5Core.so.5: cannot open shared object file: No such file or directory
```

fixed by

```shell
sudo strip --remove-section=.note.ABI-tag /usr/lib/x86_64-linux-gnu/libQt5Core.so.5
```

links:

* <https://github.com/dnschneid/crouton/wiki/Fix-error-while-loading-shared-libraries:-libQt5Core.so.5>