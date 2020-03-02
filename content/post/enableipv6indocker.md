---
title: "让群晖内的docker访问ipv6网络"
date: 2020-02-23T11:23:51+08:00
draft: false
tags: ["linux"]
---

修改`/usr/syno/etc/packages/Docker/dockerd.json` ，添加：

```json
"ipv6" : true,
"fixed-cidr-v6" : "240e:e0:5745:eb00:211:32ff:fe2c:a603/64"
```
