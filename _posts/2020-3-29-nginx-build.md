---
layout: post
title: Nginx-1.17.9编译安装
categories: [Nginx,rpmbuild]
description: Nginx rpmbuild
keywords: Nginx,rpmbuild
---

以前对着别人的教程也安装过Nginx,其实都一样无非看自己需要什么依赖，安装在哪，之前的网站也用它做负载，配置证书，一系列的操作，大半年没用忘记了，今天又用到就记录下。

## Nginx安装

用到的两个安装包

[http://nginx.org/download/nginx-1.17.9.tar.gz](http://nginx.org/download/nginx-1.17.9.tar.gz)

[https://bitbucket.org/nginx-goodies/nginx-sticky-module-ng/get/master.tar.gz](https://bitbucket.org/nginx-goodies/nginx-sticky-module-ng/get/master.tar.gz)

下载之后 解压

```
vim nginx-goodies-nginx-sticky-module-ng-08a395c66e42 ngx_http_sticky_misc.c
新增两行
#include <openssl/sha.h>
#include <openssl/md5.h>
```

### 编译命令

```
./configure --prefix=/opt/meethong/nginx --add-module=/nginx-goodies-nginx-sticky-module-ng-08a395c66e42/
make
make install
```

### 启停服务

```
/opt/meethong/nginx/sbin/nginx 
/opt/meethong/nginx/sbin/nginx -s stop
```

### 配置服务

```
/opt/meethong/nginx/conf 配置
```

