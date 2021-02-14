---
title: "手动转移mhgu存档获得特典的作弊条"
date: 2021-02-14T10:15:51+08:00
draft: false
tags: ["switch"]
---



笔者把已经玩了很久的mhgu存档转移到了具有特典和dlc配信任务的新存档中，故写此作弊条。

需要用到的工具：

- 十六进制编辑器 ： https://www.hhdsoftware.com/free-hex-editor
- switch存档管理工具： https://github.com/J-D-K/JKSV
- mhgu特典空存档： 链接：https://pan.baidu.com/s/115wlcykE45TDAX3rs-5SDQ 提取码：v3ez 



首先用JKSV保存现有存档，然后导入特典存档，如果你原本的角色在第n个角色槽则需要在特典存档中第n个角色槽中建立存档，然后把该空存档导出到电脑。

用16进制编辑器打开原角色存档，搜索角色名称，从第一个字符起始的位置，选择0011D088个长度并复制。打开空存档，跳转到刚才的起始位置，粘贴进入，保存，然后导入到switch后用JKSV还原即可。

