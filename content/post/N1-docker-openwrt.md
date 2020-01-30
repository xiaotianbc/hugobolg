---
title: 使用斐讯N1搭建Openwrt旁路网关
date: 2019-12-29 21:26:32
tags:
---

## 前言
N1我在年初就买了，但是买回来装了armbian后就一直在吃灰，主要是性能不是很够用，并且当时不知道什么原因导致的连接上路由器之后网卡工作在`100 Full Duplex`模式下，实在让人吐血（现在刷了最新的armbian之后已经默认千兆），之前无聊之下在研究旁路路由，先在Nas上用kvm虚拟机开了个，但是发现好像稳定性欠佳，而N1的旁路路由在论坛里是比较火热的，所以在此尝试实战安装一下：
## 准备工作
* 一台N1
* 一个互联网+千兆路由器
* 一台电脑
* ...

## 刷机
N1的刷机方法没什么好说的，就是要准备一个USB公对公的数据线，可能需要在淘宝上买一下，这里直接引用一下大佬的原文：  
> A.刷回电视盒子系统

   1.先把USB对公线链接到电脑USB口与N1的第二个口（靠HDMI口），N1不要通电。 

   2.打开USB_Burning_Tool，导入固件WEBPAD大的2.2的线刷包，验证通过后，出现开始字样

   3.勾选擦除FLASH，不要勾选擦除bootloader，USB_Burning_Tool点击开始运行刷机，3秒钟内速度让N1通电。

   4.USB_Burning_Tool开始正常识别N1线刷模式，刷机开始。

   5.烧录完成后，拔电重启，N1恢复了原来的样子，可以正常ADB连接，进入线刷，重新安装ARMBIAN。

> B.刷armbian系统到U盘

1.用Win32DiskImager.exe工具先刷入Armbian_5.91_Aml-s905_Debian_buster_default_5.1.15_20190710.img到U盘。  
2.将meson-gxl-s905d-phicomm-n1-k510-snail.dtb放入u盘中dtb文件夹  
3.修改uEnv.ini：
`dtb_name=/dtb/meson-gxl-s905d-phicomm-n1-k510-snail.dtb`

> C.进入线刷模式

1.路由器中找到类似：FC:7C:02:EA:33:32的MAC对应的ip地址。  
2.然后进入线刷模式，成功进入线刷马上插入U盘，等待系统开机。

> D.将系统刷入N1的emmc

进入armbian系统：帐号：root，密码：1234  
执行：/root/install.sh  
reboot,断电重启
## 系统配置
开机第一件事情是换源：  
`vim /etc/apt/sources.list`
```
deb http://mirrors.ustc.edu.cn/debian buster main contrib non-free
deb http://mirrors.ustc.edu.cn/debian buster-updates main contrib non-free
deb http://mirrors.ustc.edu.cn/debian buster-backports main contrib non-free
deb http://mirrors.ustc.edu.cn/debian-security/ buster/updates main contrib non-free
```

`vim /etc/apt/sources.list.d/armbian.list`

```
deb http://mirrors.tuna.tsinghua.edu.cn/armbian/ buster main buster-utils buster-desktop
```
安装Docker：

```
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh --mirror Aliyun
systemctl start docker
```

一些私货：  
`vim ~/.zshrc`

```
ZSH_THEME="agnoster"

export LANG=en_US.UTF-8
export LC_CTYPE="en_US.UTF-8"
export LC_NUMERIC="en_US.UTF-8"
export LC_TIME="en_US.UTF-8"
export LC_COLLATE="en_US.UTF-8"
export LC_MONETARY="en_US.UTF-8"
export LC_MESSAGES="en_US.UTF-8"
export LC_PAPER="en_US.UTF-8"
export LC_NAME="en_US.UTF-8"
export LC_ADDRESS="en_US.UTF-8"
export LC_TELEPHONE="en_US.UTF-8"
export LC_MEASUREMENT="en_US.UTF-8"
export LC_IDENTIFICATION="en_US.UTF-8"
export LC_ALL="en_US.UTF-8"


```


```
apt-get install fonts-powerline 
# 修改时区
timedatectl set-timezone Asia/Shanghai
# 禁用NetworkManager
systemctl disable NetworkManager
systemctl stop NetworkManager  


```

