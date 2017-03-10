---
title: git常用命令
tags:
---

项目中经常用到的是svn，虽然一些git的基础经常用到的命令比较熟悉，但是一些比较重要，用的少的命令容易淡忘

<hr>

### 回退

1. 没有push
    1. 代码提交了，还没push，场景： 暂存区里面 多`add`了一些与这次`commit`无关的文件，想要撤销`commit`，重新`add`文件再`commit`
        ```bash
        $ git reset --mixed HEAD^ #会保留源码,只是将git commit和index 信息回退到了某个版本
        ```

2. 已经push