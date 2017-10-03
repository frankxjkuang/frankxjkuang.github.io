---
layout: post
title:  git提交代码之后自动打tag标签
date:   2017-10-03 13:29:00 +0800
categories: document
tag: log
---

* content
{:toc}


顾名思义post-commit就是commit之后执行的钩子，此处我用来自动打tag标签,新建一个post-commit文件


```git
#!/bin/sh
tag=$(git describe --tags `git rev-list --tags --max-count=1`)
num=$(git rev-list HEAD --count)
let num+=1
echo 'Create New Tag v.'$num
git tag -a 'proj@v.'${num} -m test
git push origin 'proj@v.'${num}
```
