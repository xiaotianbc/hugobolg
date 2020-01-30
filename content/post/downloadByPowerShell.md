---
title: 用PowerShell下载文件
date: 2019-12-13 19:32:47
tags:
---
有一个笑话是：IE浏览器唯一的作用是下载chrome。 但是当我安装了win8.1嵌入式版本之后，关闭UAC直接不允许打开ie浏览器。。。没办法只能用PowerShell来下载了，IE最后的使命结束了。

``` shell
$client = new-object System.Net.WebClient
$client.DownloadFile('filedownloadlink', 'filesavepath')
```
