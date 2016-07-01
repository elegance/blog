---
title: PostgreSQL之psql的使用
date: 2016-07-01 10:56:37
categories:
- PostgreSQL
- 数据库

tags:
- PostgreSQL基础
---

### psql 的使用
直接键入`psql`进入到命令行：
```
[postgres@localhost ~]$ psql 
psql (9.5.3)
Type "help" for help.
```
**安装PostgreSQL数据库时，会建立一个与初始化数据库时的操作系统用户同名的数据库用户，同时这个用户是数据库超级用户，在这个OS用户下，登录是执行的操作系统认证，可以在pg_hba.conf下配置认证**
psql命令格式：
    `psql -h <hostname or ip> -p <端口号> [数据库名称] [用户名称]`
例：
    `pgsql -h 192.168.2.11 -p 5432 testdb postgres`

查看有哪些数据库
```
[postgres@localhost ~]$ psql -l
#或者在进入了psql后，使用“\”开头调用psql命令
postgres-# \l
```

连接至某个数据库、查看有哪些表、查看表的结构：
```
postgres-# \c db_name
db_name-# \d
db_name-# \d table_name
```
