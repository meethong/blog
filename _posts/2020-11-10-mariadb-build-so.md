---
layout: post
title: MariaDB源码编译安装指定so库文件，加载路径
categories: MariaDB
description: MariaDB
keywords: MariaDB
---

**MariaDB**默认情况下会使用当前系统的so库文件依赖，由于系统不一致，有可能会导致编译的安装包无法复用,在**CPU架构**一致的情况下，启动**MariaDB**,提示依赖的**so文件**,不存在，导致 **MariaDB**启动不了

## 解决办法

修改**MariaDB** 编译文件,加入以下函数，设置**MariaDB**加载的路径，然后重新编译即可

```shell
SET(CMAKE_BUILD_WITH_INSTALL_RPATH TRUE)
SET(CMAKE_INSTALL_RPATH "\${ORIGIN}/../lib-ext")
```

![mariadb-20201111162401394](https://i.opsta.cn/mariadb/mariadb-20201111162401394.png)

默认情况下，**MariaDB** 加载的是系统的**so**

![Mariadb_2020-11-10_14-15-55](https://i.opsta.cn/mariadb/Mariadb_2020-11-10_14-15-55.png)

当**lib-ext**目录下放置**so**文件之后，他会去**lib-ext**目录下加载相关**so**，这样就有效的解决了缺**so**的问题了

![Mariadb_2020-11-10_14-15-55](https://i.opsta.cn/mariadb/Mariadb_2020-11-10_14-15-56.png)

**rpmbuild** 打包过程中可能会出现检查**so**路径的问题，将校验的配置注释即可

![mariadb_2020-11-11_16-36-50](https://i.opsta.cn/mariadb/mariadb_2020-11-11_16-36-50.png)