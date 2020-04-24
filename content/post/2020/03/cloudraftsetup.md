---
title: "CloudRaft-Ipv6大盘鸡配置"
date: 2020-01-26T16:57:51+08:00
draft: false
tags: ["linux"]
---

## 前言

纯个人防止遗忘使用，无明显解释。  
[云筏科技](www.cloudraft.cn)近日推出了一款性价比比较高的大盘鸡，我购买了最低配的一款，具体配置如下：

```markdown
KvmNat-S-200G
￥ 10.00CNY 月付
1 核 (100%峰值 合理使用)
500M 内存 (独享)
200G RaidZ2 硬盘 (IO100)
1T 流量@200M 带宽 (共享)
5 个 IPv4 端口+1 个 IPv6
无技术服务 不可退款
```

机器开通之后我使用的是 debian10 系统，先做了之前记录过的一些配置,详见[Linux 相关私货](https://78997899.xyz/posts/linux-micro-thing.html)  
还有一些：

```shell
# 创建swap分区
dd if=/dev/zero of=/swapfile bs=1M count=4096
mkswap /swapfile
swapon /swapfile
vim /etc/fstab
# 在末尾添加一行：
/swapfile swap swap defaults 0 0
#修改hostname
vim /etc/hostname
```

## 用 Cloudflare 反代实现外网访问

由于该 vps 只提供了 5 个固定端口的 ipv4 以及一个 ipv6，为了实现端口的单口复用，一般可以通过 nginx 反代实现端口复用：下面是我的 nginx 配置命令：

```shell
cd xxx

wget https://nginx.org/download/nginx-1.17.8.tar.gz && tar zxf nginx-1.17.8.tar.gz
wget https://github.com/openssl/openssl/archive/OpenSSL_1_1_1d.tar.gz && tar zxf OpenSSL_1_1_1d.tar.gz
git clone https://github.com/google/ngx_brotli.git && cd ngx_brotli && git submodule update --init && cd ..
wget https://ftp.pcre.org/pub/pcre/pcre-8.43.tar.gz && tar zxf pcre-8.43.tar.gz
wget https://www.zlib.net/zlib-1.2.11.tar.gz && tar zxf zlib-1.2.11.tar.gz
cd nginx-1.17.8
./configure --prefix=/usr/local/nginx --with-openssl=/root/workspace/maked/openssl-OpenSSL_1_1_1d --with-openssl-opt=enable-tls1_3 --with-http_v2_module --with-http_ssl_module --with-http_gzip_static_module --add-module=/root/workspace/maked/ngx_brotli --with-pcre=/root/workspace/maked/pcre-8.43 --with-zlib=/root/workspace/maked/zlib-1.2.11
```

nginx 安装完成后，先编辑 `/usr/local/nginx/conf/nginx.conf` 添加 ipv6 支持

```conf
#listen 80;下面添加一行：
listen [::]:80 ipv6only=on;
#放在最后
include conf.d/*.conf;
```

然后编辑`/usr/local/nginx/conf/conf.d/*.conf` ：

```conf
server {
    listen 80;
    listen [::]:80;
    server_name cr.78997899.xyz;
    index index.html;
    root /home/wwwroot/cr;
    location /qb/ {
        proxy_pass http://127.0.0.1:8080/;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $host;
        proxy_set_header Upgrade-Insecure-Requests 1;
        }
     location /d/ {
        proxy_pass http://127.0.0.1:8081/;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $host;
        proxy_set_header Upgrade-Insecure-Requests 1;
        }

}
```

然后再 cloudflare 里添加 AAAA 记录即可。  
用CF进行流量转发，CF转发的端口包括2052 2053 2082 2083 2086 2087 2095 2096 8080 8443 8880  

顺便附上一个 golang 目录分享程序:编译完成后放到服务器上，在需要分享的目录下运行，用 nginx 反代这个端口即可。

```go
package main

import "net/http"

func main() {
	h:=http.FileServer(http.Dir("."))
	_ = http.ListenAndServe(":10001", h)
}
```
