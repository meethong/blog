---
layout: post
title: Keepalived实现高可用
categories: Keepalived
description: Keepalived
keywords: Keepalived
---

Keepalived实现高可用,主备切换,日志配置

## keepalived高可用

需要根据自己的配置进行修改,我这边是自己打的[rpm](./2020-3-31-nginx-keepalived-builds)安装的所以就贴几个配置就ok了

### master

```shell
global_defs {
 router_id SERVER_1 #路由id号，不能重复
}
vrrp_script chk_server1 { 
    script "/chek.sh" #执行脚本
    interval 2 #间隔时间
    weight -40  # #优先级（如果脚本执行结果为0，并且weight配置的值大于0，则权重相应的增加，如果脚本执行结果非0，并且weight配置的值小于0，则优先级相应的减少）
}             
      
 vrrp_instance VI_1 {
   state BACKUP #状态 master/backup
   interface ens33  #网卡
   virtual_router_id 55  #集群id必须一致
   priority 100  #权重
   advert_int 1 #主备通讯时间间隔
   authentication {
     auth_type PASS
     auth_pass 1111 #认证号，集群中要一致
   }
   virtual_ipaddress {
     192.168.0.4/24 #使用的虚拟ip
   }
   track_script { #调用上面定义的检查脚本
     chk_server1 
   }

```

### backup

```
vrrp_instance VI_1 {
   state BACKUP 
   interface ens33
   virtual_router_id 55
   priority 99
   advert_int 1
   authentication {
     auth_type PASS
     auth_pass 1111
   }
   virtual_ipaddress {
     192.168.0.4/24
   }

```

### chek.sh

```shell
#!/bin/sh
A=`ps -C nginx --no-header |wc -l`
if [ $A -eq 0 ]
then
    exit 1
fi
```

## 日志配置

默认日志在**/var/log/messages**里 可读性很低 需要修改

修改**/etc/sysconfig/keepalived**  keepalived启动文件 按自己安装路径来

把**KEEPALIVED_OPTIONS="-D"** 修改为：**KEEPALIVED_OPTIONS="-D -d -S 0"**

![keepalived-log](https://i.opsta.cn/keepalived/keepalived-log.png)

再修改**/etc/rsyslog.conf** 

添加   **local0.*                                                /var/log/keepalived.log**

![image-20200528105508482](https://i.opsta.cn/rsyslog/rsyslog-keepalived-log.png)

目前的日志在:**/var/log/keepalived.log** 

## 切换成功

**master切换**

![76](https://i.opsta.cn/keepalived/keepalived-switch-master.png)

**buckup切换**

![77](https://i.opsta.cn/keepalived/keepalived-switch-buckup.png)

## 参考

[keepalived配置高可用](http://www.linkops.cn/820.htm)