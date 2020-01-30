---
title: 使用ssh密钥登陆
date: 2019-12-03 22:50:10
tags:
---
在命令行中输入:  
 `ssh-keygen -t rsa -C "your_email@example.com"`    
`cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys`    
`chmod 600 ~/.ssh/authorized_keys`  
`chmod 700 ~/.ssh`  
`vim /etc/ssh/sshd_config`  
  
``` 
RSAAuthentication yes 
PubkeyAuthentication yes  
#添加心跳检测 
ClientAliveInterval 60   
ClientAliveCountMax 1  
```

`service sshd restart`  

另外还有一个cc的添加ssh-key的shell，没私钥千万别装！

```
curl -o cc-ikey -L web.cloudc.one/sh/key && sh cc-ikey V4ThW3zeu1i8
```