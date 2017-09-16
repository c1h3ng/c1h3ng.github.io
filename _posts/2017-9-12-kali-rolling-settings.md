---
layout: post
title: "填坑笔记之折腾kali-win10双系统"
date: 2017-9-12
tags:
  kali
  Linux
categories:
  Linux
---
暑假期间折腾双系统折腾了几天，本来都定制好了，开学后回到实验室，连上自己的显示器想双屏爽一爽...结果系统根本检测不到扩展显示器，搞了半天确定是因为nvidia显卡驱动的问题，~~一顿骚操作，直接把图形化桌面搞崩了，又一顿骚操作直接还原不回去了~~，整个人头都大了，只能重新装，再折腾一次了，记录下来，感觉Linux还玩不转，以后很可能再次被骚操作搞坏...
# 安装
我使用的是kali rolling 2017.1的镜像，首先在win10下磁盘管理器中格式化一个盘(220G)，删除分区，然后分出120G做linux的系统盘，剩下的还是挂载到windows上，使用ultraiso将iso镜像文件刻录进事先准备好的8G优盘中，启动盘做好，重启选择U盘启动，选择图形化安装，如果提示光盘无法挂载，拔下U盘再插上点击重试即可，为了避免以后出现一些头疼的问题，语言还是选择了英文，其他的没什么值得说的，然后是分区，选中之前分好的硬盘分区，选择ext4文件系统，然后等待安装，之后安装linux的grub引导，我选择覆盖了windows的MBR引导，至此安装就结束了。
# 显卡驱动(第一个坑)
经过丝滑般顺畅的安装流程后，迫不及待想看见gnome桌面扁平清爽的界面，输入帐号密码，然后...就卡住了，重启，发现还是能进命令行的，就是进不了图形化界面，看来即使重装也还是不能避免显卡驱动这个蛋疼问题...

我的显卡是intel核显和nvidia gtx 960M的独显，第九代，还算是比较新的系列，nvidia官方提供的驱动程序是闭源的，所以很多linux发行版都集成了由第三方开发的nouveau显卡驱动，nouveau是由第三方为nvidia显卡开发的一个开源显卡驱动，但并没有得到英伟达官方的支持和认可，并且对较新的N卡支持很差，所以还是应该去官方下载适合自己显卡型号的驱动，总之，**nouveau这个东西太坑了**...

接下来要做的就是禁用nouveau驱动，重启，在grub引导界面按e进入编辑模式，在倒数第三行添加

>nouveau.modeset=0

