---
title: "Archlinux-从全新安装到能发这个文章"
date: 2020-02-29T22:44:51+08:00
draft: false
tags: ["linux"]
---

## 前言

今天尝试全新安装 archlinux，从吃完晚饭 19：00 到现在打字 21：42 分大概花了 2 个半小时。感觉还是 OK 的，记录一下过程。

## 安装准备

- 一个 8G 或者以上的 U 盘和一个支持 UEFI 启动的电脑即可。
- 一个能连接到 wiki.archlinux.org 的其他设备（手机 etc（全程靠此 wiki
- 有线网络连接，我不会用 wifi。

## 从安装到启动

首先下载好 ISO，然后刻录到 U 盘里，启动，进入一堆黑框框，以 root@archiso 开头的 Cli 时，我们就开始啦！

### 同步本地时间

`timedatectl set-ntp true`

### 分区

使用 lsblk 或 fdisk -l 确定目标磁盘，使用 mkfs.ext4 格式化。

```shell
fdisk /dev/sdX
#m for help
# n   add a new partition
# w   write table to disk and exit
mkfs.ext4 /dev/sdXx
```

### 挂载

假如硬盘是`/dev/sda`，要安装到的分区是`/dev/sda4`，EFI 分区是`/dev/sda1`

```shell
mount /dev/sda4 /mnt
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot
```

### 下载 base

首先修改镜像列表：

```shell
vim /etc/pacman.d/mirrorlist
#v   /China 找到中国的mirror
#v   按yy复制到缓冲区
#v   按gg回到文件最上层
#v   按p粘贴中国的mirror
```

安装基础包

```shell
pacstrap /mnt base linux linux-firmware
```

### 分区表

```shell
genfstab -U /mnt >> /mnt/etc/fstab
```

### chroot

进入新系统：

```shell
arch-chroot /mnt
```

### 时区

```
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
#运行 hwclock(8) 以生成 /etc/adjtime
hwclock --systohc
```

### 本地化

```
vim /etc/locale.gen
locale-gen
#警告: 不推荐在此设置任何中文 locale，会导致 TTY 乱码。
vim /etc/locale.conf  #LANG=en_US.UTF-8
```

### 网络

```
vim /etc/hostname  #myhostname
vim /etc/hosts
```

```shell
127.0.0.1	localhost
::1		localhost
127.0.1.1	myhostname.localdomain	myhostname
```

设置 Root 密码：

```
passwd
```

### Grub：

```shell
pacman -S dosfstools grub efibootmgr
grub-install --target=x86_64-efi --efi-directory=/mnt/boot/efi --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```

### 创建非 root 用户

需要安装`vi`和`visudo`，然后在 visudo 里加入新建的用户名：

```shell
useradd -m -G wheel -s /bin/bash lwxntm
```

然后基本就可以重启了，如果找不到 grub 启动项，可以在 windows 里用 easyUEFI 把`/boot/EFI/EFI/grub/grub_x64.efi`加入引导项。

## 启动后配置

使用非 root 用户登录系统后，发现：WTF?? 居然连 DHCP 都没开，也太 hardcore 了吧。。

### 用 Systemd-networkd 配置网络：

```shell
sudo vim /etc/systemd/network/20-wired.network
```

如果你需要自定义网关并且暂时禁用 ipv6：(替换 eno1 为你的网卡名)

```
[Match]
Name=eno1

[Network]
Address=192.168.1.111/24
Gateway=192.168.1.2
DNS=192.168.1.2
#DHCP=ipv4
LinkLocalAddressing=ipv4
```

如果你只需要 dhcp：

```
[Match]
Name=eno1

[Network]
DHCP=ipv4
```

然后启动即可：

```
sudo systemctl start systemd-networkd.service
```

如果 dns 解析没有正常工作的话：

```
sudo vim /etc/resolv.conf
nameserver 192.168.1.2
```

## 安装 GUI

老是黑框框也不适合 PC，还是装一下 de 吧，我比较推荐 KDE，其他的都太丑了。
先安装显卡驱动：

```shell
sudo pacman -S mesa
```

然后安装 xorg：

```shell
sudo pacman -S Xorg
```

然后直接安装 kde，kde 的应用，fcitx 输入法：

```
sudo pacman -S plasma-meta kde-applications-meta fcitx-im
```

装完了，就让他启动吧！对了，你用的应该是 zsh 吧？？`vim .zprofile`
加入以下内容，这样我们从 tty 登录以后就能自动启动`startx`：

```
if [[ ! $DISPLAY && $XDG_VTNR -eq 1 ]]; then
        exec startx
fi
#fcitx
export XIM_PROGRAM=fcitx
export XIM=fcitx
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS="@im=fcitx"
```

然后加入 xorg 的启动 kde 命令`vim .xinitrc`：

```shell
exec startplasma-x11
```

完成之后，退出再重新登录试试，理论上来说应该可以直接看到桌面了。

更换中文字体，推荐：

```shell
sudo pacman -S ttf-sarasa-gothic noto-fonts-cjk
```

配置输入法之类就不细谈了。问题不大.

## 安装SDDM

```shell
sudo pacman -S sddm sddm-kcm
```

`vim /etc/sddm.conf.d/autologin.conf`

```conf
[Autologin]
User=lwxntm
Session=plasma.desktop
```