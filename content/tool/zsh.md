---
title: Oh My Zsh
date: 2016-07-26 16:54
---

[TOC]

# 0x00 安装
> 环境：ubuntu 14.04 x86-64

```shell
// 1. 安装Zsh
sudo apt-get install zsh
// 2. 安装oh-my-zsh，权限问题，请加sudo在最前面
sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
// 3. 改变默认shell
sudo chsh -s $(which zsh)
// 4. 不知道什么原因，我无法通过这个命令更改shell，所以去直接修改/etc/passwd
// 5. 注销再登陆
```

# 0x01 配置
> 主要是换个主题，花哨一点的。

```shell
// 1. 更改主题
vim ~/.zshrc
// 2. 找到ZSH_THEME，我将它修改为agnoster，因为字体问题有乱码，其实就是无法正常显示一些字符
```
# 0x02 乱码问题
```shell
// 1. 下载字体
// 地址：https://github.com/powerline/fonts
// 2. 解压并运行
./install.sh
// 3. 修改ubuntu的terminal默认配置，选择字体中带有"derivative Powerline"的就可以了
```
