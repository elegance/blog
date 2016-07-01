---
title: centos6.5-x64.subversion安装文档
date: 2016-06-29 16:23:47
categories:
- Linux

tags:
- Centos软件安装

---

### 一、准备安装包
一共5个包，下面的都可以点击链接直接下载，都是当下（2016年6月3日）最新版本，提供了具体来源的下载页，如有版本控可到提供的下载页下载最新包。

* 源码包: [subversion-1.9.4.tar.bz2](http://mirrors.hust.edu.cn/apache/subversion/subversion-1.9.4.tar.bz2)，下载页：http://subversion.apache.org/download.cgi

* apr依赖: [apr-1.5.2.tar](http://mirrors.hust.edu.cn/apache//apr/apr-1.5.2.tar.bz2)，下载页：http://apr.apache.org/download.cgi

* apr-util依赖: [apr-util-1.5.4.tar.bz2](http://mirrors.hust.edu.cn/apache//apr/apr-util-1.5.4.tar.bz2)，下载页：http://apr.apache.org/download.cgi (同上)

* sqlite依赖: [sqlite-amalgamation-3130000.zip](http://www.sqlite.org/2016/sqlite-amalgamation-3130000.zip)，下载页：http://www.sqlite.org/download.html

* zlib依赖 [zlib-1.2.8.tar.xz](http://netassist.dl.sourceforge.net/project/libpng/zlib/1.2.8/zlib-1.2.8.tar.xz)，下载页：http://www.zlib.net/



### 二、安装
我把上面下载的包都放置到了目录： `~/svn-install`，如有目录差别，自行调整。

```bash
[root@localhost svn-install]# tar -jxvf subversion-1.9.4.tar.bz2 -C /usr/local/src/
[root@localhost svn-install]# tar -jxvf apr-1.5.2.tar.bz2 -C /usr/local/src/
[root@localhost svn-install]# tar -jxvf apr-util-1.5.4.tar.bz2 -C /usr/local/src/
[root@localhost svn-install]# tar -xvf zlib-1.2.8.tar.xz -C /usr/local/src/
[root@localhost svn-install]# unzip sqlite-amalgamation-3130000.zip -d /usr/local/src/subversion-1.9.4/
[root@localhost svn-install]# cd /usr/local/src/subversion-1.9.4/
[root@localhost subversion-1.9.4]# mv sqlite-amalgamation-3130000 sqlite-amalgamation
[root@localhost subversion-1.9.4]# cd /usr/local/src/apr-1.5.2/
[root@localhost apr-1.5.2]# ./configure --prefix=/usr/local/apr-1.5.2/
[root@localhost apr-1.5.2]# make && make install
[root@localhost apr-1.5.2]# cd /usr/local/src/apr-util-1.5.4/
[root@localhost apr-util-1.5.4]# ./configure --prefix=/usr/local/apr-util-1.5.4/ --with-apr=/usr/local/apr-1.5.2/
[root@localhost apr-util-1.5.4]# make && make install
[root@localhost apr-util-1.5.4]# cd /usr/local/src/zlib-1.2.8/
[root@localhost zlib-1.2.8]# ./configure --prefix=/usr/local/zlib-1.2.8/
[root@localhost zlib-1.2.8]# make && make install
[root@localhost zlib-1.2.8]# cd /usr/local/src/subversion-1.9.4/
[root@localhost subversion-1.9.4]# ./configure --prefix=/usr/local/subversioni-1.9.4 --with-apr=/usr/local/apr-1.5.2/ --with-apr-util=/usr/local/apr-util-1.5.4/ --with-zlib=/usr/local/zlib-1.2.8/
[root@localhost subversion-1.9.4]# make
[root@localhost subversion-1.9.4]# make install

```

### 三、配置
#### 1. 创建svn仓库
`svnadmin create /mnt/svn/repos`

#### 2. 仓库权限用户配置
1. 用户配置 `passwd`文件
```
[root@localhost conf]# vim passwd 
[users]
peter=123456
john=112233
```
2. 权限配置`authz`文件
```
[root@localhost conf]# vim authz 
[aliases]
# joe = /C=XZ/ST=Dessert/L=Snake City/O=Snake Oil, Ltd./OU=Research Institute/CN=Joe Average

[groups]
# harry_and_sally = harry,sally

# [/foo/bar]
# harry = rw
# &joe = r
# * =

# [repository:/baz/fuz]
# @harry_and_sally = rw
# * = r
[/]
peter=rw
john=rw
```

### 四、启动
1. 启动 `svnserve -d -r /mnt/svn/ --listen-port 9999`
2. 编写开机启动脚本（TODO）

配置可以参考这里：http://blog.csdn.net/typa01_kk/article/details/49226615
