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
