---
layout: post
title:  macOS git自动补全命令分支名
date:   2018-06-04 11:08:00 +0800
categories: git
tag: course
---

* content
{:toc}


一、 安装 bash-completion
====================================

```shell
// 安装
brew install bash-completion

// 卸载
brew uninstall bash-completion
```

[Homebrew](https://brew.sh/index_zh-cn)官网，如果你好未安装 `Homebrew` 的话可以参考以下命令（以官网为准哦）

```shell
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

安装完成后查看信息
```shell
brew info bash-completion
```

之后会输出类似以下信息
```shell
==> Caveats
Add the following line to your ~/.bash_profile:
  [ -f /usr/local/etc/bash_completion ] && . /usr/local/etc/bash_completion
```

复制下面这句话添加到~/.bash_profile（如果没有该文件，新建一个）
```shell
[ -f /usr/local/etc/bash_completion ] && . /usr/local/etc/bash_completion
```

```shell
vim ~/.bash_profile
```

二、 拷贝文件并设置路径
====================================
访问[github](https://github.com/git/git.git)将'contrib/completion/'目录下的 `git-completion.bash` 拷贝到本地文件内（记住地址），


```shell
source <local path>/.git-completion.bash
```

三、 启动终端
====================================

输入 `git` 命令、分支名就可以自动补全了