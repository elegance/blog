---
title: jstat jvm gc
tags:
    - 垃圾回收
categories:
    - java
---

1. 查看在运行的java应用
```bash
jps -v
```
2. 查看GC信息
```bash
jstat -gcutil pid [interval]
```

### 例子
```bash
[root@test ~]# jstat -gcutil 29458
  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT   
  5.41   0.00   5.89  19.73  97.63  94.64    120    1.156     3    0.308    1.464
```
意义分别为：survivor0使用占比、Survivor1使用占比、Eden使用占比、Old使用占比、Meta使用占比、Compressed Class Space、YGC次数、YGC耗时、FGC次数、FGC耗时、GC总耗时
