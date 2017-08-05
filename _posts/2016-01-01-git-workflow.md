---
layout: post
title:  git 工作流
date:   2016-08-27 01:08:00 +0800
categories: document
tag: 教程
---

* content
{:toc}


工作中常用的 git 命令
====================================

先上一张经典的git工作空间的流程图

![git工作流程图](http://ac-hjXcMNw4.clouddn.com/4eb960c8dc714e033aec.png)

git工作空间
------------------------------------

>git的优点就不用多说了，本文主要讲工作中常用的命令，git命令就是用以在以下各个工作空间进行代码管理

+ workspace: 本地工作区（用户开发的地方）
+ Index / Stage: 暂存区
+ Repository: 本地仓库
+ Remote: 远程仓库

新建代码库
====================================

```git
# 本地新建代码库(当前目录初始化为代码库或者指定项目名将其初始化为代码库)
$ git init [, project-name]

# 从远程仓库克隆一个项目
$ git clone [url]
```
> 代码库新建成功后会出现一个`.git`目录（该目录是默认隐藏的，使用`ls -ah`命令就可以看见），该目录是用来跟踪管理版本库的，

代码提交
====================================

```git
# 以下是基本的提交操作

# 查看当前分支的更改状态
$ git status

# 查看具体更改哪些内容(可以指定某个更改了的文件)
$ git diff [, file]

# 提交修改(可以指定文件)放入暂存区，或者`add . `表示全部提交
$ git add [, file]

# 提交新文件
$ git commit -m 'information about commit'
```
> **PS:**一般情况下使用`git commit`之后在进行详细的更改描述，使用`i`进行插入，信息写完之后按`Esc`再然后输入`:wq`回车，保存退出

版本回退
====================================

```git
# 查看提交日志(显示的是从最近到最远的提交信息)
$ git log

commit 5632164fb231179f24e0882e6d48395383f8f1e0
Author: frankxjkuang <frankxjkuang@gmail.com>
Date:   Tue Jan 20 15:11:49 2013 +0800

    first

commit ed233c827ed32a8a34578d5496d7dcd576c5ee85
Author: frankxjkuang <frankxjkuang@gmail.com>
Date:   Tue Jan 20 14:53:12 2013 +0800

    secend

# 信息太多了可以加上参数
$ git log --pretty=oneline
5632164fb231179f24e0882e6d48395383f8f1e0 first
ed233c827ed32a8a34578d5496d7dcd576c5ee85 secend

# 回到上一个版本
$ git reset --hard HEAD^

# 那如果此时又想回到未来的版本，就可以使用刚才的`commit id`(不需要写全，只要能在唯一性区分就够了)
$ git reset --hard ed233c827e

# 如果你不幸的关掉了命令框，忘记了`commit id`怎么办？
# `git reflog`会记录你的每一次命令
$ git reflog
ji74512 HEAD@{0}: reset: moving to HEAD^
ed233c8 HEAD@{2}: commit: second
5632164 HEAD@{1}: commit: first
fr54863 HEAD@{3}: commit (initial): wrote a readme file

# 此时你可以找到对应的commit id回到指定的版本了

```
> **PS:**commit之后的一串字符串是commit id，是和Git分布式的版本控制系统相关的，当然此处我们主要用来进行版本控制

撤销修改
====================================

```git
# 让指定文件回到最近一次git commit或git add时的状态
$ git checkout -- [file]

# 让所有文件回到最近一次git commit或git add时的状态
$ git checkout -- .

# 在某些情况下你只是`add`到了暂存区还未进行`commit`
# 可以使用以下命令把暂存区的修改撤销掉（unstage），重新放回工作区
$ git reset HEAD [file]
```

删除文件
====================================

```git
# 删除工作区中的文件
$ rm [file]

# 删除版本库中的文件
$ git rm [file]

# 用版本库里的版本替换工作区的版本
$ git checkout -- [file]

# 在某些情况下你只是`add`到了暂存区还未进行`commit`
# 可以使用以下命令把暂存区的修改撤销掉（unstage），重新放回工作区
$ git reset HEAD [file]
```

分支管理
====================================

+ 历史分支：主分支master存储了正式发布的历史，开发分支develop功能的集成分支
+ 功能分支：从develop分支上切出来，每一个新功能都应该是一个独立的分支，开发完成合并回develop分支
+ 发布分支：开法一个版本完成，从develop上fork一个发布分支release，所有的bug修复和文档生成等都应该在该分支上进行，完成后将release分支融合到master,另外所有的修改应该合并带develop分支


创建分支
------------------------------------

```git
# 创建开发分支并建立跟踪分支
$ git checkout -b develop origin/develop

# 新功能的开发需要从develop分支上切
$ git checkout -b some-feature develop

# 功能分支开发完成需要融合到开发分支
$ git checkout develop
$ git merge some-feature
$ git push

# 功能分支的使命完成，删除功能分支
$ git branch -d some-feature
```
> **PS:**切换到develop分支在融合功能分支之前，最好先`git pull`,develop分支是最新的

拉取远程分支(本地不存在的分支)
------------------------------------

```git
# 更新所有分支
$ git fetch

# 创建一个新的本地分支，并与指定的远程分支关联起来
$ git checkout -b <local-branch> origin/<remote-branch>
```
> **PS:**使用`git branch`可以查看所有的本地分支

删除远程分支和tag
------------------------------------

```git
# 删除远程分支
$ git push origin --delete <branchName>
# 也可以推送空的分支进行覆盖也相当于删除
$ git push origin :<branchName>

# 删除远程tag
$ git push origin --delete tag <tagname>
# 或者简写如下
$ git tag -d <tagname>
```

git分支重命名
------------------------------------

```git
# 重命名本地分支
$ git branch -m <oldBranchName> <newBranchName>
```
> **PS:**重命名远程分支，相当于删除远程分支，重命名本地分支，推送本地重命名的分支

```git
# 重命名远程分支
# 删除远程分支
$ git push --delete origin <branchname>

# 重命名本地分支
$ git branch -m <oldBranchName> <newBranchName>

# 推送本地重命名分支
$ git push origin <newBranchName>
```

git批量删除分支
------------------------------------

```git
# 批量删除名称具有bugfix的分支
$ git branch |grep 'bugfix' |xargs git branch -d
```
+ git branch 输出当前分支列表
+ grep 是对 git branch 的输出结果进行匹配，匹配值当然就是 bugfix
+ xargs 的作用是将参数列表转换成小块分段传递给其他命令

> 因此该命令的意思是：从分支列表中匹配到指定分支，然后一个一个(分成小块)传递给删除分支的命令，最后进行删除。