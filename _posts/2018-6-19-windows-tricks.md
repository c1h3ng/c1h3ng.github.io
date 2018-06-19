---
layout: post
title: "windows的一些特性"
date: 2018-6-19
tags:
  web
  Windows
categories:
  Web
---
# 0x01 Windows FindFirstFile API
``FindFirstFile``是windows系统的API函数，用于查找指定目录下的第一个文件或目录

测试代码如下：

```php
------test.php---------

<?php
if(stripos($_GET['file'],'catchme_plz') !== False)
{
	die('Come on!');
}

require_once($_GET['file']);

------catchme_plz.txt---------

<?php echo 'Congratulations!';phpinfo();?>
```

wamp环境下，在文件名不可知部分用``<``、``>``代替即可，只使用单个``<``、``>``只能代表一个字符，需要表达多个字符使用``<<``

![](https://c1h3ng.github.io/assets/images/findfirstfile.png)

php5、php7下都通过了测试，另外并非只有``require``、``include``、``require_once``、``include_once``一类的函数能使用：

| | 函数名 | |
| :---: | :---: | :---: |
| include | include_once | require |
| fopen | copy | file_get_contents |
| readfile | file_put_contents | mkdir |
| touch | move_uploaded_file | opendir |
| rewinddir | closedir |  readdir |
|   tempnam  | parse_ini_file  | require_once  |

# 0x02 Windows短文件名

还是上面的代码，利用Windows短文件名的特性，也可以实现绕过限制包含文件，创建短文件名遵循下列规则：

* 长文件名中的空格及非法字符(\. \" \/ \\ [ ] \: \; = ,)不在短文件名中出现
* 取长文件名前六个字符，唯一文件以``~1``结尾，重复文件以``~2``以此类推
* 截取文件后缀3个字符以内，超过三个字符的后缀，截取前三个字符
* 转换文件名中所有字符为大写
* 短文件名仅包含一个``.``，windows会忽略后面没有字符的最后一个句号
* 对于普通文件名，超过八个字符才会创建短文件名，若小于八个字符，但是文件名含有空格，也会创建短文件名

![](https://c1h3ng.github.io/assets/images/8dot3.png)

使用``dir /x``查看短文件名:

![](https://c1h3ng.github.io/assets/images/dirx.png)

通过短文件名的方式包含文件：

![](https://c1h3ng.github.io/assets/images/lfi-8dot3.png)

# 0x03 NTFS ADS

``ADS``即 ``NTFS Alternmate Data Streams``，NTFS交换数据流，为了使NTFS文件系统兼容苹果公司退出的HFS(分层文件系统)，主要有如下几点特征：

* 一个完整的流的格式为：\<file name\>\:\<stream name\>\:\<stream type\>
* NTFS下有如下流类型：

|Stream Type | Description |
| :---: | :--- |
| $ATTRIBUTE_LIST | Lists  the  location  of  all  attribute  records  that  do  not  fit  in  the MFT record |
| $BITMAP | Attribute for Bitmaps |
| $DATA | Contains default file data |
| $EA | Extended attribute index |
| $EA_INFORMATION | Extended attribute information |
| $FILE_NAME | File name |
| $INDEX_ALLOCATION | The type name for a Directory Stream. A string for the attribute code for index allocation |
| $INDEX_ROOT | Used to support folders and other indexes |
| $LOGGED_UTILITY_STREAM | Use by the encrypting file system |
| $OBJECT_ID | Unique GUID for every MFT record |
| $PROPERTY_SET | Obsolete |
| $REPARSE_POINT | Used for volume mount points |
| $SECURITY_DESCRIPTOR | Security descriptor stores ACL and SIDs|
| $STANDARD_INFORMATION | Standard information such as file times and quota data |
| $SYMBOLIC_LINK | Obsolete |
| $TXF_DATA | Transactional NTFS data |
| $VOLUME_INFORMATION | Version and state of the volume |
| $VOLUME_NAME | Name of the volume |
| $VOLUME_VERSION | Obsolete. Volume version |


* 修改宿主文件内容不会影响流
* 修改流内容不会影响宿主文件
* 流类型总是以\$符号作为开始，NTFS文件系统中的文件至少包含一个主流，也就是data流(\$DATA)，默认流名为空
* NTFS文件系统下文件夹没有data流，但可以指派data流，文件夹的主流为directory流(\$INDEX_ALLOCATION)，流名默认为``$I30``
* 一个文件被指派了流名，而没有指定流类型，默认为\$DATA

## 隐藏webshell
![](https://c1h3ng.github.io/assets/images/hiddentext.png)

上面以数据流的形式将 流名为``hidden.txt``，流类型为``$DATA``包含至宿主文件``test.png``中，操作前后可以发现文件大小没有任何变化，具有一定的隐蔽性，可以用来隐藏webshell，木马之类的，webshell的话直接在其他文件中包含即可执行：

![](https://c1h3ng.github.io/assets/images/adswebshell.png)

## 绕过上传黑名单
也可以通过构造类似``<file name>:<stream name>:<stream type>``这样的文件名来绕过上传黑名单，以dvwa低等级的上传关演示：

![](https://c1h3ng.github.io/assets/images/adsbypassdvwa.png)

虽然dvwa低等级上传关没有任何防护措施，假设这里有文件上传后缀名的黑名单，就可以使用这种方式绕过黑名单上传php文件

## MYSQL突破UDF提权困境

都知道mysql5.1以上的mysql在udf提权时，要将udf到处到指定的MYSQL目录下的``lib/plugin``下才有效，有时候会碰到默认安装后没有这个目录的，这里就可以使用到``$INDEX_ALLOCATION``这个流类型，以dvwa演示，通过抓包修改上传文件名为``test::$INDEX_ALLOCATION``：

![](https://c1h3ng.github.io/assets/images/index_allocation.png)

成功在上传目录下创建了test文件夹，udf提权时理论上可以通过``select 'xxx' into outfile 'C:\\mysql\\lib\\plugin::$INDEX_ALLOCATION';``这样的方式来创建``lib\plugin``目录

...但是我是没有测试成功的，先留个坑吧，折腾的太晚了，再继续可能会猝死，希望以后有机会能搞明白了

还有[IIS3.0/4.0的源代码泄露](http://www.nsfocus.net/index.php?act=sec_bug&do=view&bug_id=3442)和[IIS6.0/7.5的目录访问权限绕过](https://www.exploit-db.com/exploits/19033/)等等都和NTFS ADS有关
