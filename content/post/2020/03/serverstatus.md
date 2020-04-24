---
title: 记录搭建一个服务器监控页面的过程
date: 2019-12-22 22:16:08
tags: ["linux"]
---
## 前言

首先是页面：[https://list.78997899.xyz](https://list.78997899.xyz)  
搭建这个页面的原因是看到了：[这个帖子](https://www.hostloc.com/thread-626508-1-1.html)，便想尝试一下。

<!-- more -->

## 准备工作

* 一个Linux服务器
* 一个域名  
当然没有域名应该也是可以的，但是强迫症必须上https。  

## 建立一个可访问的网站

首先是安装nginx，方法有很多，直接`apt install nginx`也是可行的，当然最好还是编译安装，可以自己拓展插件，这不在本文的讨论范围之内了。  
安装完成之后，编辑`nginx/conf/conf.d/your_name.conf`：  

```
server {
        listen 80;
	server_name your_full_domain_name;
	root  /which_you_want_to_put;
	index index.html;
	    }
```

用[acme.sh](https://github.com/Neilpang/acme.sh/wiki/%E8%AF%B4%E6%98%8E)来实现ssl是一个很方便的办法，按照wiki里的内容做，安装完成证书之后，重新编辑`nginx/conf/conf.d/your_name.conf` 

```
    server {
        listen 443 ssl http2;
        ssl_certificate       /data/yourname.cer;
        ssl_certificate_key   /data/yourname.key;
        ssl_protocols         TLSv1.2 TLSv1.3;
        ssl_ciphers           TLS13-AES-256-GCM-SHA384:TLS13-CHACHA20-POLY1305-SHA256:TLS13-AES-128-GCM-SHA256:TLS13-AES-128-CCM-8-SHA256:TLS13-AES-128-CCM-SHA256:EECDH+CHACHA20:EECDH+CHACHA20-draft:EECDH+ECDSA+AES128:EECDH+aRSA+AES128:RSA+AES128:EECDH+ECDSA+AES256:EECDH+aRSA+AES256:RSA+AES256:EECDH+ECDSA+3DES:EECDH+aRSA+3DES:RSA+3DES:!MD5;
	server_name your_full_domain_name;
        index index.html;
        root  /which_you_want_to_put;
	
}
    server {
        listen 80;
	server_name your_full_domain_name;
	return 301 https://your_full_domain_name$request_uri;
    }

```

上面操作做完之后，重启nginx确认你的域名能正确访问，就可以进行下一步了。  

## 安装ServerStatus

ServerStatus的安装基本可以参考[https://github.com/BotoX/ServerStatus/](https://github.com/BotoX/ServerStatus/)。
  
  其中config的配置文件是类似下面的：
  
  ``` json
  {"servers":
	[
		{
			"username": "客户端登陆用户名",
			"name": "不知道具体用处但是可以和上面保证一致",
			"type": "虚拟化类型，例如KVM",
			"host": "显示的名称：例如阿里云",
			"location": "US/CN/SG",
			"password": "当然是密码啦"
    },
    
	]
}  

```

保持运行的话建议使用systemd来实现，`vim /etc/systemd/system/sergate.service`

```
[Unit]
Description=ServerStatus Master Server
After=syslog.target
After=network.target

[Service]
[Unit]
Description=ServerStatus Master Server
After=syslog.target
After=network.target

[Service]
Type=simple
User=root
#WorkingDirectory=/home/...
ExecStart=/home/lwxntm/coolcodes/ServerStatus/clients/client.py
ExecReload=/bin/kill -HUP $MAINPID

[Install]
WantedBy=multi-user.target
```

```shell
ExecStart=/path/ServerStatus/server/sergate -c /path/ServerStatus/server/config.json -d /your_web_path/status
```

## 更换美化界面

参考作者的[文档](https://github.com/krwu/ServerStatus-web/blob/master/README.zh_CN.md)即可，建议直接下载编译完成的版本。

## 参考资料
* https://github.com/BotoX/ServerStatus/
* https://github.com/krwu/ServerStatus-web/blob/master/README.zh_CN.md
* https://github.com/Neilpang/acme.sh/wiki/%E8%AF%B4%E6%98%8E