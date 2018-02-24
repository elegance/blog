---
title: n2n点对点通信
draft: true
date: 2018-01-04 11:23:12
tags: n2n
---

#### N2N介绍

通过N2N即可组成局域网，在外面可以访问家里的路由器、机器。

* Supernode 中心节点，并不参与两台主机间直接通信, 只是起到媒人的作用。
* Edge 节点都会建立tun/tap虚拟网卡，用作n2n网络的入口，Edge节点就可以互通了。

#### V1 与 V2

n2n有V1和V2两个版本， 两个版本不兼容，据说V1的性能还略高于V2,V2是增加了一些安全相关的提升。

所以我这里都是**基于V1版本**搭建的。

#### Linux Supernode 的安装

```bash
$ git clone https://github.com/meyerd/n2n
$ cd n2n/n2n_v1
$ make
$ make install 2>&1 | tee  make.log
```

启动：

```bash
$ nohup supernode -l 86 -v -f > supernode.log &
```

#### 

#### Ubuntu Edge 的安装

```bash
$ git clone https://github.com/meyerd/n2n
$ cd n2n/n2n_v1
$ make
$ sudo make install
$ sudo su
$ apt install uml-utilities
$ tunctl -t tun0
$ sudo edge -d n2n0 -c orh -k 123 -m c2:27:ad:05:b3:a5 -a 10.8.1.7 -l 104.128.82.194:86 -r -f
```

#### Openwrt  Edge的安装

这里我是在虚拟机中安装的`Openwrt`可以先下载虚拟机文件，虚拟机

`http://downloads.openwrt.org/attitude_adjustment/12.09/x86/generic/openwrt-x86-generic-combined-ext4.vdi`

安装与配置：

```bash
$ opkg update
$ opkg install n2n
$ 
$ vim /etc/config/n2n
$ option ipaddr           '10.2.5.1'
$ option supernode        '104.128.82.194'
$ option port             '86'
$ # 为自己的N2N网络组织机构取个名字
$ option community        'orh'
$ # 其他设备要使用相同的组织机构名和密码才能加入
$ option key              '123'
$ option route            '1'
```

启用，启动、停止：

```bash
/etc/init.d/n2n enable
/etc/init.d/n2n start
/etc/init.d/n2n stop
```



参考文章：

* [P2P网络-n2n穿墙](http://gohom.win/2016/09/03/n2n-p2pnet/)
* [OPKG包管理](https://wiki.openwrt.org/zh-cn/doc/techref/opkg)
* [Openwrt离线包](http://openwrt.jaru.eu.org/chaos_calmer/ar71xx/packages/)