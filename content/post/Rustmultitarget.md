---
title: "Rust跨平台编译"
date: 2020-03-03T11:23:51+08:00
draft: false
tags: ["rust"]
---

首先我们要知道，对于Rust这种语言来说，需先编译再链接：

## 安装编译器

```shell
$ rustup target list
```

可以看到列出的项目，如果需要`aarch64-unknown-linux-gnu`,可以使用下面的命令安装：

```shell
$ rustup target add aarch64-unknown-linux-gnu
```

## 安装链接器

安装完成之后需要安装链接器，一般可以使用gcc，但是我发现gcc的官网只提供了二进制的windows和linux下的工具链。  
还好在github上找到了macOS下的可用的二进制包：`https://github.com/thinkski/osx-arm-linux-toolchains/releases`  
下载解压后，把所在文件位置添加到path即可。

```shell
#~/.zshrc
export PATH=$PATH:/Users/lei/sdk/aarch64-unknown-linux-gnu/bin
```

## 用Cargo编译其他平台软件

在cargo项目下添加一个文件 `.cargo/config`：

```toml
#.cargo/config

[target.aarch64-unknown-linux-gnu]
linker = "aarch64-unknown-linux-gnu-gcc"
ar = "aarch64-unknown-linux-gnu-ar"
```

然后使用：`cargo build --release --target="aarch64-unknown-linux-gnu"`即可完成。

另外如果要减小生成的二进制文件大小的话可以开启lto优化，代价是牺牲了编译时间：

```toml
#Cargo.toml

[profile.release]
lto = true
```
