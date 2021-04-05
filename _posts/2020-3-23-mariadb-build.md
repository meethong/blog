---
layout: post
title: MariaDB Galera编译安装
categories: MariaDB
description: MariaDB,Galera
keywords: MariaDB
---

编译安装mariadb+gelera,部署仲裁节点,之前弄过一次[yum安装的MariaDB+Galera](https://meethong.github.io/2020/03/18/mariadb-cluster/ ),其实编译也很简单,增加一些参数就行,工作中需要折腾了我两三天吧估计，编译命令整错了导致没编译上，记录一下。

## 编译MariaDB

1. 编译mariadb需要用到cmake,gcc,c++之类的环境

   ```
   yum install -y cmake ncurses-devel gcc-c++
   ```

2. 下载MariaDB源码包及gelera

   MariaDB:https://downloads.mariadb.com/MariaDB/mariadb-10.1.6/source/mariadb-10.1.6.tar.gz

   gelera:http://releases.galeracluster.com/galera-3/centos/7/x86_64/galera-3-25.3.29-1.el7.x86_64.rpm

3. 直接解压并且编译

   ```
   tar -xvf mariadb-10.1.6.tar.gz
   ```

   编译命令

   ```
   cmake \
   -DCMAKE_INSTALL_PREFIX=/opt/meethong/mariadb \
   -DDEFAULT_SYSCONFDIR=/opt/meethong/mariadb \
   -DMYSQL_DATADIR=/opt/meethong/mariadb/data \
   -DMYSQL_TCP_PORT=8102 \
   -DMYSQL_UNIX_ADDR=/opt/meethong/mariadb/data/mysql.sock \
   -DEXTRA_CHARSETS=all \
   -DDEFAULT_CHARSET=utf8 \
   -DDEFAULT_COLLATION=utf8_general_ci \
   -DWITH_READLINE=1 \
   -DWITH_MYISAM_STORAGE_ENGINE=1 \
   -DWITH_INNOBASE_STORAGE_ENGINE=1 \
   -DWITH_MEMORY_STORAGE_ENGINE=1 \
   -DENABLED_LOCAL_INFILE=1 \
   -DWITH_SSL=system \
   -DWITH_ZLIB=system \
   -DWITH_LIBWRAP=0 \
   -DWITH_WSREP=ON \
   -DWITH_INNODB_DISALLOW_WRITES=ON \
   --------
   执行完camke 执行make
   make & make install
   ```

## 配置集群

**安装galera-3-25.3.29-1.el7.x86_64**

```
rpm  -ivh galera-3-25.3.29-1.el7.x86_64.rpm --nodeps --force
[root@demo ~]# rpm -qpl galera-3-25.3.29-1.el7.x86_64.rpm ##查看这个包的安装路径
警告：galera-3-25.3.29-1.el7.x86_64.rpm: 头V4 RSA/SHA512 Signature, 密钥 ID bc19ddba: NOKEY
/etc/sysconfig/garb
/usr/bin/garb-systemd
/usr/bin/garbd
/usr/lib/systemd/system/garb.service
/usr/lib64/galera-3
/usr/lib64/galera-3/libgalera_smm.so
/usr/share/doc/galera-3
/usr/share/doc/galera-3/COPYING
/usr/share/doc/galera-3/LICENSE.asio
/usr/share/doc/galera-3/LICENSE.chromium
/usr/share/doc/galera-3/LICENSE.crc32c
/usr/share/doc/galera-3/README
/usr/share/doc/galera-3/README-MySQL
/usr/share/man/man8/garbd.8.gz
[root@demo ~]# 
```

### 配置my.cnf

```
vim /opt/meethong/mariadb/my.cnf #拉到最后把这段复制进去
[galera]
wsrep_on=ON
wsrep_provider=/usr/lib64/galera-3/libgalera_smm.so#galera的库文件的地址
wsrep_cluster_address="gcomm://192.168.0.190,192.168.0.189"#各节点的ip
wsrep_node_name=demo#节点主机名
wsrep_cluster_name=galera_cluster
wsrep_node_address=192.168.0.189#节点ip
binlog_format=row#二进制日志设置为行模式  row （安全性最高，性能最低）
default_storage_engine=InnoDB#使用的默认引擎 innoDB  支持事务
innodb_autoinc_lock_mode=2#2为性能最好

配置下log日志文件地址
[mysqld]
log-error       =/opt/eetrust/mmeethong/data/mariadb.log
```

![](https://i.opsta.cn/img/mariadb-build-my-cnf.png)

```
第一台用这个命令启动
service mysqld  bootstrap
有错误就差看下日志,其他节点启动用这个
service mysqld  start
```

## 仲裁节点配置

我的测试环境为两台mariadb 一台仲裁服务 仲裁并没有安装MariaDB

安装galera-3-25.3.29-1.el7.x86_64 他的配置文件是**/etc/sysconfig/garb**

```
[root@localhost ~]# more /etc/sysconfig/garb
# Copyright (C) 2012 Codership Oy
# This config file is to be sourced by garb service script.

# A comma-separated list of node addresses (address[:port]) in the cluster
# GALERA_NODES=""

# Galera cluster name, should be the same as on the rest of the nodes.
# GALERA_GROUP=""

# Optional Galera internal options string (e.g. SSL settings)
# see http://galeracluster.com/documentation-webpages/galeraparameters.html
# GALERA_OPTIONS=""

# Log file for garbd. Optional, by default logs to syslog
# LOG_FILE=""

GALERA_NODES="192.168.0.189:4567,192.168.0.190:4567" # 这里是两节点的地址
GALERA_GROUP="galera_cluster"  # 这里的group名称保持与两节点的wsrep_cluster_name属性一致
LOG_FILE="/var/log/garb.log"
```

### 启停命令

```
[root@localhost ~]# systemctl stop garb 
[root@localhost ~]# systemctl start garb
```

### 查看集群节点

```
mysql> show status like 'wsrep_cluster_size';
+--------------------+-------+
| Variable_name      | Value |
+--------------------+-------+
| wsrep_cluster_size | 3     |
+--------------------+-------+
1 row in set
```

# 参考

1. [https://www.dwhd.org/20150924_133631.html](https://www.dwhd.org/20150924_133631.html)
2. [https://jeremy-xu.oschina.io/2018/02/mariadb-galera-cluster%E9%83%A8%E7%BD%B2%E5%AE%9E%E6%88%98/#mariadb-galera-cluster%E7%9A%84%E8%87%AA%E5%90%AF%E5%8A%A8](https://jeremy-xu.oschina.io/2018/02/mariadb-galera-cluster部署实战/#mariadb-galera-cluster的自启动)
3. [https://blog.51cto.com/1130739/1852319](https://blog.51cto.com/1130739/1852319)

