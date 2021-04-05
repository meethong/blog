---
layout: post
title: Nginx+Keepalived打成rpm安装包
categories: [Nginx,Keepalived,rpmbuild]
description: Nginx Keepalived rpmbuild
keywords: Nginx
---

前两天一直没更新今天更新下,将keepalived跟nginx打成rpm安装包,打包的时候纠结是否要在打包的时候编译还是怎么搞,最后直接把文件打个包，发现能用哈哈哈,属实整笑了！

## Nginx+Keepalive打成rpm安装包

之前已经安装编译好nginx跟keepalived了,所以直接在将两个文件夹打包,编写下启动服务及Spec文件打包即可。

### 配置讲解

#### meethong.spec

```
Name: meethong 
Version:1.0 
Release: 1
Summary: proxy 
Group: Applications/Engineering 
License: GPL
AutoReqProv:no
Source0:demo.tar.gz #压缩的文件夹包
Source1:nginx.service #nginx启停的服务脚本
Source2:nginx.conf #nginx的配置文件
Source3:keepalived.conf #keepalived的配置文件
Source4:keepalived.service  #keepalived启停的服务脚本
BuildRoot: %{_tmppath}/%{name}-%{version}-%{release}-root
%description
This package is  proxyserver Setup Program.
Prefix: /opt #安装路径
%define userpath /opt
%define ppath /opt
%define nginxpath /opt/meethong/nginx/conf
%define keppath /etc/keepalived
%define initd /usr/lib/systemd/system
#%define 定义文件路径
%prep
%setup  -b 0 -c -n meethong #将第一个文件 解压成meethong
%install 安装
install -d $RPM_BUILD_ROOT%{ppath}
install -d $RPM_BUILD_ROOT%{initd}

cp -a $RPM_BUILD_DIR/%{name}* $RPM_BUILD_ROOT%{ppath} 
%{__install} -p -D -m 0755 %{SOURCE1} $RPM_BUILD_ROOT%{initd}/nginx.service 
#安装并赋予权限
%{__install} -p -D -m 0755 %{SOURCE2} $RPM_BUILD_ROOT%{nginxpath}/nginx.conf
%{__install} -p -D -m 0755 %{SOURCE1} $RPM_BUILD_ROOT%{initd}/nginx.service
%{__install} -p -D -m 0644 %{SOURCE3} $RPM_BUILD_ROOT%{keppath}/keepalived.conf
%{__install} -p -D -m 0755 %{SOURCE4} $RPM_BUILD_ROOT%{initd}/keepalived.service
%pre
mkdir -p /etc/keepalived  
#安装前
%post
systemctl  enable keepalived
systemctl  enable nginx
#安装后
%preun
systemctl stop keepalived  
systemctl stop nginx
systemctl disable keepalived
systemctl disable nginx
rm -rf /opt/meethong/nginx
rm -rf /opt/meethong/keepalived
rm -rf /etc/keepalived
#卸载
%files  #用到的文件
%{ppath}
%{nginxpath}
%{initd}
%{keppath}
%clean
rm -rf /opt/meethong/nginx
rm -rf /opt/meethong/keepalived
rm -rf /etc/keepalived
%changelog
```

#### nginx.service

```
[Unit]
Description=nginx
After=network-online.target syslog.target 
Wants=network-online.target 

[Service]
Type=forking
KillMode=process
ExecStart=/opt/meethong/nginx/sbin/nginx -c /opt/meethong/nginx/conf/nginx.conf
ExecReload=killall  nginx

[Install]
WantedBy=multi-user.target
```

#### keepalived.service

```
[Unit]
Description=LVS and VRRP High Availability Monitor
After=network-online.target syslog.target 
Wants=network-online.target 

[Service]
Type=forking
PIDFile=/run/keepalived.pid
KillMode=process
EnvironmentFile=-/opt/meethong/keepalived/etc/sysconfig/keepalived
ExecStart=/opt/meethong/keepalived/sbin/keepalived $KEEPALIVED_OPTIONS
ExecReload=/bin/kill -HUP $MAINPID

[Install]
WantedBy=multi-user.target
```



