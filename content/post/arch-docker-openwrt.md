---
title: "archlinux里用docker运行openwrt"
date: 2020-04-06T22:44:51+08:00
draft: false
tags: ["linux"]
---

## 安装archlinux
我推荐使用anarchy来直接解决，没必要硬折腾。 
```
https://github.com/AnarchyLinux/installer/releases
```
## 安装docker
```
yay -S docker
sudo systemctl start docker.service
sudo systemctl enable docker.service
vim /etc/docker/daemon.json
{
  "storage-driver": "overlay2"
}
sudo systemctl restart docker.service
```

## 编译openwrt的镜像

必须在全局代理下执行，如果你还没有实现docker里运行，可以用virtualbox来跑koolshare的包.

```
yay -S base-devel antlr3 gperf
git clone https://github.com/coolsnowwolf/lede
# remove '#' of feeds.conf.default
#可选 git clone -b dev-19.07 https://github.com/Lienol/openwrt
cd lede
./scripts/feeds update -a 
./scripts/feeds install -a
make defconfig
make menuconfig
# Target Images -> Root fs archives-> tar.gz && images ->squashfs
make download V=s
make V=s -j13
```
完成之后镜像就在`./bin/target/x86/64/openwrt-x86-64-generic-rootfs.tar.gz`

## 配置篇

下文假设真机网卡是eno1，我们将会用docker创建一个叫macnet的macvlan虚拟网卡。
但是macvlan是无法和宿主机互通的，还要再创建一个叫vLAN的macvlan虚拟网卡，桥接到eno1，然后走：eno1->vLAN->macnet的路线。

```
#打开所需要使用的网卡的混杂模式
sudo ip link set eno1 promisc on
#加载pppoe模块
sudo modprobe pppoe
#用docker创建一个叫`macnet`的macvlan虚拟网卡
sudo docker network create -d macvlan --subnet=192.168.1.0/24 --gateway=192.168.1.1  -o parent=eno1 -o macvlan_mode=bridge macnet
#创建容器镜像
sudo docker import openwrt-x86-64-generic-rootfs.tar.gz op:lm
sudo docker run --restart always -d --network macnet --privileged --name oplm op:lm  /sbin/init
#修改op里的信息
docker exec -it oplm /bin/sh
vim /etc/config/network #修改op的ip
vim /etc/resolv.conf   #修改为一个有效的远程dns
/etc/init.d/network restart
```

允许主机访问，新建macvlan虚拟接口 下面的192.168.1.105是本机的IP，192.168.1.22是为openwrt设置的ip。

```
sudo ip link add link eno1 vLAN type macvlan mode bridge   
sudo ip addr add 192.168.1.105/24 brd + dev vLAN   
sudo ip link set vLAN up
#走到这里，我们可以尝试访问docker里的op应该已经没有问题了。下面是把默认网关改成op。
sudo ip route del default                
sudo ip route add default via 192.168.1.22 dev vLAN
```
开机自启动
以上部分命令重启后会失效，把以下文件加入开机自动启动的脚本：`/etc/docker-rc.sh`

```
sudo ip link set eno1 promisc on
sudo modprobe pppoe
sudo ip link add link eno1 vLAN type macvlan mode bridge
sudo ip addr add 192.168.1.105/24 brd + dev vLAN
sudo ip link set vLAN up
sudo ip route del default
sudo ip route add default via 192.168.1.22 dev vLAN
sudo echo "nameserver 192.168.1.22" > /etc/resolv.conf
sudo sh -c 'echo 1 > /proc/sys/net/ipv6/conf/vLAN/disable_ipv6'
sudo sh -c 'echo 1 > /proc/sys/net/ipv6/conf/eno1/disable_ipv6'
```
再添加一个systemd服务：`sudo vim /usr/lib/systemd/system/docker-macnet.service`
```
[Unit]
Description=
Documentation=
After=network.target
Wants=
Requires=
  
[Service]
ExecStart=/bin/sh /etc/docker-rc.sh
ExecStop=
ExecReload=/bin/sh /etc/docker-rc.sh
Type=simple
  
[Install]
WantedBy=multi-user.target
```
别忘了`sudo systemctl enable docker-macnet.service`
