---
title: CentOS6.5.PostgreSQL源码编译安装
date: 2016-07-01 09:36:39
categories:
- Linux
- 数据库
- PostgreSQL

tags:
- Centos软件安装
- PostgreSQL基础

---
### 准备工作
postgresql官网：http://www.postgresql.org/
下载软件：https://ftp.postgresql.org/pub/source/v9.5.3/postgresql-9.5.3.tar.bz2

## 安装、环境变量配置
``` bash
[root@localhost ~]# ./configure --prefix=/usr/local/postgresql-9.5.3 --with-python --with-perl
[root@localhost ~]# make
[root@localhost ~]# make install
[root@localhost ~]# ln -sf /usr/local/postgresql-9.5.3 /usr/local/pgsql
[root@localhost ~]# export PATH=/usr/local/pgsql/bin:$PATH  ## 设置可执行文件的路径
[root@localhost ~]# export LD_LIBRARY_PATH=/usr/local/pgsql/lib  ##设置共享库的路径
## --with-perl 加上这个选项，才能使用perl语法的自定义函数，一般都需要。
## --with-python 同上。
## -s: symbole 软链接，-f: force 如果目标链接已经存在则删除重新建立
## 如果9.5.4的版本发布了，升级只需停掉数据库在这里改下软链接地址就可以了
## 可将上面的语句加入到系统环境或者用户环境变量的配置文件中
```

## 常见问题FAQ
源码安装一般会出现一些依赖包的问题，因为每个系统的已有环境都不太一样，下面我列出了我碰到的几种依赖的问题，可以参考解决：
``` bash
[root@localhost ~]# yum install -y redline
[root@localhost ~]# yum install -y readline-devel
[root@localhost ~]# yum install -y zlib-devel
[root@localhost ~]# yum install -y perl-devel
[root@localhost ~]# yum install -y perl-ExtUtils-*
[root@localhost ~]# yum install -y python-devel
```
其实思路一般都是这样：
1. 根据报错信息得到缺少的”软件包名“
2. 检查下系统是否已经安装了这个“软件包名” `rpm -qa|grep "软件包名"`
3. 上一步的结果如果是没有，则执行`yum install 软件包名`安装，如果有则一般代表需要安装他的开发包版本，则执行`yum install 软件包名-devel`
当如在未安装前，你可以根据需要安装软件的文档先检查一遍系统的依赖。

## 数据库配置、初始
### 创建数据库簇
``` bash
[root@localhost ~]# export PGDATA=/mnt/db/pgsql
[root@localhost ~]# groupadd postgres
[root@localhost ~]# useradd -g postgres postgres
[root@localhost ~]# chown -R postgres:postgres/mnt/db/pgsql
[root@localhost ~]#  su -postgres
[postgres@localhost ~]$ initdb
## 为了系统安全，pgsql不允许使用root来初始化数据库，故新建postgres
## su 与 su - 是有区别的，su只是切换了用户的权限，su - 是切换了权限并且获取了对应用户的环境变量，上面的一些export 变量其实我们都可以防止到postgresql的用户环境变量下，方便使用postgres用户专门操作pgsql数据库
```

### 安装contrib下的工具
contrib 下面的工具比较实用，一般都会安装上。
``` bash
[postgres@localhost ~]# exit ##退出postgresql 用户，使用root用户来编译安装
[root@localhost ~]# cd /usr/local/src/postgresql-9.2.4/contrib/
[root@localhost ~]# make
[root@localhost ~]# make install
```
### 启动和停止数据库
``` bash
[root@localhost ~]# su - postgres## 编译安装完了后，到了日常的系统维护，就养成切换回专门的用户来操作吧
[postgres@localhost ~]$ pg_ctl start -D $PGDATA
[postgres@localhost ~]$ pg_ctl stop -D $PGDATA [-m SHUTDOWN-MODEL]
## SHUTDOWN-MODEL: smart 等待所有连接终止后关闭数据库； fast：快速关闭数据库，断开客户端连接，让已有事务回滚，相对于oracle的immediate; immediate: 立即关闭数据库，下次启动时进行恢复，相对于oracle 的abort模式； 这里oracle dba注意区别一下。
```

### 修改监听IP和端口
在**数据目录下**编辑 `postgresql.conf`文件，找到如下内容：
``` bash
#listen_address = 'localhost'
#port = 5432
```
其中`listen_address`表示监听的IP地址，默认是在“localhost”处监听，这会让远程主机无法登录这台数据库，可以修改成具体的内网地址，或者直接改成`*`,表示本地所有地址上监听。

参数`port`表示监听的数据库端口号，默认为`5432`。如果一台机器需要安装几个数据库实例，可以调整这里的端口。

修改这两个参数，需要重启数据库生效。

### 与数据库log相关的参数
``` bash
# 日志的收集一般是打开的
logging_collector = on

# 日志的目录，一般默认即可
log_directory = 'pg_log'

# 日志的切换和覆盖可以使用以下几种方案

# 1. 每天生成一个新的日志文件
log_filename = 'postgresql-%Y-%m-%d_%H%M%S.log'
log_truncate_on_rotation = off
log_rotation_age = 1d

# 2. 每当写满一定大小(如10M)，则切换一个日志
log_filename = 'postgresql-%Y-%m-%d_%H%M%S.log'
log_truncate_on_rotation = off
log_rotation_age = 0
log_rotation_size = 10M

# 保留7天的日志进行循环覆盖
log_filename = 'postgresql-%a.log'
log_truncate_on_rotation = on
log_rotation_age = 7d
log_rotation_size = 0
```

### 内存参数设置
PostgreSQL可以修改以下两个主要参数:
- shared_buffers: 共享内存的大小，主要用于共享数据块
- work_mem: 单个SQL执行时，排序、hash join 所使用的内存，SQL运行完后，内存就释放了。
