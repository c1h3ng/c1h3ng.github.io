---
layout: post
title: "树莓派学习笔记——打造FC卡带游戏机"
date: 2017-10-4
tags:
  Linux
  Raspberrypi
  Lakka
categories:
  Raspberrypi
---
# 0x01 准备
制作FC卡带游戏机我们需要准备以下东西
```
显示器
VGA转HDMI转接头
手机充电器(5v==2A)
一张 micro SD 卡(8G以上，这里我用的是32G的)
读卡器
一台windows系统电脑(用于格式化SD卡及烧录镜像)
第三代树莓派
有线USB手柄(即插即用，无需下载驱动)
键盘
```
东西准备好就可以开始了
# 0x02 正片
SD卡是我从旧手机里抽出来的，里面还有数据，所以我们需要格式化，插入读卡器再插入电脑，格式化为FAT32文件系统，如果windows本身的磁盘格式化工具没有FAT32这个选项，可以下载一个diskgenius格式化SD卡

制作游戏机我们当然需要一个强大的游戏模拟器，这里我们使用lakka，lakka支持很多游戏，经典的红白机，街机，甚至是playstation的游戏，我们先去官网下载镜像：[Lakka官网](http://www.lakka.tv/get/)
这里根据自身情况选择操作系统平台，硬件平台，这里用到的是树莓派所以选择：GNU/Linux -> Raspberry Pi 3，然后下载镜像

下载完成后，解压出img镜像，windows下我们使用win32diskImager来烧录镜像到SD卡中：

![](https://github.com/c1h3ng/c1h3ng.github.io/blob/master/assets/images/lakka.png?raw=true)

win32diskImager选择镜像和SD卡：

![](https://github.com/c1h3ng/c1h3ng.github.io/blob/master/assets/images/win32disk.png?raw=true)

选择写入，然后等待一会就烧录到SD卡中了：

![](https://github.com/c1h3ng/c1h3ng.github.io/blob/master/assets/images/write.png?raw=true)

![](https://github.com/c1h3ng/c1h3ng.github.io/blob/master/assets/images/success.png?raw=true)

镜像烧录完成，现在拔出SD卡，插入树莓派中，给树莓派通上电，第一次需要等待片刻，初始化完成后会自动reboot，之后就可以使用了：

![](https://github.com/c1h3ng/c1h3ng.github.io/blob/master/assets/images/boot.jpg?raw=true)

BUT，现在还没有游戏可以玩，只能玩一个2048：

![](https://github.com/c1h3ng/c1h3ng.github.io/blob/master/assets/images/2048.jpg?raw=true)

我们先连上wifi，当然也可以直接使用网线，Settings -> Wi-Fi，选择wifi输入密码即可：

![](https://github.com/c1h3ng/c1h3ng.github.io/blob/master/assets/images/wifi.jpg?raw=true)

连上之后现在树莓派就与PC在同一局域网中了，此时我们可以在PC中的网络中可以发现名为lakka的设备，其中ROMs就是我们要用来存放游戏的目录，到网上下载游戏放到目录下，在lakka中扫描ROMs目录即可扫描到游戏：

![](https://github.com/c1h3ng/c1h3ng.github.io/blob/master/assets/images/network.jpg?raw=true)

# 0x03 彩蛋
这里将游戏合集分享出，需要的下载即可：[百度网盘]( https://pan.baidu.com/s/1kVAAAwR) | 密码: 8ybw

解压到ROMs目录下后，在lakka中切换到ROMs目录选择scan directory，等待扫描完后，就会出现一个能运行的游戏列表了，然后就可以愉快的玩耍了：

![](https://github.com/c1h3ng/c1h3ng.github.io/blob/master/assets/images/supermario.jpg?raw=true)
