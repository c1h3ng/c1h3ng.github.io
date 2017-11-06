---
layout: post
title: "phpmyadmin突破secure_file_priv写shell"
date: 2017-11-6
tags:
  phpmyadmin
categories:
  Web
---
写在以前博客上的文章，放过来备忘
# 0x01 secure_file_priv
 secure_file_priv变量是指定loadfile,into outfile的作用目录，首先修改my.ini中secure_file_priv的值为NULL，改为null后mysql将不能使用loadfile，into outfile进行读写文件：

 ![](https://github.com/c1h3ng/c1h3ng.github.io/blob/master/assets/images/myini.png?raw=true)

现在再来试试能不能进行文件读写：

![](https://github.com/c1h3ng/c1h3ng.github.io/blob/master/assets/images/outfile.png?raw=true)

已经无法进行文件读写了，还是有姿势可以getshell
# 0x02 两个全局变量
mysql中有两个全局变量是我们getshell需要用到的重要变量

* **general_log**
* **general_log_file**

**general_log** 是mysql中记录sql操作的日志，所有的查询语句会记录在一个日志文件中，但因为时间长了会导致日志文件非常大，所以默认为关闭，有时候在管理员需要进行排错时才会暂时性的打开这个变量

**general_log_file** 就是操作日志存放的路径，默认值如下：

![](https://github.com/c1h3ng/c1h3ng.github.io/blob/master/assets/images/log.png?raw=true)

# 0x03 getshell

知道这两个全局变量的作用了，就可以利用它们来getshell，首先mysql中set全局变量需要当前用户具有 **SUPER** 权限，否则没有权限进行set，拥有 **SUPER** 权限后，先打开日志记录功能，再将日志路径修改为shell地址：

![](https://github.com/c1h3ng/c1h3ng.github.io/blob/master/assets/images/general_log.png?raw=true)

![](https://github.com/c1h3ng/c1h3ng.github.io/blob/master/assets/images/general_log_file.png?raw=true)

现在只需执行sql语句，mysql会将执行的语句内容记录到我们指定的文件中，就可以getshell了：

![](https://github.com/c1h3ng/c1h3ng.github.io/blob/master/assets/images/sqlexec.png?raw=true)

![](https://github.com/c1h3ng/c1h3ng.github.io/blob/master/assets/images/phpinfo.png?raw=true)
