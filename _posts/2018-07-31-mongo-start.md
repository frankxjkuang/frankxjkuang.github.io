---
layout: post
title:  初入 mongo，第一次 nosql
date:   2018-07-31 21:44:00 +0800
categories: mongo
tag: note
---

* content
{:toc}


之前大学的时候必修 “SQL”，现在还是隐约记住 `SELECT * FROM table WHERE condition`，什么关系表、主键、外键都还记得一点点（哈哈）...说了这么多，不知所云，前段时间学了下 [mongodb](https://docs.mongodb.com/) ，忘了记笔记了，所以这里主要是记录一下，也算是学过（我好幽默）。下面主要就是怎么安装，配置以及基本使用。


# 安装
我用的是 `brew` 安装的，[官网的链接](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-os-x/?_ga=2.58309098.1733521690.1532955043-1818517193.1522632268)讲的比较清楚的了。

这里多说一句，`brew` 默认的安装路径是 `/usr/local/Cellar/mongodb`，安装完毕之后会提示你需要修改mongodb的配置信息可以执行命令 `mongod --config /usr/local/etc/mongod.conf`

# 配置
上面提到的 `mongod.conf` 文件就是配置文件，这里我们先不着急配置，先新建数据库的文件夹，之后我们会使用这个地址作为配置信息里的数据库地址（配置文件默认dbPath: /usr/local/var/mongodb）

官方都是在根目录下新建的目录那么我们也这样吧：

```bash
mkdir -p /data/db
```

这个时候启动 `sudo mongod`，可能会报错 `Permission denied（权限拒绝）` 因此我们需要获取数据库目录的操作权限

```bash
sudo chown -R <username> /data

# 查看当前 username
whoami
```

> **Tip:** 这里提一下，如果你愿意的话可以在 `/data` 目录下新建一个 `log` 目录，里面新建一个 `mongo.log` 文件用来指定替换默认日志目录

修改 `mongod.conf` 文件，默认路径为 `/usr/local/etc/mongod.conf`，注释或者删除内容，以下是比较全的配置信息，选择需要的（或者全部）拿过去：

```bash
# Only accept local connections
bind_ip = 127.0.0.1
# 数据库路径
dbpath = /data/db
# 日志输出文件路径，这里可以用上面 Tip 说的地址替换（如果你做了的话）
logpath = /usr/local/var/log/mongodb/mongo.log
# 错误日志采用追加模式，配置这个选项后mongodb的日志会追加到现有的日志文件，而不是从新创建一个新文件
logappend = true
# 启用日志文件，默认启用
journal = true
# 这个选项可以过滤掉一些无用的日志信息，若需要调试使用请设置为false
quiet = false
# 是否后台启动，有这个参数，就可以实现后台运行
fork = true
# 端口号 默认为27017
port = 27017
# 指定存储引擎（默认不需要指定）
storageEngine = mmapv1
# 开启网页日志监控，有这个参数就可以在浏览器上用28017查看监控界面
httpinterface = true
```

# 把 mongodb/bin 加入 $PATH ，以免我们每次输入 sudo monogd，变成直接 monogd

```bash
vim ~/.zshrc

# 加入以下内容
export PATH=/usr/local/Cellar/mongodb/4.0.0/bin:${PATH}
```

# 启动

```bash
mongod

# 再开启一个终端
mongo

# 基本操作
# 显示所有数据库
show dbs

# 创建或者使用数据库（有名字则使用，没有则）
use <dbname> 

# 显示已经存在的数据库的表
show collections  
# 或者 show tables  

# 创建集合 相当于创建表
db.createCollection('collectionName')

# 插入数据(举个栗子)
db.collection.save({name: "xx", age: 18}}})

# 推出
exit
# 或者 
use admin
db.shutdownServer()
```

# 总结

这里简单几步操作就已经初步开始上手了，继续深入学习，基本也就跟 `SQL` 的数据操作套路差不多了，稍微看了下文档感觉没什么好说的，万事开头难，后面修行看个人了...

