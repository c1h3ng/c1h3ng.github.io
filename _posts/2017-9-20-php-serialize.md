---
layout: post
title: "php反序列化安全问题"
date: 2017-9-20
tags:
  php
  web
categories:
  php
---
php反序列化漏洞是去年比较火的一个bug，去年SWPUCTF校赛学长们还出了一道反序列化的题，当时就了解了下，后来打比赛，其他学校的师傅也陆陆续续出到过反序列化的题，原来的博客有写过，博客托管到github后再记录一下以免忘记。
# 关于serialize和unserialize
serialize和unserialize分别是php中的序列化和反序列化函数，官方说明如下：

![serialize](https://c1h3ng.github.io/assets/images/serialize.png)

![unserialize](https://raw.githubusercontent.com/c1h3ng/c1h3ng.github.io/master/assets/images/unserialize.png)

在数据传输和存储过程中为了保证数据结构和变量类型不丢失，会使用到序列化和反序列化。假设需要将一个数组或者一个对象存储到文件或者数据库中，直接存储的话只是一串普通的字符串，原来的数据的结构和类型完全丢失了，但将其序列化后就可以将数据转化成一串标识了数据类型长度的可存储的字符串，在取用数据时将其反序列化就可以恢复原本的数据类型及结构，这就是serialize和unserialize两个函数的作用。
```
•	a - array
•	b - boolean
•	d - double
•	i - integer
•	o - common object
•	r - reference
•	s - non-escaped binary string
•	S - escaped binary string
•	C - custom object
•	O - class
•	N - null
•	R - pointer reference
•	U - unicode string
```
以上为序列化中常见格式，来看一个实例：
```
<?php
$a = array(1,"apple",true,3.1415926,NULL);
echo serialize($a);
echo "\n";
var_dump(unserialize(serialize($a)));
?>
```
![xulie](https://c1h3ng.github.io/assets/images/xulie.png)

在对对象进行序列化时，属性的修饰符(public,var,protected,private)的不同，生成的序列化字符串也会有所不同，来看实例：
```
<?php
class test{
	var $a = "var";
	public $b = "public";
	protected $c = "protected";
	private $d = "private";
}
$test = new test();
echo serialize($test);
echo "\n";
echo urlencode(serialize($test));
?>
```
![fanxulie](https://raw.githubusercontent.com/c1h3ng/c1h3ng.github.io/master/assets/images/fanxulie.png)

可以看见var和public修饰的属性都是公有的，所以序列化之后的格式是一样的，protected修饰的序列化后变成了s:4:" * c"，编码之后可以发现这里多出了"\0*\0"，private修饰的属性序列化后的形式变为了s:7:" test d"，比公有属性多出了"\0<类名>\0"，在进行长度计算时也会计算在内。
# 关于魔术方法\__sleep()和__wakeup()
在对对象进行序列化和反序列化时，如果存在魔术方法\__sleep()和\__wakeup()，那么在序列化之前会触发\__sleep()方法，在反序列化前会触发\__wakeup()方法，但是反序列化时如果对序列化的字串反序列化失败就不会触发\__wakeup()方法
# 绕过__wakeup进行危险操作
上一个直观的例子，绕过\__wakeup()方法进行一些危险的操作：
```
<?php
class test{
	var $file = null;
	function __destruct(){
		echo "__destruct...<br>";
		echo file_get_contents($this->file);
	}
	function __wakeup(){
		echo "__wakeup...<br>";
		$this->file = "1.txt";
	}
}
$payload = $_GET['payload'];
unserialize($payload);
?>
```
![example](https://c1h3ng.github.io/assets/images/example.png)

可以看到，\__destruct()方法里是一个读文件的操作，__wake()方法里给属性file赋值为1.txt，GET传入能够反序列化的序列化字串最终是肯定读取1.txt的内容的，如下：

![eg](https://github.com/c1h3ng/c1h3ng.github.io/blob/master/assets/images/eg.png?raw=true)

但是如果反序列化失败就不会触发\__wakeup()方法，这里就会产生了任意文件读取漏洞，我们传入 O:4:"test":2:{s:4:"file";s:11:"/etc/passwd";}，这里将类中属性的数量改为2，但这串序列化字串中只有一个属性，所以反序列化会失败，成功绕过\__wakeup()方法从而这里产生了一个任意文件读取的漏洞：

![vul](https://github.com/c1h3ng/c1h3ng.github.io/blob/master/assets/images/vul.png?raw=true)

如果实际情况中使用了eval(),assert(),exec()之类的危险函数，就会造成很大的安全隐患。
