---
title: svn 分支管理
date: 2017-03-08 17:20:15
tags:
---


[参考](http://www.iteye.com/topic/28013)

## 1. svn 初始目录结构

```bash
D:\pro-repo
├─branches
├─tags
└─trunk
```

如果没有对应的结构目录，可以使用以下命令来创建：
```bash
svn mkdir branches
```

## 2. branchs/tags 的建立
需求一：产品开发已经基本完成，并且通过很严格的测试，这时候我们就想发布给客户使用，发布我们的1.0版本：

```bash
svn copy svn://server/trunk svn://server/tags/release-1.0 -m "1.0 released"  
```

需求二: 需要开发一个新的功能
```bash
svn copy svn://server/trunk svn://server/branches/fea-01 -m "新开分支开发01新需求" 
```

咦，这个和branches有什么区别，好像啥区别也没有？ 
是的，branches和tags是一样的，都是目录，只是我们不会对这个release-1.0的tag做修改了，不再提交了，如果提交那么就是branches 

需求三：有一天，突然在线上发现一个致命的bug,那么所有的branches一定也一样了，该怎么办？ 
首先我们是从线上发现的，那么我第一步先：
```bash
svn copy svn://server/tags/release-1.0 svn://server/branches/bugfix-01 -m "1.0 线上bug (copy from tags/release-1.0) " 
```
然后我们拉去分支`bugfix-01`代码到本地，开发、测试，如果没有问题的话，合并分支到`在生命周期`的分支，这里是：`branches/fea-01`、`trunk/`
```
$ svn -r 6:7 merge  https://orh-vm-pc/svn/test-tzbms/branches/bugfix-01 trunk # bug 合并至主干
$ svn -r 0:HEAD merge https://orh-vm-pc/svn/test-tzbms/trunk branches/fea-01 # 主干合并至需求分支
$ svn delete  https://orh-vm-pc/svn/test-tzbms/branches/bugfix-01 -m "删除-bug-01"


```
新的bug产生时(上次升级没有打tag，直接取trunk):
```bash
$ svn copy https://orh-vm-pc/svn/test-tzbms/trunk https://orh-vm-pc/svn/test-tzbms/branches/bugfix-02 -m "1.0 线上bug (copy from trunk) "
$ cd trunk/
$ svn merge https://orh-vm-pc/svn/test-tzbms/branches/bugfix-02 # bug 合并至主干 （和上面的指定版本功能效果一样，这里省去了指定版本的麻烦）
$ svn merge https://orh-vm-pc/svn/test-tzbms/trunk # 主干合并至分支
```

合并后对 trunk 严格测试没问题的话，就可以发布`release-1.1`了。


## 3. 几个问题

#### 1. 考虑到部署release时，每次都要打上tag，会略显繁琐， `tags` 不一定非得有，也就是上面用`trunk`作为线上分支

#### 2. 上面有提到`在生命周期`的分支，这是为了强调`branchs`下的分支尽可能缩短其生命周期，因为在生命周期的分支都得维护，线上bug、其他分支新功能这些代码所有修改都得合并入`在生命周期`的分支
在生命周期的分支，一般有：
```bash
trunk/

branches/fea-01
branches/fea-02
branches/fea-....

branches/bugfix-01
```
同时开发的需求`fea-n`可能有几个，`bugfix`线上bug通常就作为一个了。

#### 3. 冲突解决
如果一个文件在两个分支上都被修改，那么合并两个分支时将可能发生冲突。

之所以说可能发生冲突，是因为如果两个分支修改的文件明显不同，`merge`会自动的帮我们处理好。比如一端对文件第一行修改了内容，一端对文件增加了一行内容这种情况。

当文件发生冲突时可以有以下方式来处理：

#### 工具设置

我们可以使用`TortoiseSVN`设置下对比工具来方便我们compare和merge，比如`BeyondCompare`

操作步骤：`项目目录-右键`  ---> `TortoiseSVN` ---> `Settings` ---> `Diff Viewer`
![Diff-tool-set](/images/svn-branch-mgt/diffViewer-setting.jpg)
其中代码：
```
"D:\tools\BeyondCompare\Beyond Compare\BCompare.exe" %base %mine /title1=%bname /title2=%yname /leftreadonly
"D:\tools\BeyondCompare\Beyond Compare\BCompare.exe"
```
合并工具设置：
![merge-tool-set](/images/svn-branch-mgt/merge-tool-setting.jpg)
其中代码：
```
"D:\tools\BeyondCompare\Beyond Compare\BCompare.exe"  %mine %theirs %base %merged /title1=%yname /title2=%tname /title3=%bname /title4=%mname
```

#### 解决冲突
![edit-conflict](/images/svn-branch-mgt/edit-conflicts.jpg)
![merging](/images/svn-branch-mgt/merging.jpg)
在上一步合并完后，就可以标记解决完冲突了：
![resolve](/images/svn-branch-mgt/resolve.jpg)

~~#### 3. `merge`的几种方式及应用场景， 参考[svn分支合并类型](http://chunanyong.iteye.com/blog/697255)~~
#### 3. `merge` 我使用merge的两种方式
1. 指定分支地址，合并至当前目录:
```bash
# 将 `bug-01`分支 合并到 `trunk`
$ cd trunk/
$ svn merge https://server/svn/pro/branches/bug-01

# 将 `trunk` 合并至 分支 `fea-01`
$ cd ../branches/fea-01
$ svn merge https://server/svn/pro/trunk
```
2. 指定分支地址和版本，合并至指定分支目录：
```bash
# 将 bugfix-01 分支 合并至trunk
$ svn -r 6:7 merge  https://orh-vm-pc/svn/test-tzbms/branches/bugfix-01 trunk/

# 强 trunk 合并至 需求分支
$ svn -r 0:HEAD merge https://orh-vm-pc/svn/test-tzbms/trunk branches/fea-01
```


注意：

**以上的`svn copy`命令会直接发给服务端执行，也就是说操作即时生效，无需commit**

`svn merge` merge动作是在本地执行的，所以可以放心执行，如果有出现失误，不提交clean 重新merge即可。

## Other
```bash
$ svn copy https://orh-vm-pc/svn/test-tzbms/trunk https://orh-vm-pc/svn/test-tzbms/branches/fea-01 -m "01需求分支"
Committed revision 4.
Administrator@DIY-B-PC MINGW64 /d/svn-test
$ svn copy https://orh-vm-pc/svn/test-tzbms/trunk https://orh-vm-pc/svn/test-tzbms/tags/release-1.0 -m "1.0发布了"
Committed revision 5.
$ svn copy https://orh-vm-pc/svn/test-tzbms/tags/release-1.0 https://orh-vm-pc/svn/test-tzbms/branches/bugfix-01 -m "线上bug 01"
Committed revision 6.
$ svn commit -m "bug 解决提交"
Committed revision 7.
$ svn update
Updating '.':
At revision 7.
$ svn -r 6:7 merge  https://orh-vm-pc/svn/test-tzbms/branches/bugfix-01 trunk
--- Merging r7 into 'trunk':
U    trunk\api\api.html
--- Recording mergeinfo for merge of r7 into 'trunk':
 U   trunk
$ svn commit -m "合并bug-01分支修改至主干"
Sending        branches\bugfix-01
Sending        trunk
Sending        trunk\api\api.html
Transmitting file data .
Committed revision 8.
Administrator@DIY-B-PC MINGW64 /d/svn-test
$ svn status
M       branches\fea-01\api\api.html

Administrator@DIY-B-PC MINGW64 /d/svn-test
$ svn commit -m "需求分支-团队合作的普通提交"
Sending        branches\fea-01\api\api.html
Transmitting file data .
Committed revision 9.
$ svn merge trunk@head branches/fea-01@head branches/fea-01
--- Merging differences between repository URLs into 'branches\fea-01':
 G   branches\fea-01
--- Recording mergeinfo for merge between repository URLs into 'branches\fea-01':
 U   branches\fea-01
```