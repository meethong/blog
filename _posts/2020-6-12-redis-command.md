---
layout: post
title: Redis常用命令
categories: [Redis]
description: Redis
keywords: Redis
---



## redis

```
flushdb //删除当前数据库中的所有Key

flushall //删除所有数据库中的key

keys * //获取所有key

keys admin* //列出开头的

dbsize  //查看key数量

monitor //查看redis正在做什么
```



