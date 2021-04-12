---
layout: post
title: windows 10部署使用Docker
categories: [Docker]
description: Docker
keywords: Docker
---

Windows 去安装 Docker是基于 Hyper-V的，所以必须**启用Hyper-V Windows功能** 官网的安装包也会去开启相关功能。

# Docker

[Docker安装文档](https://docs.docker.com/compose/install/)

[Docker安装包](https://download.docker.com/win/stable/Docker%20Desktop%20Installer.exe)

## docker-compose安装

要用到PowerShell 最好设置代理 太慢了

## PowerShell设置代理

- 设置代理

```
netsh winhttp set proxy "127.0.0.1:1080" 
```

- 恢复默认

```
netsh winhttp reset proxy
```

注意要在管理员模式运行，不然提示权限不足。

```shell
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12 #支持https
Invoke-WebRequest "https://github.com/docker/compose/releases/download/1.26.1/docker-compose-Windows-x86_64.exe" -UseBasicParsing -OutFile $Env:ProgramFiles\Docker\docker-compose.exe  #安装docker-compose
```

## 修改docker默认存储路径

windows10的docker使用的是Hyper-V虚拟机，所以镜像存放的目录就是Hyper-V的目录，首先停止docker。

打开Hyper-V管理器，1.开始菜单右键->控制面板->管理工具->Hyper-V 管理器

右键选择Hyper-V设置

![docker_2020-07-04_20-38-13](https://i.opsta.cn/docker/docker_2020-07-04_20-38-13.png)

然后docker跟他设置一样重启就行

![docker_2020-07-04_20-39-30](https://i.opsta.cn/docker/docker_2020-07-04_20-39-30.png)

## 报错

Docker.Core.Backend.BackendException:
Error response from daemon: open \\.\pipe\docker_engine_linux: The system cannot find the file specified.

在win10 命令行提示符执行：

```shell
Net stop com.docker.service

Net start com.docker.service
```

## 为容器设置代理

让容器访问谷歌 开心的冲浪 

修改 **C:\Users\用户名\.docker** 下的 **config.json**  可以把下面这段复制进去 然后重启 **docker**

```shell
{
  "proxies": {
    "default": {
      "httpProxy": "192.168.24.149:7890", #代理地址
      "httpsProxy": "192.168.24.149:7890",#代理地址
      "noProxy": "127.0.0.1,localhost,172.17.0.*,192.168.24.*" #不代理请求
    }
  }
}
```

