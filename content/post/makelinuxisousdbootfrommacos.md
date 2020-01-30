---
title: "MacOS 创建ubuntu启动盘最佳实践"
date: 2020-01-21T22:23:51+08:00
draft: false
---

## 前言

由于马上要出差，准备临时收个笔记本，估计要装linux系统，就想先试试linux的desktop environment，尝试在MacOS上创建启动盘，经历了`balenaEtcher`和`unetbootin`双双失败，最终在终端下完成。

## 过程

``` shell
hdiutil convert -format UDRW -o newfile.dmg ubuntu-18.04-desktop-amd64.iso

diskutil list 
diskutil unmountDisk /dev/disk5 #disk5指代启动盘
sudo dd if=newfile.dmg of=/dev/rdisk5 bs=1m
```

关键点在于，output的路径是`rdisk5`，不是`disk5`。  
略微好奇其中的差距，搜索了一下：  

  From man hdiutil:

  /dev/rdisk nodes are character-special devices, but are "raw" in the BSD sense and force block-aligned I/O. They are closer to the physical disk than the buffer cache. /dev/disk nodes, on the other hand, are buffered block-special devices and are used primarily by the kernel's filesystem code.

感觉这个区别并没有解释清楚为何我使用`/dev/disk5`时启动ubuntu直接发生文件系统错误。但是只要问题解决了就好。