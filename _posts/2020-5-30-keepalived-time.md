---
layout: post
title: Keepalived实现定时任务
categories: [Keepalived,Shell]
description: Keepalived Shell
keywords: Keepalived,Shell
---

keepalived实现定时任务,其实crontab更好用。。。但是有的魔改linux无法使用crontab,就很无奈可能这就是**麒麟系统吧！

## 实现思路

要定时去执行,不能用**contab**，那就只能用**keepalived**的检查服务 多少秒执行一次脚本，通过脚本去判断，如果当前时间等于这个时间那就执行

### keeplived配置

**keepalived**主要是执行检查脚本,然后如果降权就跳备机 跳备机同时执行恢复数据库的操作

```shell

vrrp_script chk_server1 { 
    script "/status.sh" #执行脚本
    interval 2 #间隔时间
    weight -40  # #优先级（如果脚本执行结果为0，并且weight配置的值大于0，则权重相应的增加，如果脚本执行结果非0，并且weight配置的值小于0，则优先级相应的减少）
}  
track_script { #调用上面定义的检查脚本
     chk_server1 
   }
     notify_backup "/etc/keepalived/regain.sh" #状态为backup执行脚本
}
```

### 检测脚本

获取当前时间，先删除三个月以前的备份数据库,如果为晚上2点就执行备份，并且检测该文件存不存在如果存在就执行备份存在就不备份,当不是2点时就检测 **Mysql,Ngiinx,Tomcat**服务down就写info里面记录,成功就清空该文件,统计超过三次就给keepalived降权，然后就跳备机。

第一次写**shell**，总感觉逻辑不是很通顺。。待改进吧，纯粹查命令写出来，

```shell
#!/bin/sh
hours=`date +"%H%M"`
static=200
database=/opt/keepalived/info/database`date +"%Y-%m-%d"`.sql
status=/opt/keepalived/info/
find $status -mtime +91 -name "*.sql" -exec rm -rf {} \;
if [ $hours -eq $static ];
 then
   if [ ! -f "$database" ];
   then
   /opt/mariadb/bin/mysqldump -uroot -ppassword database > $database
   echo "导出成功!";
   else
   echo "文件已存在!";
   fi
else
   echo "开始服务检查";
   `exec >/dev/tcp/127.0.0.1/80`
    if [ $? -ne 0 ];
    then
    echo "nginx is down";
    `systemctl start nginx`
      [ $? -ne 0 ];
      echo "nginx 无法启动";
  
        echo 1 >> $status/nginx.info
    else
        echo "nginx 服务正常";
        
        echo  > $status/nginx.info
      fi
       `exec >/dev/tcp/127.0.0.1/8080`
          if [ $? -ne 0 ];
          then
             echo "tomcat is down";
             `systemctl start tomcat`
             [ $? -ne 0 ];
             echo "Tomcat 无法启动";
          
               echo 2 >> $status/Tomcat.info
             else
             echo "tomcat 服务正常";
             echo  > $status/tomcat.info
             fi 
               `exec >/dev/tcp/127.0.0.1/3306`
                 if [ $? -ne 0 ];
                 then 
                 echo "mysql is down";
                 `systemctl start mysql`
                 [ $? -gt 0 ];
                  echo "mysql 无法启动";
                   echo 3 >> $status/mysql.info
                 else
                 echo "mysql 服务正常";
              echo  > $status/mysql.info
          fi
        if [ `cat $status*.info|wc -w`  -ge 3 ]
        then
        exit 1
	else 
        exit 0
       fi
     fi

```

### 恢复脚本

最好记录下日志说明什么时候恢复过。

```shell
#!/bin/sh
database=/opt/keepalived/info/database`date +"%Y-%m-%d"`.sql
time=`date "+%Y_%m_%d %H:%M:%S"`
update=/opt/keepalived/info/update.txt
/opt/mariadb/bin/mysql -hIP -uroot -ppassword database  < $database
 if   [ $? -ne 0 ];
    then
    echo "$time数据库以$database恢复失败" >> $update
    else
    echo "$time数据库以$database恢复成功" >> $update
fi
```

