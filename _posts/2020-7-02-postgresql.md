---
layout: post
title: Postgresql笔记
categories: [Postgresql]
description: Postgresql
keywords: Postgresql
---

Postgresql使用笔记

# Postgresql

Postgresql配置日志 配置文件路径 在安装路径下的**Postgresql.conf**

> 日志设置

```shell
#设置日志路径
log_directory = '/opt/ops/logs'      

#日志的格式
log_destination = 'csvlog,stderr'         # stderr, csvlog, syslog, and eventlog,
                                      
#日志的类型                                     
log_statement = 'all'                   # none, ddl, mod, all

logging_collector = on  
#日志文件名的格式
log_filename = '%Y-%m-%d.log' 
#文件默认的权限是0600
log_file_mode = 0600
#每个级别都包括以后的所有级别。级别越靠后，被发送的消息越少
log_min_messages = debug1  # values in order of decreasing detail:
                                        #   debug5
                                        #   debug4
                                        #   debug3
                                        #   debug2
                                        #   debug1
                                        #   info
                                        #   notice
                                        #   warning
                                        #   error
                                        #   log
                                        #   fatal
                                        #   panic (effectively off)

log_min_error_statement = debug1
```





