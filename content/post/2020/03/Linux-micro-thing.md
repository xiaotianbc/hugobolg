---
title: Linux相关命令记录
date: 2019-12-31 21:11:43
tags: ["linux"]
---
仅仅是为了方便记忆，一般都为 ~~ArchLinux~~ debian10使用。

## 依赖

``` shell
apt update
apt install -y vim zsh htop curl git wget unzip screen
```

## 安装ssh-key

``` shell
echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDQ4YL6VNGhNDFhj/CJbmtUpcTFpyC2YqK19L4dAbTvtsPog3OgqNkdLJnxL6dONqucnrusoOykAI3/5dwHIT5IXHTkye4pEywHAbZBNES7ZGitZgCbmpMhmaecz9ZE3mGeSBkOqYDho33uH5xT9O0AU0pgLRo7BO//ae+gnsH1WEkbK4y0a+typw9QcAupTi+wmfg/w+u6zYGiB3CPZhNqk5SEBvt9WI4cDbcovCTCfhsu1Ty6tD2tCEHeBRzd9UVlZfDpY/dBOcCbBQVEU2Zf1sQos0lkEjcV77oh5REtrha3DojIqZqYSWw+l7cKny6u6Z4W5O/IIVNsS5Tda3oN canxuetian@gmail.com" > ~/.ssh/authorized_keys


```

## 禁用密码登录

``` shell
vim /etc/ssh/sshd_config

PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
PasswordAuthentication no
PermitRootLogin without-password
```

### 删除登陆信息

``` shell
touch ~/.hushlogin
```

## besttrace

``` shell
wget https://78997899.xyz/od/Linux/besttrace4linux.zip
unzip be*
chmod +x besttrace
mv besttrace /usr/local/bin/btr
rm -rf ./besttrace*
```

## zsh

``` shell
sh -c "$(wget https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"
#linux
curl -fsSL https://starship.rs/install.sh | bash
#mac
brew install starship
# ~/.zshrc
eval "$(starship init zsh)"
```

## LinuxBrew

```shell
sudo apt-get install build-essential gcc
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
echo 'eval $(/home/linuxbrew/.linuxbrew/bin/brew shellenv)' >> ~/.zshrc
```
## ssh心跳

``` shell
echo "ServerAliveInterval 30\n" >> /etc/ssh/ssh_config
echo "ServerAliveCountMax 60\n" >> /etc/ssh/ssh_config
```

## Rust

```shell
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
export CARGO_HOME="$HOME/.cargo"
export RUSTUP_HOME="$HOME/.rustup"
export PATH=$PATH:$CARGO_HOME/bin:$RUSTUP_HOME


#export RUSTUP_DIST_SERVER=https://mirrors.ustc.edu.cn/rust-static
#export RUSTUP_UPDATE_ROOT=https://mirrors.ustc.edu.cn/rust-static/rustup
```

## adoptJDK

``` shell
wget https://github.com/AdoptOpenJDK/openjdk8-binaries/releases/download/jdk8u242-b08/OpenJDK8U-jdk_x64_linux_hotspot_8u242b08.tar.gz
tar -C /usr/local -zxf ./OpenJDK8U-jdk_x64_linux_hotspot_8u242b08.tar.gz
rm ./OpenJDK8U-jdk_x64_linux_hotspot_8u242b08.tar.gz
# .zshrc
export JAVA_HOME=/usr/local/jdk8u242-b08
export PATH=$PATH:$JAVA_HOME/bin
```

## Nodejs 13

``` shell
curl -sL https://deb.nodesource.com/setup_13.x | bash -
apt-get install -y nodejs
```

## node-gyp出现错误

``` shell
sudo rm -rf $(xcode-select -print-path)
xcode-select --install
```

## Arch 踩坑

```shell
pacman -S archlinux-keyring
vim /etc/locale.gen
# 取消en_US.UTF8前面的注释
locale-gen
echo 'LANG=zh_CN.UTF-8'  > /etc/locale.conf
```

## 修改时区

``` shell
timedatectl set-timezone Asia/Shanghai
```

## macOS相关

``` shell
#Show Library folder
chflags nohidden ~/Library
#Show hidden files
#This can also be done by pressing command + shift + ..

defaults write com.apple.finder AppleShowAllFiles YES
#Show path bar
defaults write com.apple.finder ShowPathbar -bool true
#Show status bar
defaults write com.apple.finder ShowStatusBar -bool true
#Disable unidentified developer warnings
sudo spctl --master-disable
```

## Linux创建sudo权限用户

```shell
sudo adduser foo
sudo usermod -a -G sudo foo
#取消sudo权限
sudo deluser foo sudo
```

## 修改为zh_CN.UTF-8

```shell
vim /etc/default/locale
LANGUAGE=zh_CN.UTF-8
LC_MESSAGES=zh_CN.UTF-8
LANG=zh_CN.UTF-8
```
