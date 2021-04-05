---
layout: post
title: MariaDB,Galera多主集群安装
categories: MariaDB
description: MariaDB
keywords: MariaDB
---

记录下MariaDB,Galera多主集群的安装与配置

###	环境配置

| 192.168.0.184 | centos-mysql   |
| ------------- | -------------- |
| 192.168.0.186 | centos-master  |
| 192.168.0.187 | centos-minion1 |

三台均用阿里云镜像站,配置一个Mariadb 镜像如下

```
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo

vi /etc/yum.repos.d/MariaDB.repo

[mariadb]
name = MariaDB
baseurl = http://mirrors.ustc.edu.cn/mariadb/yum/10.1/centos7-amd64
gpgkey=http://mirrors.ustc.edu.cn/mariadb/yum/RPM-GPG-KEY-MariaDB
gpgcheck=1
```

### 升级所有包

```
yum upgrade -y #不改变软件设置和系统设置，系统版本升级，内核不改变
```

### 安裝Mariadb

```
yum install -y MariaDB MariaDB-server MariaDB-shared MariaDBb-common galera rsync
```

### 启动服务 修改密码

```
[root@localhost ~]# systemctl start mariadb
[root@localhost ~]# /usr/bin/mysqladmin -u root password 'root'
```

### 关闭防火墙

```
[root@localhost ~]# systemctl stop firewalld.service
[root@localhost ~]# systemctl disable firewalld.service
```

### 配置域名解析

```
vim /etc/hosts
192.168.0.184 centos-mysql
192.168.0.186 centos-master
192.168.0.187 centos-minion1
```

### 加大文件描述符

```
vi /etc/security/limits.conf

soft nofile 65536
hard nofile 65536

vi /etc/sysctl.conf

fs.file-max=655350  
net.ipv4.ip_local_port_range = 1025 65000  
net.ipv4.tcp_tw_recycle = 1 

 sysctl -p
```

### 安装percona-xtrabackup

https://www.percona.com/downloads/XtraBackup/Percona-XtraBackup-2.4.6/binary/tarball/percona-xtrabackup-2.4.6-Linux-x86_64.tar.gz

```
tar -xvf percona-xtrabackup-2.4.6-Linux-x86_64.tar.gz 
[root@centos-mysql opt]#  cd percona-xtrabackup-2.4.6-Linux-x86_64/bin/
[root@centos-mysql bin]#  cp -a * /usr/bin/
```

### 修改server.cnf

```
#节点ip[root@centos-master ~]# cat /etc/my.cnf.d/server.cnf 


[server]

[mysqld]

[galera]
wsrep_on=ON
wsrep_provider=/usr/lib64/galera/libgalera_smm.so  #galera的库文件的地址
wsrep_cluster_address="gcomm://192.168.0.184,192.168.0.186,192.168.0.187"#各节点的ip
wsrep_node_name=centos-master #节点主机名　　
wsrep_node_address=192.168.0.186#节点ip
binlog_format=row #二进制日志设置为行模式  row （安全性最高，性能最低）
default_storage_engine=InnoDB #使用的默认引擎 innoDB  支持事务
innodb_autoinc_lock_mode=2　#2为性能最好  

[embedded]

[mariadb]

[mariadb-10.1]
```

### 启动服务

```
sudo -u mysql /usr/sbin/mysqld --wsrep-new-cluster &> /tmp/wsrep_new_cluster.log &
tail -f /tmp/wsrep_new_cluster.log  # 如果出现 ready for connections, 表示启动成功
#只需要任意一台机子启动 这条命令 
```

其余使用这个

```
systemctl start mariadb 
```

```
在任意数据库服务器执行以下命令验证 MariaDB Galera Cluster
$ mysql -uroot -p -e "show status like 'wsrep_cluster_size'#这里应该显示集群里有3个节点
$ mysql -uroot -p -e "show status like 'wsrep_connected'  #这里应该显示ON
$ mysql -uroot -p -e "show status like 'wsrep_incoming_addresses' #这里应该显示3个ip
$ mysql -uroot -p -e "show status like 'wsrep_local_state_comment'#这里显示节点的同步状态
```

### 注意事项

1. 如果三台服务都关闭了,需要更改配置文件才能起来
2. 会以master关闭前数据为主 再启动的master

```
[root@centos-master ~]# more /var/lib/mysql/grastate.dat

GALERA saved state

version: 2.1
uuid:    9b049efe-682f-11ea-bc1f-96ff892d587d
seqno:   -1
safe_to_bootstrap: 0
#把safe_to_bootstrap改为1 就可以启动了
```

