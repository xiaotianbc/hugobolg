---
title: "微软拼音添加小鹤方案"
date: 2020-05-16T13:23:51+08:00
draft: false
tags: ["双拼"]
---

直接添加注册表：

```
Windows Registry Editor Version 5.00

[HKEY_CURRENT_USER\Software\Microsoft\InputMethod\Settings\CHS]
"UserDefinedDoublePinyinScheme1"="小鹤双拼*2*^*iuvdjhcwfg^xmlnpbksqszxkrltvyovt"
```