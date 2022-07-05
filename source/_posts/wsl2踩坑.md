---
title: wsl踩坑
date: 2021-02-07 22:46:06
tags:
---

本文针对的是使用 wsl2 的开发者，记录的是我在使用 wsl 的过程中遇到的一些坑。欢迎大家一起来补充

## wsl hot reload not work

https://github.com/microsoft/WSL/issues/4417#issuecomment-526279339

注：在 wsl2 中，如果打开的文件是在 /mnt 下（也就是在Windows中下载），是无法 hot reload 的，可以在 linux 子系统中下载项目

## wsl terminal .bashrc file

Windows 和 wsl2 的命令行使用的 bashrc 不是同一个文件。Windows 的是在 Windows 的用户目录下，wsl 在 `~/.bashrc`。这两个不是一个位置。

## wsl install node

https://github.com/nodejs/help/wiki/Installation#how-to-install-nodejs-via-binary-archive-on-linux

## Windows browse linux file system

- 可以通过在地址栏中输入 `\\wsl$` 来访问

- 通过 wsl --mount 生成(preview 中，持续关注), https://docs.microsoft.com/en-us/windows/wsl/wsl2-mount-disk

## open current folder in terminal

```
explorer.exe .
```

为了方便，可以设置 alias

```
alias explorer="explorer.exe ."
```

## yarn global add not work

https://stackoverflow.com/a/40333409

You should add `export PATH="$PATH:$(yarn global bin)"` to your ~/.bash_profile or whatever you use.


## wsl vmmem 进程内存占用过大

我开了4个VSCode，结果电脑内存直接炸了，一查，vmmem进程占了5G

https://blog.n0ts.cn/1155.html

win+R,输入`%UserProfile%`，会打开一个文件夹，在其中新建一个`.wslconfig`,其中输入一下内容

```
[wsl2]
memory=4GB
swap=2GB
localhostForwarding=true
```

memory为系统内存上限，可根据实际情况进行配置

运行`wsl --shutdown`命令，关闭wsl的运行

然后运行`wsl`命令，重启wsl

## wsl2 set proxy

因为 shadowSocks 不是运行在 wsl2 本地，所以得通过 IP 访问 Windows。Windows 的 IP 对于 wsl2 是动态的，可以通过一个脚本来获取。
在 `~/.bashrc` 中加入以下配置。
注意，在代理软件中的设置中打开 **允许来自局域网的链接** 选项

```
export HOST_IP=$(grep -oP '(?<=nameserver\ ).*' /etc/resolv.conf)
export PROXY_ADDR="http://$HOST_IP:1080"
yarn config set proxy "$PROXY_ADDR"
yarn config set https-proxy "$PROXY_ADDR"
npm config set proxy "$PROXY_ADDR"
npm config set https-proxy "$PROXY_ADDR"
export http_proxy=$PROXY_ADDR
export https_proxy=$PROXY_ADDR
```

如果，还是无法成功连接，可能是因为 Windows 的防火墙的原因。
按 win 按钮打开开始菜单，搜索 允许应用通过 Windows Defender 防火墙。更改其中的配置，寻找你的代理程序（对我是 ShadowSockR)，把所有的勾都打上。应该就可以解决了。

## 局域网访问

自动化脚本映射端口（需提前配置需要映射的端口）：https://github.com/microsoft/WSL/issues/4150#issuecomment-504209723

直接的命令：

```
# $addr: 0.0.0.0, $remoteport: wsl2 的内网地址，可通过 ipconfig 查看
netsh interface portproxy add v4tov4 listenport=$port listenaddress=$addr connectport=$port connectaddress=$remoteport
```

## 解除 windows defender 影响

https://github.com/microsoft/WSL/issues/4139#issuecomment-732067409