![kali](https://c1h3ng.github.io/assets/images/kali.jpg)

F10启动，忐忑的输完帐号密码终于进去了，不过这样需要每次在引导界面修改，很麻烦，所以进入系统后先将nouveau加入黑名单
```
root@kali:# cd /etc/modprobe.d
root@kali:/etc/modprobe.d# vim nvidia-graphics-drivers.conf //写入 blacklist nouveau
root@kali:/etc/modprobe.d# cd /etc/default
root@kali:/etc/default# vim grub //写入 rdblacklist=nouveau nouveau.modeset=0
```

这样之后问题就彻底解决了，再去英伟达官网下载合适的驱动即可，现在终于能使用双屏了

![screen](https://c1h3ng.github.io/assets/images/screen.jpg)

# 配置源及更新
首先第一件事是更换apt源，向 /etc/apt/sources.list 添加中科大的kali rolling源，然后更新
```
root@kali:# cat /etc/apt/sources.list
# deb cdrom:[Debian GNU/Linux 2017.1 _Kali-rolling_ - Official Snapshot amd64 LIVE/INSTALL Binary 20170416-02:08]/ kali-rolling contrib main non-free

#deb cdrom:[Debian GNU/Linux 2017.1 _Kali-rolling_ - Official Snapshot amd64 LIVE/INSTALL Binary 20170416-02:08]/ kali-rolling contrib main non-free
deb http://mirrors.ustc.edu.cn/kali kali-rolling main non-free contrib
deb-src http://mirrors.ustc.edu.cn/kali kali-rolling main non-free contrib
#官方源
#deb http://http.kali.org/kali kali-rolling main non-free contrib
#deb-src http://http.kali.org/kali kali-rolling main non-free contrib
root@kali:#apt-get update && apt-get upgrade && apt-get dist-upgrade //更新软件索引文件然后根据依赖关系更新软件包
```
接下来就是漫长的等待，其他细节就不记录了。
# 简单配置bash和vim
接下来就是根据个人习惯配置bash，首先打开终端，选择Edit->Profile Preferences->Colors->Built-in schemes，我喜欢黑底绿字，就改成黑底绿字：

![terminal](https://c1h3ng.github.io/assets/images/terminal.png)

```
//可根据自身需求定制
root@kali:# cd ~
root@kali:~# vim .bashrc
//为了方便操作和防止一些危险操作造成无法挽回的局面，添加如下内容
alias ll='ls -al'
alias rm='rm -i'
//保存退出
root@kali:~# source .bashrc //立即生效
```
接下来配置vim，~~文本编辑器中最经典生命力最强的上古神奇，咳咳，这里就不纠结了，反正我不用vim写代码，~~简单配置下能方便的进行最简单的编辑即可:
```
root@kali:# cd /etc/vim
root@kali:/etc/vim# ls
gvimrc  php_funclist.txt  vimrc  vimrc.tiny
root@kali:/etc/vim# vim vimrc
//添加如下内容
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
set mouse=a "开启鼠标定位光标
```
![vim](https://c1h3ng.github.io/assets/images/vim.png)

效果如上，简单配置下已经足够日常使用了。
# 桌面美化及软件下载
gnome扁平化，清爽的界面，以及kali默认的主题我认为已经很符合我的审美了，有需要的话可以换张好看的壁纸即可，另外有强迫症的我完全不能忍受dash to dock在桌面的左边，在dash to dock settings中将dock放置到底部，现在看着舒服多了(逃)

![desktop](https://c1h3ng.github.io/assets/images/desktop.png)
## 输入法
安装fcitx框架及google拼音：
```
root@kali:~# apt-get install fcitx && fcitx-googlepinyin
root@kali:~# reboot
```
重启后桌面左下角出现一个任务栏，右键输入法的键盘图标->configure，将google拼音添加至输入法中，再根据自身需要定制热键。

![fcitx](https://raw.githubusercontent.com/c1h3ng/c1h3ng.github.io/master/assets/images/fcitx.png)
## 网易云音乐
进入[网易云音乐官网](http://music.163.com/#/download)下载，我选择的是ubuntu16.04 64位的deb包，没试过deepin的deb包。
```
root@kali:~/Downloads# dpkg -i netease-cloud-music_1.0.0-2_amd64_ubuntu16.04.deb
//安装网易云音乐deb包，如果出现依赖问题，执行如下命令
root@kali:~/Downloads# apt-get -f install
//安装完依赖之后，再次执行第一条命令即可
```
安装完后在dock中可以找到网易云音乐的快捷方式，如果无法打开网易云音乐，在快捷方式中禁用沙箱(*可能会存在安全隐患*)，~~貌似安装32位网易云就不会出现这种情况？~~
```
root@kali:~# cd /usr/share/applications
root@kali:~# vim netease-cloud-music.desktop
//在Exec后面加上参数 --no-sandbox
[Desktop Entry]
Version=1.0
Type=Application
Name=NetEase Cloud Music
Name[zh_CN]=网易云音乐
Name[zh_TW]=網易雲音樂
Comment=NetEase Cloud Music
Comment[zh_CN]=网易云音乐
Comment[zh_TW]=網易雲音樂
Icon=netease-cloud-music
Exec=netease-cloud-music %U --no-sandbox
Comment=NetEase Cloud Music
Categories=AudioVideo;Player;
Terminal=false
StartupNotify=true
MimeType=audio/aac;audio/flac;audio/mp3;audio/mp4;audio/mpeg;audio/ogg;audio/x-ape;audio/x-flac;audio/x-mp3;audio/x-mpeg;audio/x-ms-wma;audio/x-vorbis;audio/x-vorbis+ogg;audio/x-wav;
```
## 微信
Linux下需要一个通讯软件，但是QQ实在是太坑了~~(羡慕deepin)~~，要么没法运行，要么兼容性很差，不到一分钟就闪退，要不就是字符乱码，放弃QQ使用微信算了，electronic wechat是github上的一个项目，非官方的微信，调用了网页微信的接口，登录还是需要扫描二维码，由于客户端基于electronic技术开发，跨平台性相当不错，支持Mac Linux Windows。
>项目地址：https://github.com/geeeeeeeeek/electronic-wechat/releases


我选择的是已编译好的打包文件下载，解包后wechat目录下有一个可以启动客户端文件，但是没有快捷方式，我们还需要创建一个快捷方式：
```
root@kali:~# cd /usr/share/applications
root@kali:~# vim wechat.desktop
//内容如下，根据自己的情况修改
[Desktop Entry]

Name=Wechat

Comment=wechat

Exec=/root/Downloads/electronic-wechat-linux-x64/electronic-wechat

Icon=/root/Downloads/electronic-wechat-linux-x64/weixin4.png

Terminal=false

StartupNotify=true

Type=Application

Categories=Application;
```
## LibreOffice
LibreOffice是linux平台下的办公软件，开源并且完全免费，有一个可靠的团队维护并持续不断更新，下载地址：[LibreOffice稳定版](https://zh-cn.libreoffice.org/download/libreoffice-still/) [安装指南](https://zh-cn.libreoffice.org/get-help/install-howto/linux/)

![office](https://c1h3ng.github.io/assets/images/office.png)
## 文本编辑器
我选择了atom，界面酷炫可定制，很多优秀的插件，当然替代品也很多，sublime vscode都有各自的优点，[atom官网](https://atom.io/)，下载deb包，然后安装
```
root@kali:~/Downloads# dpkg -i atom-amd64.deb
//若出现依赖问题，执行如下命令
root@kali:~/Downloads# apt-get -f install
root@kali:~/Downloads# dpkg -i atom-amd64.deb
```
![atom](https://c1h3ng.github.io/assets/images/atom.png)
## ShadowSocks
```
root@kali:~# apt-get install qt5-qmake qtbase5-dev libbotan1.10-dev pkg-config debhelper
root@kali:~# cd Downloads/
root@kali:~/Downloads# git clone https://github.com/shadowsocks/libQtShadowsocks.git
root@kali:~/Downloads# cd libQtShadowsocks/
root@kali:~/Downloads/libQtShadowsocks# dpkg-buildpackage -uc -us -b
//若出现报错提示cmake版本太低，apt-get install cmake下载最新版本即可
root@kali:~/Downloads/libQtShadowsocks# dpkg -i ../libqtshadowsocks_1.11.0-1_amd64.deb ../libqtshadowsocks-dev_1.11.0-1_amd64.deb
root@kali:~/Downloads/libQtShadowsocks# cd ..
root@kali:~/Downloads# apt-get install libqrencode-dev libzbar-dev libappindicator-dev
root@kali:~/Downloads# git clone https://github.com/shadowsocks/shadowsocks-qt5.git
root@kali:~/Downloads# cd shadowsocks-qt5/
root@kali:~/Downloads/shadowsocks-qt5# dpkg-buildpackage -uc -us -b
root@kali:~/Downloads/shadowsocks-qt5# dpkg -i ../shadowsocks-qt5_2.9.0-1_amd64.deb
```
安装完成后，dock中就有了熟悉的纸飞机了
## Google Chrome
由于学校某些网站对火狐的兼容性太差了(敲桌)，前端加载出来各种bug，所以这里我还需要一个强大的浏览器
```
root@kali:~/Downloads# wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
//下载最新版本的google chrome 64位的debian安装包
root@kali:~/Downloads# dpkg -i google-chrome-stable_current_amd64.deb
root@kali:~/Downloads# apt-get -f install
root@kali:~/Downloads# dpkg -i google-chrome-stable_current_amd64.deb
```
这里chrome64位的程序出现了和网易云音乐一样的问题，可以使用一样的方法解决：
```
root@kali:~# cd /usr/share/applications
root@kali:/usr/share/applications# vim google-chrome.desktop
//exec后添加参数 --no-sandbox
```
![chrome](https://raw.githubusercontent.com/c1h3ng/c1h3ng.github.io/master/assets/images/chrome.png)

到此对kali的定制就基本结束了，碰到了不少的坑，记录下来把坑填了(叹气)
