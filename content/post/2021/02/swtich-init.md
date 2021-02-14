---
title: "从零开始使用大气层RCM破解switch的作弊条"
date: 2021-02-14T09:23:51+08:00
draft: false
tags: ["switch"]
---



笔者近日折腾了于18年购入后只玩了BOTW就一直吃灰的switch一台，从目前联网可更新的最新系统11.0.1开始进行大气层破解，书写作弊条以记录。

- 初始化你的switch（如果没有需要保留的存档的话）。

- 准备一张TF卡，格式化成exFAT格式，直接插入switch，确认可以正常识别，长按电源键把switch关机。

- 使用读卡器加载TF卡，下载最新的大气层系统文件(别忘了下载`fusee-primary.bin`)：https://github.com/Atmosphere-NX/Atmosphere/releases， hekate（bootloader）https://github.com/CTCaer/hekate/releases/tag/v5.5.4-v2 ，还有sigpatches文件（用于运行非官方发行的nsp文件，缺失运行部分游戏会报错）：https://github.com/eXhumer/patches/releases，还有dbi（mtp工具）：https://github.com/rashevskyv/dbi/releases， lockpick (用于备份key文件，可选)：https://github.com/shchmue/Lockpick_RCM/releases。

- 把大气层系统与sigpatches和hekate里的bootloader文件夹复制到tf卡。

- 把 `fusee-primary.bin` 复制到大气层目录。

- 把下面内容保存为`hekate_ipl.ini`复制到bootloader文件夹。

  ```
  {------ Atmosphere ------}
  {Pick this option to launch CFW.}
  [Atmosphere]
  payload=atmosphere/fusee-primary.bin
  { }
  {------ Stock ------}
  {NOTE: This option does not launch CFW.}
  [Stock]
  fss0=atmosphere/fusee-secondary.bin
  stock=1
  { }
  ```

  

- 插入rcm短接器，按音量+与电源键启动rcm模式，用`TegraRcmGUI` 或者 `nxloaderRB` 等工具引导进入hekate（可先引导进入lockpick备份key，可选）。

- 把能备份的文件全部备份，然后用读卡器导入电脑。防止任何可能出现的意外。

- 制作虚拟系统。

- 使用`NxNandManager`瘦身虚拟系统。

  下面是一些常用的自制程序：

  [https://github.com/rashevskyv/dbi](https://github.com/rashevskyv/dbi)

  [https://github.com/WerWolv/Hekate-Toolbox](https://github.com/WerWolv/Hekate-Toolbox)

  [https://github.com/J-D-K/JKSV](https://github.com/J-D-K/JKSV)

  [https://github.com/joel16/NX-Shell](https://github.com/joel16/NX-Shell)

