---
layout: post
title:  MongoDB索引简记
date:   2019-04-04 11:44:00 +0800
categories: mongo
tag: note
---

* content
{:toc}


#### 查询集合的索引信息
```
db.wp_user_device.getIndexes()

[
    {
        "v" : 1,                 // 索引版本
        "key" : {                // 索引字段以及排序方向
            "_id" : 1            // 根据_id升序索引
        },
        "name" : "_id_",          // 索引的名称
        "ns" : "wp.wp_user_device" // db.col
    }
]
```

#### 索引类型
##### 单字段索引（Single Field Index）
```
// 对iemi字段创建升序索引（降序：-1）
db.wp_user_device.createIndex({iemi: 1})
```

- 对于单字段索引升序/降序效果
- mongoDb默认创建的id索引也是这种类型

##### 复合索引（Compound Index）
针对多个字段联合创建索引，先按第一个字段排序，第一个字段相同的文档按照第二个字段排序，依次类推
```
// 对age、name字段创建一个复合索引
db.person.createIndex({age: 1, name: 1})
```

age字段的取值很有限，即使有相同age字段的文档很多；而name字段的取值则很丰富，拥有相同name字段的文档很少；显然先按name字段查找再在相同name的文档里查找age字段更为高效

- 复合索引能匹配符合索引前缀的查询，比如：`db.person.find({age: 18})`，但是`db.person.find({name: "frank"})`则不能

##### 多key索引（Multikey Index）
当索引的字段为数组时，创建出的索引为多key索引，多key索引会为数组的每个元素建立一条索引，比如person表加入一个habbit字段（数组类型）用于描述兴趣爱好，需要查询有相同兴趣爱好的人就可以利用habbit字段的多key索引
```
// {name: "frank", age: 18, habbit: ["code", "write"]}
// 自动创建多key索引
db.person.createIndex({habbit: 1})

db.person.find({habbit: "football"})
```

##### 其它类型索引

- 哈希索引（Hashed Index）是指按照某个字段的hash值来建立索引，目前主要用于MongoDB Sharded Cluster的Hash分片，hash索引只能满足字段完全匹配的查询，不能满足范围查询
- 地理位置索引（Geospatial Index）就能很好的解决O2O的应用场景，比如【查找附近的美食】等
- 文本索引（Text Index）能解决快速文本查找的需求，比如有一个博客文章的集合，需要根据博客的内容来快速查找，则可以针对博客内容建立文本索引


**索引额外熟悉**MongoDB除了支持多种不同类型的索引，还能对索引定制一些特殊的属性

- 唯一索引（Unique Index）保证索引对应的字段不重复，比如_id
- TTL索引，可以针对某个时间字段，制定文档的过期时间
- 部分索引（Partial Index）只针对符合某个特定条件的文档建立索引
- 稀疏索引（Sparse Index）只针对存在索引字段的文档建立索引


#### 注意事项
索引并不是越多越好，因为索引文件本身要消耗存储空间，同时索引会加重插入、删除和修改记录时的负担，此外数据库在运行时也会消耗资源维护索引，一般以下两种情况不建议建索引：

第一种：表记录比较少，经验来说2000条为分界线吧
第二种：索引的选择性（Selectivity）较低，指不重复的索引值（也叫基数，Cardinality）与表记录数（#T）的比值：

`Index Selectivity = Cardinality / #T`

常见慢查询：
1. 不等于和不包含查询
2. 通配符在前面的模糊查询，like ""
3. 无索引的count查询和排序（复合索引顺序不匹配）
4. 多个范围查询（范围列可以用到索引（必须是最左前缀），但是范围列后面的列无法用到索引）
5. skip跳过过多的行数

#### 正确建立索引
在没有建立索引的情况下，对MongoDB数据表进行查询操作的时候，需要把数据都加载到内存。当数据的数据量达到几十上百万的时候，这样的加载过程会对系统造成较大的冲击，并影响到其他的请求处理

索引是对数据库表中一列或多列的值进行排序的一种结构，建立索引以后，对索引字段进行查询时，仅会加载索引数据，并提高查询速度

1. 建立合适的索引
为每一个查询建立合适的索引

组合索引是创建的索引由多个字段组成，例如：
`db.person.ensureIndex({name: 1, age: -1})` 

交叉索引是每个字段单独建立索引，但是在查询的时候组合查找，例如：
```
db.person.ensureIndex({name: 1})
db.person.find({name: "frank", age: 18})
```
交叉索引的查询效率较低，在使用时，当查询使用到多个字段的时候，尽量使用组合索引，而不是交叉索引。

2. 组合索引的字段排列顺序
当我们的组合索引内容包含匹配条件以及范围条件的时候，比如包含用户（匹配条件）以及年龄（范围条件），那么匹配条件应该放在范围条件之前

比如需要查询：
```
db.person.find({name: "frank", age: {$gt: 10}})
```
那么组合索引应该这样创建：
```
db.person.ensureIndex({name: 1, age: -1})
``` 

3. 查询时尽可能仅查询出索引字段
比如我们表有三个字段：username、age、mobile
索引是这样建立的：
```
db.person.ensureIndex({name: 1, age: -1})
```

仅需要查到某个用户年龄（age），可以这样写：
```
db.person.find({name: "frank"}, {_id: 0, age: 1})
// 不需要_id的时候直接忽略，避免不必要的磁盘操作
```

4. 对现有的数据大表建立索引的时候，采用后台运行方式
在对数据集合建立索引的过程中，数据库会停止该集合的所有读写操作，因此建立索引的数据量大，建立过程慢的情况下，建立采用后台运行方式，避免影响正常的业务流程
```
// 默认 background: false
db.person.ensureIndex({name: "frank", age: -1}, {background: true})
```