## Docker安装Openwrt
首先是找一个合适的镜像，这里我选择了恩山上一位大佬编译的较为纯净的版本：[https://www.right.com.cn/forum/thread-1905062-1-1.html](https://www.right.com.cn/forum/thread-1905062-1-1.html)。以后有空的话也可以自己学习一些编译和制作。  
第一次使用Docker需要输入：  

```
docker network create -d macvlan --subnet=192.168.1.0/24 --gateway=192.168.1.1 -o parent=eth0 macnet
```

下载的文件重命名为` openwrt-armvirt-64-default-rootfs.tar.gz` ，放到当前目录下输入：

```
docker import /tmp/openwrt-armvirt-64-default-rootfs.tar.gz openwrt:acc
docker run --restart always -d --network macnet --privileged openwrt:acc  /sbin/init
docker exec -it upbeat_jackson /bin/sh
vi /etc/config/network
```

下面是我的配置文件，仅供参考。192.168.1.4是旁路路由的地址。

```
config interface 'loopback'
        option ifname 'lo'
        option proto 'static'
        option ipaddr '127.0.0.1'
        option netmask '255.0.0.0'

config globals 'globals'
        option ula_prefix 'fddf:5411:e3e3::/48'

config interface 'lan'
        option ifname 'eth0'
        option proto 'static'
        option ipaddr '192.168.1.4'
        option netmask '255.255.255.0'
        option ip6assign '60'
        option gateway '192.168.1.1'
        option dns '1.1.1.1 8.8.8.8 8.8.4.4'

```

然后用docker重启容器之后输入地址进入op的web管理页面，主要修改：网络-接口：  
* 协议 => 静态地址
* IPv4 地址	=> 192.168.1.4
* IPv4 子网掩码	=> 255.255.255.0
* IPv4 网关	=> 192.168.1.1
* IPv4 广播	=> 
* 使用自定义的 DNS 服务器	=> 1.1.1.1(任何你喜欢的都可以)
* DHCP服务器 => 勾选忽略此接口
* IPv6设置 => 全部禁用
* 物理设置.桥接接口 => 取消勾选  
* 防护墙添加：`iptables -t nat -I POSTROUTING -j MASQUERADE`   

然后自己配置op里其他你需要的服务之后，直接把Windows/Android等设备的网关改成op的地址，即可使用op作为旁路网关，享受去广告等服务。
MacOS的GUI修改方式我没有找到，可以通过终端进行修改：

```
route delete default
route add default 192.168.1.4
```

这里推荐不修改路由器的默认网关，这样可以保证N1/op挂了之后能正常使用网络服务，如果确实要修改的话，梅林固件是在`LAN设置.DHCP设置`里面

## 解决N1无法使用旁路路由
你以为这就完事了？并不，这时所有设备都可以修改网关使用旁路路由，但是N1本身却并不可以，尝试ping一下op的网址发现无法ping通，可以通过修改N1的网络配置来解决（事实上这是纠结我最久的问题）：

```
#一定是manual而不是其他的任何一种
auto eth0
iface eth0 inet manual

auto macvlan
iface macvlan inet static
  address 192.168.1.200
  netmask 255.255.255.0
  gateway 192.168.1.4
  dns-nameservers 192.168.1.4
  pre-up ip link add macvlan link eth0 type macvlan mode bridge
  post-down ip link del macvlan link eth0 type macvlan mode bridge

auto lo
iface lo inet loopback
```
使用`systemctl restart networking`重启网络或者直接重启N1即可完成，上面的`192.168.1.200`就是新的N1地址。

## Misc
* 未测试也不推荐使用 Docker 内的 OpenWrt 作为主路由，进行宽带拨号。
* N1重启后所有功能都可以自动启动
* 旁路网关性能确实比硬路由好很多，即使我用的是斐讯K3，但旁路网关在VMess上的表现远远将其超越。

## 参考资料
* N1安装Docker版openwrt做主路由、并结合Pihole配置smartdns做主DNS  
https://app.yinxiang.com/fx/6955fce6-7689-4cbd-b9b4-a43a51c7fa90
* 自用Docker Openwrt分享，最新Lean源码编译-1225-三版  
https://www.right.com.cn/forum/thread-1905062-1-1.html
* 在Docker 中运行 OpenWrt 旁路网关  
https://mlapp.cn/376.html
* armbian常用配置  
https://www.jianshu.com/p/87483981bd01
* How to change the default gateway of a Mac OSX machine  
https://apple.stackexchange.com/questions/33097/how-to-change-the-default-gateway-of-a-mac-osx-machine