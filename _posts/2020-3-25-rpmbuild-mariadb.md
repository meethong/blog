---
layout: post
title: rpmbuild打包mariadb集群
categories: [MariaDB,rpmbuild]
description: MariaDB rpmbuild
keywords: MariaDB,rpmbuild
---

上两天在研究如何编译安装mariadb+gelera就是为了把它打成rpm包，其实打成rpm包很简单解压一下就行了,把mariadb做成服务,主要是为了方便安装，实现一条命令安装完成mariadb+gelera集群

## 打包前准备

把so放到mariadb目录下，修改my.cnf配置的路径

![](https://i.opsta.cn/img/rpmbuild-mariadb-my.cnf.png)

编写master启动脚本

```
[root@demo2 mariadb]# more mysqlCluster-start.sh 
sed -i 's/0/1/g' /opt/**隐藏路径**/mariadb/data/grastate.dat
pkill mysqld
netstat -nlp | grep :4567 | awk '{print $7}' | awk -F"/" '{ print $1 }' #根据端口关闭进程
/opt/**隐藏路径**/mariadb/bin/mysqld --wsrep-new-cluster --user=mysql
```

## 安装rpmbuild

```
yum install rpm-build
yum install rpmdevtools 
rpmdev-setuptree #生成rpmbuild目录 

[root@demo rpmbuild]# ls
BUILD  RPMS  SOURCES  SPECS  SRPMS
[root@demo rpmbuild]# pwd
/root/rpmbuild
```

## 文件解析

```
[root@demo rpmbuild]# tree *
BUILD
RPMS
SOURCES
├── mariadb-cluster.tar.gz #打包的mariadb安装路径的文件夹
└── mariadb.service #编写的mariadb服务名
SPECS
└── mariadb.spec
SRPMS
0 directories, 3 files
```

```
[root@demo SOURCES]# more mariadb.service 
[Unit]
Description=mysqld Server

[Service]
Type=forking
ExecStart=/opt/**隐藏路径**/mariadb/mysqld start
ExecStop=/opt/**隐藏路径**/mariadb/mysqld stop
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```



```
[root@demo SPECS]# more mariadb.spec 
Name: Mariadb-Cluster
Version:1
Release: 1
Summary: Mariadb-Cluster1.0
Group: Applications/Engineering
License: GPL
AutoReqProv:no
Source0:mariadb-cluster.tar.gz
Source1:mariadb.service
BuildRoot: %{_tmppath}/%{name}-%{version}-%{release}-root
%description
This package is mariadb Setup Program.
Prefix: /opt/**隐藏路径**
%define initd /usr/lib/systemd/system
%define ppath /opt/**隐藏路径**
%prep
%setup  -b 0 -c -n mariadb


#%build

%install
install -d $RPM_BUILD_ROOT%{initd}
install -d $RPM_BUILD_ROOT%{ppath}
cp -a $RPM_BUILD_DIR/*  $RPM_BUILD_ROOT%{ppath}
%{__install} -p -D -m 0755 %{SOURCE1} $RPM_BUILD_ROOT%{initd}/mariadb.service

%pre
userdel -r mysql
groupadd mysql
useradd -g mysql mysql
%post
systemctl  enable mariadb.service
%preun
systemctl  stop   mariadb.service
systemctl disable mariadb.service
rm -rf /opt/**隐藏路径**/mariadb
%files
%defattr(-,root,root)
%attr(755,mysql,mysql) /opt/**隐藏路径**/mariadb/*
%{initd}

%clean
rm -rf /opt/**隐藏路径**/mariadb

%changelog
```

### 编译rpm包

```
rpmbuild  -bb mariadb.spec 

[root@demo rpmbuild]# tree 
.
├── BUILD
├── BUILDROOT
├── RPMS
│   └── x86_64
│       └── Mariadb-Cluster-1-1.x86_64.rpm
├── SOURCES
│   ├── mariadb-cluster.tar.gz
│   └── mariadb.service
├── SPECS
│   ├── err.log
│   └── mariadb.spec
└── SRPMS

7 directories, 5 files


```

### 安装运行

需要修改my.cnf配置文件

```
[galera]
wsrep_on=ON
wsrep_provider=/opt/eetrust/mariadb/libgalera_smm.so#galera的库文件的地址
wsrep_cluster_address="gcomm://192.168.0.190,192.168.0.189"#各节点的ip
wsrep_node_name=demo#节点主机名
wsrep_cluster_name=galera_cluster
wsrep_node_address=192.168.0.189#节点ip
binlog_format=row#二进制日志设置为行模式  row （安全性最高，性能最低）
default_storage_engine=InnoDB#使用的默认引擎 innoDB  支持事务
innodb_autoinc_lock_mode=2#2为性能最好
```

修改内容为正确的主机名,IP即可

如果将它作为**master**启动

```
[root@demo mariadb]# ./mysqlCluster-start.sh 
Shutting down MySQL.... SUCCESS! 
Bootstrapping the cluster.. Starting MySQL.. SUCCESS! 
```

从节点启动

```
systemctl start mariadb
```

