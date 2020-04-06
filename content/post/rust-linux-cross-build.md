---
title: "Mac上为Linux交叉编译Rust程序"
date: 2020-03-28T12:23:51+08:00
draft: false
tags: ["rust"]
---

目标是创建 x86_64-unknown-linux-musl 平台的应用程序，通过静态连接musl, 不再依赖glibc库。

``` shell
brew install FiloSottile/musl-cross/musl-cross
rustup target add x86_64-unknown-linux-musl # First time only
cargo build --release --target x86_64-unknown-linux-musl
```

你还需要创建musl-gcc:

``` shell
ln -s /usr/local/bin/x86_64-linux-musl-gcc /usr/local/bin/musl-gcc
```

你还需要设置linker, 创建.cargo/config,增加下面的两行：

``` shell
[target.x86_64-unknown-linux-musl]
linker = "x86_64-linux-musl-gcc"
```

然后运行下面的命令进行交叉编译：

``` shell
CROSS_COMPILE=x86_64-linux-musl- cargo build --release --target x86_64-unknown-linux-musl
```
