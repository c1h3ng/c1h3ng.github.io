---
layout: post
title: "树莓派学习笔记——配置树莓派"
date: 2017-9-24
tags:
  Linux
  Raspberrypi
categories:
  Raspberrypi
---
大三了，专业核心课程变成了嵌入式相关的一些课，听的一脸懵逼，课还没上多久老师就要求报名参加嵌入式设计大赛，还有学分，~~太过分了(摔鼠标)，羡慕会安卓的室友，都不需要对硬件有多了解而且还算是嵌入式...~~为了拿到学分，只能逼自己弄点东西出来了，所以我想到了树莓派，一块信用卡大小的计算机，麻雀虽小五脏俱全，并且还是基于ARM架构的主板，那么也就算是特殊的嵌入式开发板咯

![Raspberrypi](https://c1h3ng.github.io/assets/images/rpi.jpg)

# 0x01 加装外壳和风扇
树莓派买来只是一块主板，提供了USB,HDMI,以太网等接口，以及提供电源用的micro usb接口(安卓)，外存使用的SD卡槽，看着很简陋，并且没有任何保护硬件的措施，首先给刚到手的树莓派加装一个壳以及散热的风扇，东西可以直接网购入手，如下：

![shell](https://github.com/c1h3ng/c1h3ng.github.io/blob/master/assets/images/shell.jpg?raw=true)

按照商家给的安装说明安装即可， 注意下风扇红线黑线不要插错或插反针脚就行了：

![setup](https://github.com/c1h3ng/c1h3ng.github.io/blob/master/assets/images/setup.jpg?raw=true)

# 0x02 安装系统
树莓派官方推出系统的是基于debian的raspbian，当然还有很多其他的系统，可以参考这个链接：[https://en.wikipedia.org/wiki/Raspberry_Pi#Software](https://en.wikipedia.org/wiki/Raspberry_Pi#Software)
这里我们选择raspbian，毕竟是为树莓派量身定制的系统，首先准备了一张64G的SD卡以及读卡器，插入windows用diskgenius格式化成FAT32的文件系统，在官网下载镜像，我下载的是NOOBS，是树莓派官方发布的工具，是一种很方便的树莓派设置程序，可以快速简单的安装raspbian，下载链接：[NOOBS](https://www.raspberrypi.org/downloads/noobs/)，选择左边的，下载zip即可，右边的lite为精简版，下载完成后解压到已经格式化好的SD卡的根目录下，再将SD卡插入树莓派，插上电源，接上显示器即可进入图形化安装界面：

![noobs](https://raw.githubusercontent.com/c1h3ng/c1h3ng.github.io/master/assets/images/noobs.jpg)

然后就是漫长的安装过程，
当然也可以选择直接在官网下载[raspbian的镜像](https://www.raspberrypi.org/downloads/raspbian/)，然后使用win32 DiskImager将解压出的img镜像文件烧录到SD卡中，再给树莓派通电即可进入系统

![pi](https://raw.githubusercontent.com/c1h3ng/c1h3ng.github.io/master/assets/images/pi.jpg)

# 0x03 初始配置
为了今后的方便，需要把SSH打开，并且让树莓派默认使用文本模式，就不再需要显示器了，raspbian中设置非常方便，在图形化界面菜单栏 > Preferences > Raspberry Pi Configuration中可设置树莓派默认开启SSH以及默认使用文本模式(禁用图形化界面)

![config](https://raw.githubusercontent.com/c1h3ng/c1h3ng.github.io/master/assets/images/config.jpg)

![cli](https://raw.githubusercontent.com/c1h3ng/c1h3ng.github.io/master/assets/images/cli.jpg)

设置好后reboot，进入就是文本模式，以及默认开启SSH，现在就可以通过SSH远程登录了：

![wenben](https://raw.githubusercontent.com/c1h3ng/c1h3ng.github.io/master/assets/images/wenben.jpg)

![ssh](https://raw.githubusercontent.com/c1h3ng/c1h3ng.github.io/master/assets/images/ssh.png)

ssh远程登录使用vi会出现，上下左右退格键没法用，所以我们需要下载vim
```bash
pi@raspberrypi:~ $ sudo apt-get install vim
```
下载完成后，还是把源换成国内源，速度可观一点
```bash
pi@raspberrypi:~ $ sudo vim /etc/apt/sources.list
deb http://mirrors.163.com/debian/ wheezy main non-free contrib
deb http://mirrors.163.com/debian/ wheezy-updates main non-free contrib
deb http://mirrors.163.com/debian/ wheezy-backports main non-free contrib
deb-src http://mirrors.163.com/debian/ wheezy main non-free contrib
deb-src http://mirrors.163.com/debian/ wheezy-updates main non-free contrib
deb-src http://mirrors.163.com/debian/ wheezy-backports main non-free contrib
deb http://mirrors.163.com/debian-security/ wheezy/updates main non-free contrib
deb-src http://mirrors.163.com/debian-security/ wheezy/updates main non-free contrib
#deb http://mirrordirector.raspbian.org/raspbian/ stretch main contrib non-free rpi
# Uncomment line below then 'apt-get update' to enable 'apt-get source'
#deb-src http://archive.raspbian.org/raspbian/ stretch main contrib non-free rpi
```
# 0x04 vim和bash
这里还是直接套用我kali里的配置：
```bash
pi@raspberrypi:~ $ sudo vim .bashrc //只能修改pi用户的bash配置
//添加
alias ll='ls -al'
alias rm='rm -i'
pi@raspberrypi:~ $ sudo vim /etc/vim/vimrc //所有用户的vim配置在/etc/vim/vimrc中
//添加
syntax on "自动语法高亮
set number "显示行号
set cursorline "突出显示当前行
set ruler "打开状态栏标尺
set softtabstop=4 "退格键可以删除4个空格
set tabstop=4 "tab长度为4
set ignorecase smartcase "搜索时忽略大小写
set nowrapscan "禁止在搜索到文件两端时重新搜索
set incsearch "输入搜索内容就显示搜索结果
set hlsearch "搜索时高亮显示被找到的文本
set smartindent "开启新行时使用智能自动缩进
set mouse=a "开启鼠标光标定位
```
对树莓派最基本的配置就这样了，后面就可以用树莓派干点有意思的事了_(:Ⅰ」∠)_
