---
layout: post
title: "apache最新解析漏洞(CVE-2017-15715)绕过文件上传限制"
date: 2018-4-10
tags:
  web
categories:
  Web
---

在p牛博客最近更新的文章，[传送门](https://www.leavesongs.com/PENETRATION/apache-cve-2017-15715-vulnerability.html)，感觉很有意思，自己在自己本地测试了一下

# 0x01 正则表达式中的 '$'

apache这次解析漏洞的根本原因就是这个 ``$``，正则表达式中，我们都知道$用来匹配字符串结尾位置，我们来看看[菜鸟教程](http://www.runoob.com/regexp/regexp-syntax.html)中对正则表达符``$``的解释：

> 匹配输入字符串的结尾位置。如果设置了 RegExp 对象的 Multiline 属性，则 $ 也匹配 '\\n' 或 '\\r'。要匹配 $ 字符本身，请使用 \\$。

那么就明白了，在设置了 RegExp 对象的 Multiline 属性的条件下，``$``还会匹配到字符串结尾的换行符

# 0x02 Linux环境

这里本地是debian系的kali linux，apache配置文件路径在``/etc/apache2/``下，``apache2.conf``是apache核心配置文件，由于我本地php作为apache的mod方式运行的，所以需要在``mods-enabled``目录下找到关于apache-php模块的配置：

![](https://github.com/c1h3ng/c1h3ng.github.io/blob/master/assets/images/apache-php7.png?raw=true)

可以看见php7.0.conf是``mods-available/php7.0.conf``的软链接，配置如下：

```html
<FilesMatch ".+\.ph(p[3457]?|t|tml)$">
    SetHandler application/x-httpd-php
</FilesMatch>
<FilesMatch ".+\.phps$">
    SetHandler application/x-httpd-php-source
    # Deny access to raw php sources by default
    # To re-enable it's recommended to enable access to the files
    # only in specific virtual host or directory
    Require all denied
</FilesMatch>
# Deny access to files without filename (e.g. '.php')
<FilesMatch "^\.ph(p[3457]?|t|tml|ps)$">
    Require all denied
</FilesMatch>

# Running PHP scripts in user directories is disabled by default
#
# To re-enable PHP in user directories comment the following lines
# (from <IfModule ...> to </IfModule>.) Do NOT set it to On as it
# prevents .htaccess files from disabling it.
<IfModule mod_userdir.c>
    <Directory /home/*/public_html>
        php_admin_flag engine Off
    </Directory>
</IfModule>
```

第一行就告诉了我们apache会将哪些后缀的文件当做php解析：

```html
<FilesMatch ".+\.ph(p[3457]?|t|tml)$">
```

以如下方式结尾的文件会被apache当做php解析：

```
php
php3
php4
php5
php7
pht
phtml
```

如果我们再结合我们上面提到的关于``$``的使用，很容易想到，如果后缀名是上面这些后缀名以换行符结尾，那么也是可以解析的，本地构造文件：

![](https://github.com/c1h3ng/c1h3ng.github.io/blob/master/assets/images/cve-2017-15715.png?raw=true)

文件构造好了，从浏览器打开试试看看能不能解析：

![](https://github.com/c1h3ng/c1h3ng.github.io/blob/master/assets/images/php0x0a.png?raw=true)

可以看见是能解析的，那么在文件上传黑名单就可以通过这种思路来绕过了。

# 0x02 Windows环境

关于windows环境，p牛博客下面有一些人说测试失败，我也进行了测试，虚拟机环境 __win7+phpstudy__ : *Apache/2.4.23 (Win32) OpenSSL/1.0.2j PHP/5.4.45*

用p牛的代码测试：

```php
<html>
<body>

    <form action="test.php" method="post" enctype="multipart/form-data">

    <input type="file" name="file" />

    <input type="text" name="name" />

    <input type="submit" value="上传文件" />

    </form>

</body>
</html>
<?php
if(isset($_FILES['file'])) {
    $name = basename($_POST['name']);
    $ext = pathinfo($name,PATHINFO_EXTENSION);
    if(in_array($ext, ['php', 'php3', 'php4', 'php5', 'phtml', 'pht'])) {
        exit('bad file');
    }
    move_uploaded_file($_FILES['file']['tmp_name'], './' . $name);
}
?>

```

抓包修改文件名，上传：

![](https://github.com/c1h3ng/c1h3ng.github.io/blob/master/assets/images/burp-upload.png?raw=true)

![](https://github.com/c1h3ng/c1h3ng.github.io/blob/master/assets/images/windows-upxxx.png?raw=true)

可以看见，这里出现了两个warning，其实并非测试不成功，可以看见其实是绕过了我们代码里的黑名单的，已经执行到了``move_uploaded_file``了，说明程序并没有因为没有绕过黑名单而exit，但是因为涉及到文件读写，而windows操作系统不允许后缀以换行符结尾的文件命名方式，所以这里会文件会创建失败，就出现了这两个warning了
