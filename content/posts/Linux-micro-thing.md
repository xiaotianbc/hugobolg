---
title: Linux相关私货
date: 2019-12-31 21:11:43
tags:
---
仅仅是为了方便记忆，一般都为debian9+使用。  

依赖

``` shell
apt update
apt install -y vim zsh htop curl git wget unzip screen
```

安装ssh-key

``` shell
echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDQ4YL6VNGhNDFhj/CJbmtUpcTFpyC2YqK19L4dAbTvtsPog3OgqNkdLJnxL6dONqucnrusoOykAI3/5dwHIT5IXHTkye4pEywHAbZBNES7ZGitZgCbmpMhmaecz9ZE3mGeSBkOqYDho33uH5xT9O0AU0pgLRo7BO//ae+gnsH1WEkbK4y0a+typw9QcAupTi+wmfg/w+u6zYGiB3CPZhNqk5SEBvt9WI4cDbcovCTCfhsu1Ty6tD2tCEHeBRzd9UVlZfDpY/dBOcCbBQVEU2Zf1sQos0lkEjcV77oh5REtrha3DojIqZqYSWw+l7cKny6u6Z4W5O/IIVNsS5Tda3oN canxuetian@gmail.com" > ~/.ssh/authorized_keys


```

禁用密码登录

``` shell
vim /etc/ssh/sshd_config

PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
PasswordAuthentication no

```

删除登陆信息

``` shell
touch ~/.hushlogin
```

besttrace

``` shell
wget https://cdn.ipip.net/17mon/besttrace4linux.zip
unzip be*
chmod +x besttrace
mv besttrace /usr/local/bin/btr
rm -rf ./besttrace*
```

zsh

``` shell
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
ZSH_THEME="agnoster"
```

ssh心跳

``` shell
echo "ServerAliveInterval 30\n" >> /etc/ssh/ssh_config
echo "ServerAliveCountMax 60\n" >> /etc/ssh/ssh_config
```

Nodejs 13  

``` shell
curl -sL https://deb.nodesource.com/setup_13.x | bash -
apt-get install -y nodejs
```

node-gyp出现错误  

``` shell
sudo xcode-select -s /Applications/Xcode.app/Contents/Developer
```

macOS相关

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
