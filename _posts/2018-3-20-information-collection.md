---
layout: post
title: "渗透前信息收集的个人总结"
date: 2018-3-20
tags:
  web 渗透测试
categories:
  Web
---
# 0x01 搜集子域名

一般来说这就是我的第一步，工具很多，seay法师的``子域名挖掘机``，lijiejie的``subdomainsbrute``，结合起来效果更好，法师的子域名挖掘机内含一个百万大字典，可以直接将字典放到subdomainsbrute里，毕竟python跨平台性更好，子域名挖掘机要在linux平台上跑起来还是要折腾一下的，有折腾的时间，子域名早就跑完了

# 0x02 C段

域名仅仅是便于记忆的一串字符，实际上的寻址还是靠ip地址，DNS服务器会将域名解析到对应的ip地址上，完成一次访问。上一步收集的子域名很重要，但ip段才是重中之重，子域名服务器所在的C段，才是我们要收集的，如果目标很大，那么极有可能同C段的服务器都处在一个内网，目标越大命中率越高，**并且还能接触到目标的测试服务器、边缘应用，这些系统一般安全性不高**，将子域名挖掘机或subdomainsbrute导出的文本拿来去掉域名，收集ip的ABC段，去掉D段，再生成一个完整的C段，脚本如下：

```python
import sys
import re

def main(path1,path2):
    f1 = open(path1,'r')
    f2 = open(path2,'w')
    c_list = []
    iplist = []
    for line in f1:
        #print line
        c_block = re.findall("\d{1,3}.\d{1,3}.\d{1,3}",line)
        if re.match("^(172|10|192)+.\d+.\d+",c_block[0]):
            pass
        else:
            c_list.append(c_block[0])
    for c in c_list:
        #print c
        ip = re.findall("\d{1,3}.\d{1,3}.\d{1,3}",c)
        if ip[0] not in iplist:
            iplist.append(ip[0])
    for cblock in iplist:
        for i in range(1,255):
            f2.write(cblock+"."+str(i)+"\n")

if __name__ == '__main__':
    try:
        path1 = sys.argv[1]
        path2 = sys.argv[2]
        main(path1,path2)
    except:
        print 'eg: python xxx.py ./domains.txt ./c-block.txt'

```

![](https://github.com/c1h3ng/c1h3ng.github.io/blob/master/assets/images/cblock.png?raw=true)

经过处理之后的文本可以很方便的导入一些扫描器

# 0x03 端口扫描

ip段有了，接下来就是筛选出存活主机并扫描端口，这两样nmap都能实现，当然也可以用其他工具像``fenghuangscan``，``amap``之类的

将前面处理后的文本交给nmap进行扫描，再对nmap的扫描结果简单处理一下，可以使用python的libnmap库处理nmap导出的xml文件，还可以配合awk简单处理nmap结果：

```bash
nmap  -vv -sS -sV -iL target.txt -p 21,22,23,25,53,67,68,80,110,139,143,161,389,443,445,512,513,514,873,1080,1352,1433,1521,2049,2181,3306,3389,4848,5000,5432,5632,5900,6379,7001,8080,8069,9090,9000,9200,9300,11211,27017 > nmap.txt && awk /Discovered/{'print $6":"$4'} ./nmap.txt | awk -F"/" {'print $1'} > output.txt
```

![](https://github.com/c1h3ng/c1h3ng.github.io/blob/master/assets/images/ip:port.png?raw=true)

处理后的文本更加直观，也可以使用nmap在这一阶段同时进行指纹识别

# 0x04 指纹识别

知道开放了哪些端口了，接下来就是服务探测指纹识别了，工具很多，``nmap``、``amap``、``netcat``、``dmitry``等等，web应用的指纹识别可以使用``whatweb``，还可以使用在线的平台，像 **[钟馗之眼](https://www.zoomeye.org)、[censys](https://censys.io)、[shodan](https://www.shodan.io)**

完整的服务探测指纹识别能使渗透工作轻松不少，从中可以了解到很多目标信息，目标主机运行了什么软件，软件的版本，web应用使用的开发框架，web容器种类和版本，这些信息都是很重要的，知道这些就能展开一个有针对性的黑盒测试，针对薄弱处进行渗透

# 0x05 后记

由于自己挖洞的经验还不够丰富，渗透前做的信息收集就总结这么多了，至于google hack、github敏感信息收集之类的，渗透时也很有必要，但这些就略去不写了，近期准备专心刷一刷src了，另外附上常见端口入侵总结：


| 端口 | 服务 | 入侵方式 |
| :---: | :---: |  :---: |
| 21 | ftp/tftp | 爆破/嗅探/溢出/后门 |
| 22 | ssh | 爆破/openssh漏洞 |
| 23 | telnet | 爆破/嗅探 |
| 25 | smtp | 邮件伪造 |
| 53 | dns | 域传送/劫持/缓存投毒/欺骗 |
| 67/68 | dhcp | 劫持/欺骗 |
| 110 | pop3 | 爆破 |
| 139 | samba | 爆破/未授权访问/远程命令执行 |
| 143 | imap | 爆破 |
| 161 | snmp  | 爆破 |
| 389 | ldap | 注入/未授权访问 |
| 445 | smb | ms17-010 |
| 512/513/514 | linux r | 直接rlogin  |
| 873 | rsync | 未授权访问 |
| 1080 | socket | 爆破 |
| 1352 | lotus | 爆破/信息泄漏 |
| 1433 | mssql | 爆破/注入 |
| 1521 | oracle | 爆破/注入 |
| 2049 | nfs | 配置不当 |
| 2181 | zookeeper | 未授权访问 |
| 2375 | docker remote api | 未授权访问 |
| 3306 | mysql | 爆破/注入 |
| 3389 | remote desktop | 爆破/shift后门 |
| 4848 | glassfish | 爆破/认证绕过 |
| 5000 | sybase/DB2 | 爆破/注入 |
| 5432 | postgresql | 爆破/注入/缓冲区溢出 |
| 5632 | pcanywhere | 代码执行 |
| 5900 | vnc | 爆破/认证绕过 |
| 6379 | redis | 未授权访问/爆破 |
| 7001 | weblogic | java反序列化/控制台弱口令 |
| 80/443 | http/https  | web应用漏洞/心脏滴血 |
| 8069 | zabbix | 远程命令执行/注入 |
| 8161 | activemq | 弱口令/写文件 |
| 8080 | tomcat | 爆破 |
| 8083/8086 | influxDB | 未授权访问 |
| 9000 | fastcgi | 远程命令执行 |
| 9090 | websphere | 爆破/java反序列化 |
| 9200/9300 | elasticsearch | 远程代码执行 |
| 11211 | memcached | 未授权访问 |
| 27017 | mongodb | 未授权访问/爆破 |
