---
title: Linux操作常用命令
date: 2017-04-12 10:33:08
tags: Linux
---

记录一些常用的技巧，隔一段时间不用又给淡忘了。

### 打包压缩、解压
```shell
// .tar.bz2 包
# tar -jcv -f /root/test/etc.tar.bz2 /etc/
# tar -jxv -f  /root/test/etc.tar.bz2  -C /tmp/

// .tar.gz 包
# tar -zcv -f /root/test/etc.tar.gz /etc/
# tar -zxv -f  /root/test/etc.tar.gz  -C /tmp/
```

### 建立链接
```bash
# ln -sf /usr/local/postgresql-9.2.4 /usr/local/pgsql
## -s: symbole 软链接，-f: force 如果目标链接已经存在则删除重新建立
## 如果9.2.5的版本发布了，升级只需停掉数据库在这里改下软链接地址就可以了
```

### 当前目录 各文件夹 大小 linux
```bash
# find $1 -maxdepth 1 | xargs du -sh
```

### 强制停止应用的两种方式
```bash
# ps -ef|grep java|grep 'tomcat-web-api'|awk '{print $2}'|xargs kill -9

# ps -ef|grep java|grep 'tomcat-web-api'|awk '{system("kill -9 " $2)}'
```

### 某次服务器中毒，删除与病毒文件同样大小字节的文件
```bash
# find / -size 1223123c | xargs rm -rf
```

### find类 其他
```bash
// 正则匹配删除
# find ./ -maxdepth 1|grep "2016-12-1[2-7]"|xargs rm -rf

// 反向匹配，找出名字不含 "tar" 的文件/文件夹
# find ./ -maxdepth 1|grep "tar" -v
```

### 根据进程获取进程号，打印进程的运行信息
```bash
# pgrep mysql | xargs -I {} ls -l /proc/{}
```

### crontab 同步时间，
```bash
// crontab 分、时、日、月、周
// 10分钟同步一次
# */10 * * * * /usr/sbin/ntpdate time.windows.com && hwclock -w
```

### 输出磁盘的读写情况
```bash
# iostat -k 2
```

### 新增磁盘
```bash
// fdisk: 操作磁盘分区表
// mkfs：对磁盘分区进行文件系统格式化

// 列出磁盘装置，找到我们要操作的装置名称，便于对其进行分区
# fdisk -l 

// 开始对磁盘进行分区处理，注意不要加上数字
# fdisk /dev/xvdb  

// 对分区进行格式化
# mkfs -t ext4 -c /dev/xvdb1

// 建立磁盘挂载目录
# mkdir /data

// 手动挂载
# mount /dev/xvdb1 /data/

// 开机自动挂载
# echo "/dev/xvdb1 /data/ ext4 defaults 0 0" >> /etc/fstab
```

### 看你在Linux下最常用的命令是哪些？
```bash
history | awk '{CMD[$2]++;count++;} END { for (a in CMD )print CMD[ a ]" " CMD[ a ]/count*100 "% " a }' | grep -v "./" | column -c3 -s " " -t |sort -nr | nl | head -n10
```