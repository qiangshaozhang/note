#### kali使用proxychains代理流量转发至burp

前言

有时候使用脚本，不知道流量数据包是什么样的，需要抓包看看数据包。秃子教过在windows下使用Proxifier代理工具进行流量转发至burp，举一反三。

kali自带了proxychains，编辑文件,注释掉socks4,添加本机真实ip，不能写127.0.0.1记住端口后面设置burp端口需要一致。

```
vim /etc/proxychains4.conf
```

![image-20231221214208667](/home/gary/文档/GitHub/note/图片/image-20231221214208667.png)

在burp代理添加上代理ip

![image-20231221215049474](/home/gary/文档/GitHub/note/图片/image-20231221215049474.png)

就正常执行exp的前提上加上proxychains4

```
proxychains4 python CVE-2023-41892.py http://surveillance.htb/
```

在burp上成功捕获流量

![image-20231221215013812](/home/gary/文档/GitHub/note/图片/image-20231221215013812.png)

