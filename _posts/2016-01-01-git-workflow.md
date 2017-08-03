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