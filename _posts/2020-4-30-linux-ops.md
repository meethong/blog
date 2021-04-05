---
layout: post
title: Linux小技巧
categories: [Linux,Nginx]
description: Linux Nginx
keywords: Linux
---

分享一下日常用到的命令,经常忘记，记录一下！！！

## linux小技巧

### linux查看某个进程占用的端口

先获取 pid 然后 grep

```
netstat -nltp 

netstat -nltp | grep 10980
```

## Nginx 日志分析

### 根据状态码进行请求次数排序

```
cat access.log | cut -d '"' -f3 | cut -d ' ' -f2 | sort | uniq -c | sort -r
```

```
awk '{print $9}' access.log | sort | uniq -c | sort -r
```

### 找出状态码的请求url

```
awk '($9 ~ /404/)' access.log | awk '{print $7}' | sort | uniq -c | sort -r
```

### 找出请求的ip地址

```
awk -F\" '($2 ~ "/zzz.php"){print $1}' access.log | awk '{print $1}' | sort | uniq -c | sort -r
```

### 找出url有php

```
awk -F\" '($2 ~ "php"){print $2}' access.log | awk '{print $2}' | sort | uniq -c | sort -r
```

### 查看单个ip访问

```javascript
grep ^123.123.123.123 access.log 
```

### 获取pv数

```
$ cat access.log | wc -l
```

### 获取ip数

```
$ cat access.log | awk '{print $1}' | sort -k1 -r | uniq | wc -l
```