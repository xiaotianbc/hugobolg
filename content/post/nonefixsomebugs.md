---
title: "斐讯N1踩坑不完全记录"
date: 2020-01-09T23:19:34+08:00
---


斐讯 N1 想要当做一个单独的 linux 主机用需要踩很多坑，在此记录一下：

如果安装的时候发现有提示`tar ...stamp ... in future`这是因为解压出的文件的修改时间大于当前系统时间导致的，不想看到的话就修改一下当前时间：  

```shell
date -s "2020-01-11 15:15:15"
```

在安装之前，要编辑`/root/install.sh`里的分区设定：

```shell
parted -s "${DEV_EMMC}" mkpart primary fat32 700M 828M
parted -s "${DEV_EMMC}" mkpart primary ext4 829M 100%
```

改成下面这种数值是比较安全的：

```shell
parted -s "${DEV_EMMC}" mkpart primary fat32 1024M 1152M
parted -s "${DEV_EMMC}" mkpart primary ext4 1153M 100%
```

进系统之后把软件源换一下，本人上海电信测试华为云效果不错。armbian软件源建议直接注释掉，一般用不到。

```list
deb [arch=arm64,armhf] http://mirrors.huaweicloud.com/debian buster main contrib non-free
deb [arch=arm64,armhf] http://mirrors.huaweicloud.com/debian buster-updates main contrib non-free
deb [arch=arm64,armhf] http://mirrors.huaweicloud.com/debian buster-backports main contrib non-free
deb [arch=arm64,armhf] http://mirrors.huaweicloud.com/debian-security/ buster/updates main contrib non-free
```

然后把`initramfs-tools`这个软件包设置为`hold`即不更新，更新这个软件会导致boot分区发现变化。

``` shell
echo "initramfs-tools hold" | dpkg --set-selections
echo "initramfs-tools-core hold" | dpkg --set-selections
```

关于联网方面，有无线连接和有限连接两种选择。  

>如果使用有线连接，可以把一些用处不大的服务关掉。  

``` shell
sysystemctl stop unattended-upgrades.service
systemctl disable unattended-upgrades.service
sysystemctl stop NetworkManager-wait-online.service
systemctl disable NetworkManager-wait-online.service
sysystemctl stop NetworkManager.service
systemctl disable NetworkManager.service
```

 >如果是无线连接:
 
 进系统后直接输入`nmtui`即可进行配置。`Edit a connection=>WIFI.add=>SSID,Password,IPv4CONFIGURATION...etc`配置完成后选择一次`Activate a connection`即可完成连接，无需其他干预，另外请千万不要使用`armbian-config`程序内的任何网络相关选项，会导致难以排查的问题。  

N1挂载优盘的方法：

```shell
#/etc/fstab
/dev/sda1        /home/lwxntm/sda1   exfat       utf8,uid=1000,gid=1000,umask=0000      0 0
```
