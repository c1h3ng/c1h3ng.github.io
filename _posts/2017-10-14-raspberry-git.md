---
layout: post
title: "树莓派学习笔记——搭建私有Git服务器"
date: 2017-10-14
tags:
  git
  Raspberrypi
categories:
  Raspberrypi
---
# 0x01 Git
Git是一款免费开源的分布式版本控制系统，用于敏捷高效地处理或大或小的项目版本管理，是Linux之父Linus Torvalds为了帮助管理Linux内核开发而开发的一个开放源码的版本控制软件。
# 0x02 树莓派搭建git私有服务器
首先需要在树莓派上安装git:
```bash
pi@raspberrypi:~$ sudo apt-get install git
```
修改主机名：
```bash
pi@raspberrypi:~$ sudo vim /etc/hostname
//修改主机名
pi@raspberrypi:~$ sudo vim /etc/hosts
//替换已修改的主机名
```
我的主机名为raspberrypi，没必要修改了，如果觉得不方便可以修改，以后ssh连接时就不需要再记一串ip地址了。
然后添加一个git用户和用户组：
```bash
pi@raspberrypi:~$ sudo adduser git
```
在pc上生成ssh公钥私钥实现免密使用git服务：
```bash
root@kali:~# ssh-keygen -t rsa -C git@raspberrypi -f ~/.ssh/rasp-rsa
//生成密钥
root@kali:~# ssh-add ~/.ssh/rasp_rsa
//添加ssh私钥
pi@raspberrypi:~$ sudo vim /home/git/.ssh/authorized_keys
//添加pc上生成的rasp_ras.pub公钥
```
若有其他需要使用git的主机，一并导入authorized_keys中

# 0x03 禁止git用户登录bash shell
出于安全性考虑，防止授权用户随意修改git用户工作区，需要像github那样禁用git用户登陆使用bash shell：
```bash
pi@raspberrypi:~$ sudo vim /etc/passwd
git:x:112:118:,,,:/home/git:/bin/bash
修改为：
git:x:112:118:,,,:/home/git:/usr/bin/git-shell
```
现在pc上想ssh远程登录会有如下效果：
![](https://github.com/c1h3ng/c1h3ng.github.io/blob/master/assets/images/gitshell.png?raw=true)

报错提示git家目录下没有git-shell-commands目录，解决办法：
```bash
pi@raspberrypi:/home/git $ mkdir git-shell-commands
pi@raspberrypi:/home/git $ cd git-shell-commands
pi@raspberrypi:/home/git/git-shell-commands $ vim no-interactive-login
#!/bin/sh
printf '%s\n' "Hi $USER! You've successfully authenticated, but I do not"
printf '%s\n' "provide interactive shell access."
exit 128
pi@raspberrypi:/home/git/git-shell-commands $ chown git:git -R ../
```
再次尝试登录：
![](https://github.com/c1h3ng/c1h3ng.github.io/blob/master/assets/images/git-shell.png?raw=true)

不会影响ssh传输：
![](https://github.com/c1h3ng/c1h3ng.github.io/blob/master/assets/images/clone.png?raw=true)
