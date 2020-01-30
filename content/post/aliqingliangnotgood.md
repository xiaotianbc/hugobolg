---
title: "阿里云轻量没有想象中的好"
date: 2020-01-28T12:23:51+08:00
draft: false
---
原本准备这个月ECS到期之后，购买一个hk的轻量，去程联调回程CN2应该还是比较香的，但是今天测试发现速度差距还是比较大的：

``` shell

#阿里云轻量hk下午三点的ping
--- 149.129.111.* ping statistics ---
2692 packets transmitted, 2463 packets received, 8.5% packet loss
round-trip min/avg/max/stddev = 44.096/45.856/54.434/0.681 ms

#同一时间的ecs新加坡
--- 161.117.87.* ping statistics ---
2698 packets transmitted, 2698 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 69.914/71.706/77.631/0.599 ms

```

hk的丢包在白天已经快到10%了，基本是无法使用的类型。  
