---
title: "修复WSL中glibc的错误"
date: 2020-05-03T13:23:51+08:00
draft: false
tags: ["wsl"]
---
 
最近在`Windows[18363.778]`上使用WSL的时候遇到了一些的问题，包括但不限于`htop`卡死，`Rust`无法安装等.  
原因是`glibc`的`1.3.1`版本修改了硬件时钟相关的ABI，导致在WSL上无法正常工作，如果你在使用`Ubuntu 20.04`或者`ArchLinux`等系统时就会触发。  
微软在github上表示相关补丁已经在`版本2004`以上修复了这一问题， 但是如果你还在用1909或者更早的版本，则需要手动降级才能解决：  

Ubuntu 20.04:

```shell
sudo add-apt-repository ppa:rafaeldtinoco/lp1871129
sudo apt update
sudo apt install libc6=2.31-0ubuntu8+lp1871129~1 libc6-dev=2.31-0ubuntu8+lp1871129~1 libc-dev-bin=2.31-0ubuntu8+lp1871129~1 -y --allow-downgrades
sudo apt-mark hold libc6
```

ArchLinux:

```shell
yay -S downgrade
downgrade glibc lib32-glibc
#choose 1.3.0 -3
```