---

layout: post
title: Nginx反向代理谷歌镜像
categories: Nginx
description: linux
keywords: linux,Nginx
---

用过某度搜索引擎的都知道，，广告一大堆,而且一搜就是  CSDN 的东西重复的太多。。。 还搜不到啥东西，而 谷歌搜索 应该是对内容进行了过滤之类的

## 准备工作

- 一台能访问 google.com 的 VPS  

  vps没啥要求，128m都能完美运行

- 一个域名

  ​    网上有很多地方可以申请免费的[域名](http://www.freenom.com/)，例如[Freenom](http://www.freenom.com/)，可以免费申请.tk，.ml，.cf，.ga等后缀的顶级域名，一年免费，到期可以免费续期。

- 用到的文件

   [Nginx源码包](http://nginx.org/download/nginx-1.17.9.tar.gz)

  [Openssl源码包](https://www.openssl.org/source/openssl-1.1.1g.tar.gz)

  [substitution模块](https://github.com/yaoweibin/ngx_http_substitutions_filter_module/archive/master.zip)

### 安装

 Nginx 支持反向代理，但是谷歌可能提示 人机验证之类的 

```
yum -y install gcc
yum -y install pcre-devel
yum install -y zlib zlib-devel
```

```
./configure \
--user=nginx                          \
--group=nginx                         \
--prefix=/etc/nginx                   \
--sbin-path=/usr/sbin/nginx           \
--conf-path=/etc/nginx/nginx.conf     \
--pid-path=/var/run/nginx.pid         \
--lock-path=/var/run/nginx.lock       \
--error-log-path=/var/log/nginx/error.log \
--http-log-path=/var/log/nginx/access.log \
--with-http_gzip_static_module        \
--with-http_stub_status_module        \
--with-http_ssl_module                \
--with-pcre                           \
--with-file-aio                       \
--with-http_realip_module             \
--without-http_scgi_module            \
--without-http_uwsgi_module           \
--without-http_fastcgi_module         \
--with-openssl=../openssl-1.1.1g      \
--add-module=../ngx_http_substitutions_filter_module 
```

有的人用ngx_http_google_filter_module 这个模块 但是我这里没有用 有需要 可以自己加上

```
--add-module=../ngx_http_google_filter_module \
```

![nginx编译成功](https://i.opsta.cn/nginx/nginx-proxy-build.webp)

```
make && make install  #安装
```

查看模块是否已经安装上

![查看nginx安装的模块](https://i.opsta.cn/nginx/nginx-proxy-version.webp)

## 配置nginx

http 下定义 google 地址

```
 upstream www.google.com {
    #中国香港 google.com
    server 216.58.221.68:443 weight=6;
    #中国台湾 google.com
    server 74.125.23.99:443 weight=5;
    #日本东京都东京 google.com
    server 172.217.25.68:443 weight=4;
    #日本东京都东京 google.com
    server 216.58.200.196:443 weight=4;
    #日本大阪府大阪 google.com
    server 216.58.197.4:443 weight=3;
    #新加坡 google.com
    server 74.125.130.147:443 weight=2;
    #美国 我的小鸡 ping www.google.com 所得
    server 216.58.217.196:443 weight=1;
    server 172.217.11.164:443 weight=1;
    #美国 google.com
    server 74.125.28.104:443 weight=1;
    #美国 google.com
    server 74.125.28.147:443 weight=1;
    #美国华盛顿州西雅图 google.com
    server 172.217.3.196:443 weight=1;
    }

```

放到  location / {下

```
  proxy_redirect off;
        proxy_cookie_domain google.com <google.domain>;
        proxy_pass https://www.google.com;
        proxy_connect_timeout 60s;
        proxy_read_timeout 5400s;
        proxy_send_timeout 5400s;
        proxy_set_header Host "www.google.com";
        proxy_set_header User-Agent $http_user_agent;
        proxy_set_header Referer https://www.google.com;
        proxy_set_header Accept-Encoding "";
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Real-Port $remote_port;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;
        proxy_set_header Accept-Language "zh-CN";
        proxy_set_header Cookie "PREF=ID=047808f19f6de346:U=0f62f33dd8549d11:FF=2:LD=en-US:NW=1:TM=1325338577:LM=1332142444:GM=1:SG=2:S=rE0SyJh2W1IQ-Maw";
```

## 自定义页面

```
subs_filter_types *;
```

### 插入google分析代码
```
subs_filter '<head>' '<head><!--Global site tag(gtag.js)-Google Analytics--><script async src="https://www.googletagmanager.com/gtag/js?id=UA-148080910-5"></script><script>window.dataLayer=window.dataLayer||[];function gtag(){dataLayer.push(arguments)}gtag("js",new Date());gtag("config","UA-148080910-5");</script>';
```

### 替换域名为镜像域名
```
subs_filter "www.google.com" "a.sudo.gq";
```

## 参考

[nginx代理搭建google镜像站](http://blog.niostack.com/posts/b3c3715a/)