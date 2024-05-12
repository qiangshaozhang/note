## CSRF跨站请求伪造

#### 条件

1、需要请求伪造数据包

2、无过滤防护，有过滤防护能绕过

3、受害者需要触发



#### 案例一（无防护）

burp抓到添加用户的包

![image-20240508142831522](D:\Infiltration\笔记\note\图片\image-20240508142831522.png)



使用burp自带的转换为csrf的poc

![image-20240508142905364](D:\Infiltration\笔记\note\图片\image-20240508142905364.png)



勾选上include-auto-submit script，删除点击标签，使其不用点击就能访问



![image-20240508143110314](D:\Infiltration\笔记\note\图片\image-20240508143110314.png)



这个数据包构造好了之后，点击这个html，如果该用户刚好登录了这个cms的后台并且有权限增加用户那么就会成功创建一个admin用户。

![image-20240508143805850](D:\Infiltration\笔记\note\图片\image-20240508143805850.png)

防护措施：检测(Referer)来源



#### 案例二

抓取zblog新增数据包



![image-20240508162022987](D:\Infiltration\笔记\note\图片\image-20240508162022987.png)



转化为csrf poc

![image-20240508145742674](D:\Infiltration\笔记\note\图片\image-20240508145742674.png)

将html上传到外网服务器访问，被过滤

![image-20240508163309343](D:\Infiltration\笔记\note\图片\image-20240508163309343.png)



根据网页目录找到cmd.php文件，搜索MemberMng

![image-20240508164557567](D:\Infiltration\笔记\note\图片\image-20240508164557567.png)

按住ctrl点击CheckIsRefererValid

![image-20240508164902521](D:\Infiltration\笔记\note\图片\image-20240508164902521.png)

按住ctrl点击CheckHTTPRefererValid,HTTP_REFERER函数是获取referer头的，如果referer为空的话就执行成功

![image-20240508164958814](D:\Infiltration\笔记\note\图片\image-20240508164958814.png)

将referer头改成网站地址绕过



![image-20240508161417110](D:\Infiltration\笔记\note\图片\image-20240508161417110.png)

或者在referrer头后门加上同源地址

![image-20240508170424502](D:\Infiltration\笔记\note\图片\image-20240508170424502.png)



成功添加，但是这个并没有什么用，因为被攻击者不会主动去修改referfer头

![image-20240508163434906](D:\Infiltration\笔记\note\图片\image-20240508163434906.png)

在生成的poc头部加上这个<meta name="referrer" content="no-referrer">使其数据包自动将referer头去空

![image-20240508184728613](D:\Infiltration\笔记\note\图片\image-20240508184728613.png)

通过burp抓包可以看到已经没有了referer头了

![image-20240508190019265](D:\Infiltration\笔记\note\图片\image-20240508190019265.png)



放包后成功添加，因为代码逻辑就是为空就是True

![image-20240508190058861](D:\Infiltration\笔记\note\图片\image-20240508190058861.png)



#### 绕过

###### referer验证

```
规则匹配绕过问题（代码逻辑不严谨）
添加<meta name="referrer" content="no-referrer">到头部文件
referer头加上同源地址：http://xx.xx.xx.xx/http://xx.xx.xx.xx
配合文件上传绕过（严谨使用同源绕过）
配合存储XSS绕过（严谨使用同源绕过）
```

###### token验证

```
Token参数值复用（代码逻辑不严谨）
Token参数删除（代码逻辑不严谨）
Token参数值置空（代码逻辑不严谨）
```





```
#CSRF-无检测防护-检测&生成&利用
检测：黑盒手工利用测试，白盒看代码检验（有无token，来源检验等）
生成：BurpSuite->Engagement tools->Generate CSRF Poc
利用：将文件防止自己的站点下，诱使受害者访问（或配合XSS触发访问）

#CSRF-Referer同源-规则&上传&XSS
https://blog.csdn.net/weixin_50464560/article/details/120581841
严谨代码PHP DEMO：
<?php
// 检测来源
function checkReferrer() {
    $expectedReferrer = "http://example.com"; // 期望的来源页面

    if (!isset($_SERVER['HTTP_REFERER']) || $_SERVER['HTTP_REFERER'] !== $expectedReferrer) {
        die("非法访问");
    }
}

// 处理表单提交
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    // 检测来源
    checkReferrer();

    // 获取用户输入的数据
    $name = $_POST['name'];
    $email = $_POST['email'];

    // 输出用户输入的数据
    echo "姓名：$name<br>";
    echo "邮箱：$email<br>";
    exit;
}
?>

<!DOCTYPE html>
<html>
<head>
    <title>检测来源示例</title>
</head>
<body>
    <h1>检测来源示例</h1>
    <form action="<?php echo $_SERVER['PHP_SELF']; ?>" method="POST">
        <label for="name">姓名：</label>
        <input type="text" name="name" id="name" required />
        <br>
        <label for="email">邮箱：</label>
        <input type="email" name="email" id="email" required />
        <br>
        <input type="submit" value="提交" />
    </form>
</body>
</html>

绕过0：规则匹配绕过问题（代码逻辑不严谨）
1、<meta name="referrer" content="no-referrer">
2、http://xx.xx.xx.xx/http://xx.xx.xx.xx
绕过1：配合文件上传绕过（严谨使用同源绕过）
绕过2：配合存储XSS绕过（严谨使用同源绕过）


#CSRF-Token校验-值删除&复用&留空
https://blog.csdn.net/weixin_50464560/article/details/120581841
严谨代码PHP DEMO：
<?php
session_start();

// 生成并存储 CSRF Token
function generateCSRFToken() {
    $token = bin2hex(random_bytes(32));
    $_SESSION['csrf_token'] = $token;
    return $token;
}

// 检查 CSRF Token 是否有效
function validateCSRFToken($token) {
    return isset($_SESSION['csrf_token']) && $_SESSION['csrf_token'] === $token;
}

// 处理表单提交
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    // 检查 CSRF Token
    if (!isset($_POST['csrf_token']) || !validateCSRFToken($_POST['csrf_token'])) {
        die("CSRF Token 验证失败");
    }

    // 获取用户输入的数据
    $name = $_POST['name'];
    $email = $_POST['email'];

    // 输出用户输入的数据
    echo "姓名：$name<br>";
    echo "邮箱：$email<br>";
    exit;
}

// 生成 CSRF Token
$csrfToken = generateCSRFToken();
?>

<!DOCTYPE html>
<html>
<head>
    <title>CSRF Token 示例</title>
</head>
<body>
    <h1>CSRF Token 示例</h1>
    <form action="<?php echo $_SERVER['PHP_SELF']; ?>" method="POST">
        <input type="hidden" name="csrf_token" value="<?php echo $csrfToken; ?>" />
        <label for="name">姓名：</label>
        <input type="text" name="name" id="name" required />
        <br>
        <label for="email">邮箱：</label>
        <input type="email" name="email" id="email" required />
        <br>
        <input type="submit" value="提交" />
    </form>
</body>
</html>

绕过0：将Token参数值复用（代码逻辑不严谨）
绕过1：将Token参数删除（代码逻辑不严谨）
绕过2：将Token参数值置空（代码逻辑不严谨）

```

