---
title: Linux相关私货
date: 2019-12-31 21:11:43
tags:
---
仅仅是为了方便记忆，一般都为debian10使用。  

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
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git $ZSH_CUSTOM/themes/powerlevel10k
# .zshrc
ZSH_THEME="powerlevel10k/powerlevel10k"
```

## ssh心跳

``` shell
echo "ServerAliveInterval 30\n" >> /etc/ssh/ssh_config
echo "ServerAliveCountMax 60\n" >> /etc/ssh/ssh_config
```

## Go

```shell
wget https://dl.google.com/go/go1.13.7.linux-amd64.tar.gz
tar -C /usr/local -xzf go*
#.zshrc
export GOPATH=$HOME/go
export PATH=$PATH:/usr/local/go/bin:$GOPATH/bin
```

## Rust

```shell
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
export CARGO_HOME="/root/.cargo"
export RUSTUP_HOME="/root/.rustup"
export PATH=$PATH:/usr/local/go/bin:/root/go/bin:$CARGO_HOME/bin:$RUSTUP_HOME


#export RUSTUP_DIST_SERVER=https://mirrors.ustc.edu.cn/rust-static
#export RUSTUP_UPDATE_ROOT=https://mirrors.ustc.edu.cn/rust-static/rustup
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

### IpV6 访问github?

```conf
#hosts
2a04:4e42::133 assets-cdn.github.com 2a04:4e42::133
camo.githubusercontent.com 2a04:4e42::133 cloud.githubusercontent.com
2a04:4e42::133 gist.githubusercontent.com 2a04:4e42::133
avatars.githubusercontent.com 2a04:4e42::133
avatars0.githubusercontent.com 2a04:4e42::133
avatars1.githubusercontent.com 2a04:4e42::133
avatars2.githubusercontent.com 2a04:4e42::133
avatars3.githubusercontent.com 2a04:4e42::133
marketplace-images.githubusercontent.com 2a04:4e42::133
user-images.githubusercontent.com 2a04:4e42::133
raw.githubusercontent.com
```