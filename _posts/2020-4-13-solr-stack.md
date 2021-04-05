---
layout: post
title: Solr指定的堆栈大小报错
categories: Solr
description: Solr
keywords: Solr
---

启动报错提示指定的堆栈太小，请至少指定 328 k

## Solr报错

运行`./solr start -force -f `  提示相关内容

```
The stack size specified is too small, Specify at least 328k
Error: Could not create the Java Virtual Machine.
Error: A fatal exception has occurred. Program will exit.
```

这个问题查找相关配置烦了我挺久，第一次使用不太清楚，随即Google 找到个类似报错设置java内存大小导致solr启动不了，然后查找配置看到了配置256k的地方 我寻思这个太小设置为1024就启动成功了

```
vim solr/bin/solr
```

![solr文件](https://i.opsta.cn/img/20200414101330.png)

修改256为1024即可 再启动

![solr启动](https://i.opsta.cn/img/solr-stack-start.png)