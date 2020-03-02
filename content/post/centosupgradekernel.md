---
title: "CentOS升级内核版本"
date: 2020-01-29T19:23:51+08:00
draft: false
tags: ["linux"]
---

迫于某些原因开始使用Centos，记录一下升级centos内核的办法：  

导入公钥：

```shell
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
```

To install ELRepo for RHEL-7, SL-7 or CentOS-7:

```shell
yum install https://www.elrepo.org/elrepo-release-7.0-4.el7.elrepo.noarch.rpm
```

列出可用的内核

```shell
yum --disablerepo="*" --enablerepo="elrepo-kernel" list available
```

安装mainline内核

```
yum -y --enablerepo=elrepo-kernel install kernel-ml.x86_64 kernel-ml-devel.x86_64 
```

查看当前grub内核启动顺序

```shell
awk -F\' '$1=="menuentry " {print $2}' /etc/grub2.cfg
```

修改`/etc/default/grub`里默认值为`0`（如果上面的命令第一个是刚刚安装的内核），然后重新创建内核配置文件：

``` shell
grub2-mkconfig -o /boot/grub2/grub.cfg
```