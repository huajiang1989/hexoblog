
---
title:  Git问题Everything up-to-date解决
category: git
tags: git
---

今天提交代码的时候遇到了一个小问题，这里解决了记下小记。

提交代码遇到『Everything up-to-date』

出现这个问题的原因是git提交改动到缓存，要push的时候不会将本地所有的分支都push掉，所以出现这个问题。我们应该告诉git提交哪个分支。
<!--more-->
这里有种特殊的情况是如果你是fork别人的仓库再clone到本地的话，即使git上只有一个主分支，他还是可能出现这个错误。那么我们就需要新建分支提交改动然后合并分支。

接下来先创建一个新分支提交改动
```bash
$ git branch newbranch
```
然后输入这条命令检查是否创建成功
```bash
$ git branch
```
这时终端输出
```bash
  newbranch
* master
```
这样就创建成功了，前面的*代表的是当前你所在的工作分支。我们接下来就要切换工作分支。
```bash
$ git checkout newbranch
```
这样就切换完了，可以 $ git branch 确认下。然后你要将你的改动提交到新的分支上。
```bash
$ git add .
$ git commit -a
```
此时可以 $ git status 检查下提交情况。如果提交成功，我们接下来就要回主分支了，代码和之前一样。
```bash
$ git checkout master
```
然后我们要将新分支提交的改动合并到主分支上
```bash
$ git merge newbranch
```
合并分支可能产生冲突这是正常的，虽然我们这是新建的分支不会产生冲突，但还是在这里记录下。下面的代码可以查看产生冲突的文件，然后做对应的修改再提交一次就可以了。
```bash
$ git diff
```
我们的问题就解决了，接下来就可以push代码了。
```bash
$ git push -u origin master
```
删除新建的这个分支
```bash
$ git branch -D newbranch
```
如果想保留分支只是想删除已经合并的部分只要把大写的D改成小写的d就行了。