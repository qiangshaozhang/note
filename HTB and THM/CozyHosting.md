## CozyHosting -未完

使用nmap对目标机进行扫描

```
nmap -sV -sC 10.10.11.230
```

![image-20231210205403478](/home/gary/文档/GitHub/note/图片/image-20231210205403478.png) 

发现了22和80端口

登录上去先用gobuster目录爆破

```
gobuster dir -u http://cozyhosting.htb -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt -t 200
```

![image-20231211093932149](../图片/image-20231211093932149.png)



自己去网页上看看，发现报错页面出现Whitelabel Error Page 这是Springboot的报错页面

![image-20231211091527304](../图片/image-20231211091527304.png)

使用springboot未授权访问专门字典爆破一下

```
gobuster dir -u http://cozyhosting.htb -w /usr/share/seclists/Discovery/Web-Content/spring-boot.txt -t 200
```

![image-20231211095846838](../图片/image-20231211095846838.png)

```
/actuator/mappings    (Status: 200) [Size: 9938]
/actuator             (Status: 200) [Size: 634]
/actuator/health      (Status: 200) [Size: 15]
/actuator/env/path    (Status: 200) [Size: 487]
/actuator/env/home    (Status: 200) [Size: 487]
/actuator/env/lang    (Status: 200) [Size: 487]
/actuator/sessions    (Status: 200) [Size: 48]
/actuator/env         (Status: 200) [Size: 4957]
/actuator/beans       (Status: 200) [Size: 127224]


其中对寻找漏洞比较重要接口的有：

/env、/actuator/env

GET 请求 /env 会直接泄露环境变量、内网地址、配置中的用户名等信息；当程序员的属性名命名不规范，例如 password 写成 psasword、pwd 时，会泄露密码明文；

同时有一定概率可以通过 POST 请求 /env 接口设置一些属性，间接触发相关 RCE 漏洞；同时有概率获得星号遮掩的密码、密钥等重要隐私信息的明文。

/refresh、/actuator/refresh

POST 请求 /env 接口设置属性后，可同时配合 POST 请求 /refresh 接口刷新属性变量来触发相关 RCE 漏洞。

/restart、/actuator/restart

暴露出此接口的情况较少；可以配合 POST请求 /env 接口设置属性后，再 POST 请求 /restart 接口重启应用来触发相关 RCE 漏洞。

/jolokia、/actuator/jolokia

可以通过 /jolokia/list 接口寻找可以利用的 MBean，间接触发相关 RCE 漏洞、获得星号遮掩的重要隐私信息的明文等。

/trace、/actuator/httptrace

一些 http 请求包访问跟踪信息，有可能在其中发现内网应用系统的一些请求信息详情；以及有效用户或管理员的 cookie、jwt token 等信息。
```

