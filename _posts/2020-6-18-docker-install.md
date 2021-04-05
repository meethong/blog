---
layout: post
title: Docker笔记第一天
categories: [Docker]
description: Docker
keywords: Docker
---

最新的 Centos7 安装 Docker 19.03.11,本文基于Centos7安装测试，务必保持环境一致

# 安装Docker

### 环境查看

```shell
# 1.查看内核
[root@localhost yum.repos.d]# uname -r
3.10.0-693.el7.x86_64
# 2.查看系统版本
[root@localhost yum.repos.d]# cat /etc/os-release 
NAME="CentOS Linux"
VERSION="7 (Core)"
ID="centos"
ID_LIKE="rhel fedora"
VERSION_ID="7"
PRETTY_NAME="CentOS Linux 7 (Core)"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:centos:centos:7"
HOME_URL="https://www.centos.org/"
BUG_REPORT_URL="https://bugs.centos.org/"

CENTOS_MANTISBT_PROJECT="CentOS-7"
CENTOS_MANTISBT_PROJECT_VERSION="7"
REDHAT_SUPPORT_PRODUCT="centos"
REDHAT_SUPPORT_PRODUCT_VERSION="7"

```

### 安装

```shell
# 1.卸载旧的版本
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
# 2.需要的安装包
yum install -y yum-utils
# 3.设置阿里云安装镜像
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo 
# 4.安装docker
yum install docker-ce docker-ce-cli containerd.io
# 5.启动docker
systemctl start docker
# 6.查看docker版本
[root@localhost yum.repos.d]# docker version
Client: Docker Engine - Community
 Version:           19.03.11
 API version:       1.40
 Go version:        go1.13.10
 Git commit:        42e35e61f3
 Built:             Mon Jun  1 09:13:48 2020
 OS/Arch:           linux/amd64
 Experimental:      false

Server: Docker Engine - Community
 Engine:
  Version:          19.03.11
  API version:      1.40 (minimum version 1.12)
  Go version:       go1.13.10
  Git commit:       42e35e61f3
  Built:            Mon Jun  1 09:12:26 2020
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.2.13
  GitCommit:        7ad184331fa3e55e52b890ea95e65ba581ae3429
 runc:
  Version:          1.0.0-rc10
  GitCommit:        dc9208a3303feef5b3839f4323d9beb36df0a9dd
 docker-init:
  Version:          0.18.0
  GitCommit:        fec3683

```

### 镜像加速

```shell
vim /etc/docker/daemon.json #修改配置文件
[root@localhost ~]# more /etc/docker/daemon.json
{
    "registry-mirrors": [
        "https://docker.mirrors.ustc.edu.cn",
        "https://registry.docker-cn.com"
    ]
}

#重启服务生效
systemctl daemon-reload 
systemctl restart docker
```

```shell

# 7.hello world
docker run hello-world
```

![docker-helloworld](https://i.opsta.cn/docker/docker-helloworld.png)

```shell
# 8 查看docker镜像
[root@localhost yum.repos.d]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         latest              bf756fb1ae65        5 months ago        13.3kB
```

### 卸载

```shell
# 1.卸载依赖
yum remove docker-ce docker-ce-cli containerd.io
# 2.删除资源
rm -rf /var/lib/docker
```

### 常用命令

```
docker version #显示docker的版本信息
docker info #显示docker的系统信息 包括镜像和容器的数量
docker 命令 --help #帮助命令
```

[docker官方命令参考手册](https://docs.docker.com/engine/reference/commandline/cli/)

## 镜像命令

```shell
[root@localhost yum.repos.d]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         latest              bf756fb1ae65        5 months ago        13.3kB

# 解释
REPOSITORY   镜像的仓库
TAG          镜像的标签
IMAGE ID     镜像的ID
CREATED 	 镜像的创建时间
SIZE  		 镜像的大小
# 可选项
-a, --all   #列出所有的镜像
-q, --quiet #只列出镜像的ID

```

### docker search 搜索镜像

```shell
[root@localhost yum.repos.d]# docker search java
NAME                                     DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
node                                     Node.js is a JavaScript-based platform for s…   8940                [OK]                
tomcat                                   Apache Tomcat is an open source implementati…   2756                [OK]                

#可选项 通过搜藏来过滤
--filter=STARS=3000 #搜索出来的镜像STARS大于3000的
[root@localhost /]# docker search mysql --filter=STARS=3000
NAME                DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
mysql               MySQL is a widely used, open-source relation…   9645                [OK]                
mariadb             MariaDB is a community-developed fork of MyS…   3506                [OK]      
```

### docker pull 下载镜像

```shell
[root@localhost /]# docker pull mysql
Using default tag: latest #如果不写tag 默认就是latest
latest: Pulling from library/mysql
8559a31e96f4: Pull complete #分层下载 docker image的核心 联合文件系统
d51ce1c2e575: Pull complete 
c2344adc4858: Pull complete 
fcf3ceff18fc: Pull complete 
16da0c38dc5b: Pull complete 
b905d1797e97: Pull complete 
4b50d1c6b05c: Pull complete 
c75914a65ca2: Pull complete 
1ae8042bdd09: Pull complete 
453ac13c00a3: Pull complete 
9e680cd72f08: Pull complete 
a6b5dc864b6c: Pull complete 
Digest: sha256:8b7b328a7ff6de46ef96bcf83af048cb00a1c86282bfca0cb119c84568b4caf6 #签名
Status: Downloaded newer image for mysql:latest
docker.io/library/mysql:latest #真实地址

#指定版本下载
[root@localhost /]# docker pull mysql:5.7
5.7: Pulling from library/mysql
5.7: Pulling from library/mysql
8559a31e96f4: Already exists 
d51ce1c2e575: Already exists 
c2344adc4858: Already exists 
fcf3ceff18fc: Already exists 
16da0c38dc5b: Already exists 
b905d1797e97: Already exists 
4b50d1c6b05c: Already exists 
d85174a87144: Pull complete 
a4ad33703fa8: Pull complete 
f7a5433ce20d: Pull complete 
3dcd2a278b4a: Pull complete 
Digest: sha256:32f9d9a069f7a735e28fd44ea944d53c61f990ba71460c5c183e610854ca4854
Status: Downloaded newer image for mysql:5.7
docker.io/library/mysql:5.7

```

![docker-images-mysql](https://i.opsta.cn/docker/docker-images-mysql.png)

### docker rmi 删除镜像

```shell
[root@localhost /]# docker rmi -f 9cfcce23593a #可以写容器id和容器name
[root@localhost /]# docker rmi -f 容器id 容器id 容器id #删除多个容器
[root@localhost /]# docker rmi -f $(docker images -aq) #删除全部的容器

```

## 容器命令

### 新建容器并启动

```shell
[root@localhost /]# docker pull centos

docker run [可选参数] image
# 参数说明
--name="Name" 容器名字 tomcat1 tomcat2,用来区分容器
-d 			  后台方式运行
-it 		  使用交互方式运行 进入容器查看内容
-p
	-p ip:主机端口:容器端口
	-p 主机端口:容器端口 (常用)
    -p 容器端口
    容器端口
-p  随机指定端口

#测试
[root@localhost /]# docker run -it centos /bin/bash
[root@9a2366039d94 /]# ls #查看容器内的centos
bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

[root@9a2366039d94 /]# exit #从容器退出
exit
[root@localhost /]# Ctrl+p+q #容器不停止退出

```

### 列出所有的运行的容器

```shell
#docker ps 
		#列出当前正在运行的容器
-a      #列出当前正在运行的容器+历史运行过的容器
-n=?    #显示最近创建过的容器
-q      #只显示容器的编号
		
[root@localhost /]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[root@localhost /]# docker ps  -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                          PORTS               NAMES
9a2366039d94        centos              "/bin/bash"         5 minutes ago       Exited (0) About a minute ago                       brave_ellis
3ba0098333eb        bf756fb1ae65        "/hello"            2 hours ago         Exited (0) 2 hours ago                              boring_villani
```

### 删除容器

```shell
docker rm 容器id    #删除指定的容器
docker rm -f  $(docker images -aq) #删除所有容器
docker ps -a -q|xargs docker rm    #删除所有容器
```

### 启动和停止容器

```shell
docker start 容器id #启动容器
docker restart容器id #重启容器
docker stop 容器id #停止容器
docker kill 容器id #停止容器
```

## 常用命令

### 后台启动容器

```shell
# 命令 docker run -d 镜像名!
[root@localhost ~]# docker run -d centos
1f8d666a521722368cda35dbe7f56f3b46216ea32bd59819a03affa445357269
[root@localhost ~]# docker ps 
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[root@localhost ~]# 
#问题 docker ps  发现centos停止了
#docker容器使用后台运行 容器中必须要有一个前台进程,如果docker发现没有，就会自动停止

```

### 查看日志

```shell
# 显示日志
-tf  		#显示日志
--tail   	#显示日志条数
[root@localhost ~]# docker logs -tf --tail 10 0ef56c3b0827
```

### 查看容器中进程信息

```shell
# docker top 容器id
[root@localhost ~]# docker top 718891259ec5 
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                44346               44327               0                   20:37               pts/0               00:00:00            /bin/bash

```

### 查看镜像的元数据

```shell
# docker inspect  容器id
[root@localhost ~]# docker inspect 718891259ec5 
[
    {
        "Id": "718891259ec5623c1a0969e160433fef56f54b53a78eab71954dd37ff4a682bb",
        "Created": "2020-06-19T12:37:59.121676336Z",
        "Path": "/bin/bash",
        "Args": [],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 44346,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2020-06-19T12:38:00.670251197Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:831691599b88ad6cc2a4abbd0e89661a121aff14cfa289ad840fd3946f274f1f",
        "ResolvConfPath": "/var/lib/docker/containers/718891259ec5623c1a0969e160433fef56f54b53a78eab71954dd37ff4a682bb/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/718891259ec5623c1a0969e160433fef56f54b53a78eab71954dd37ff4a682bb/hostname",
        "HostsPath": "/var/lib/docker/containers/718891259ec5623c1a0969e160433fef56f54b53a78eab71954dd37ff4a682bb/hosts",
        "LogPath": "/var/lib/docker/containers/718891259ec5623c1a0969e160433fef56f54b53a78eab71954dd37ff4a682bb/718891259ec5623c1a0969e160433fef56f54b53a78eab71954dd37ff4a682bb-json.log",
        "Name": "/ecstatic_shaw",
        "RestartCount": 0,
        "Driver": "overlay2",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": null,
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "default",
            "PortBindings": {},
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "CapAdd": null,
            "CapDrop": null,
            "Capabilities": null,
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "private",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "ConsoleSize": [
                0,
                0
            ],
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": [],
            "BlkioDeviceReadBps": null,
            "BlkioDeviceWriteBps": null,
            "BlkioDeviceReadIOps": null,
            "BlkioDeviceWriteIOps": null,
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DeviceRequests": null,
            "KernelMemory": 0,
            "KernelMemoryTCP": 0,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": null,
            "OomKillDisable": false,
            "PidsLimit": null,
            "Ulimits": null,
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "MaskedPaths": [
                "/proc/asound",
                "/proc/acpi",
                "/proc/kcore",
                "/proc/keys",
                "/proc/latency_stats",
                "/proc/timer_list",
                "/proc/timer_stats",
                "/proc/sched_debug",
                "/proc/scsi",
                "/sys/firmware"
            ],
            "ReadonlyPaths": [
                "/proc/bus",
                "/proc/fs",
                "/proc/irq",
                "/proc/sys",
                "/proc/sysrq-trigger"
            ]
        },
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/023249bbfd4df4b3f0dcbc9be7ca09d187f71ed0bc5059ce525e58aca1adf1ae-init/diff:/var/lib/docker/overlay2/08a0e1d413960f76d1f62861f7ec8641eec1789495f2199397ad63f853d775b9/diff",
                "MergedDir": "/var/lib/docker/overlay2/023249bbfd4df4b3f0dcbc9be7ca09d187f71ed0bc5059ce525e58aca1adf1ae/merged",
                "UpperDir": "/var/lib/docker/overlay2/023249bbfd4df4b3f0dcbc9be7ca09d187f71ed0bc5059ce525e58aca1adf1ae/diff",
                "WorkDir": "/var/lib/docker/overlay2/023249bbfd4df4b3f0dcbc9be7ca09d187f71ed0bc5059ce525e58aca1adf1ae/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [],
        "Config": {
            "Hostname": "718891259ec5",
            "Domainname": "",
            "User": "",
            "AttachStdin": true,
            "AttachStdout": true,
            "AttachStderr": true,
            "Tty": true,
            "OpenStdin": true,
            "StdinOnce": true,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/bin/bash"
            ],
            "Image": "centos",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {
                "org.label-schema.build-date": "20200611",
                "org.label-schema.license": "GPLv2",
                "org.label-schema.name": "CentOS Base Image",
                "org.label-schema.schema-version": "1.0",
                "org.label-schema.vendor": "CentOS"
            }
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "e95fd62687e905550f2052f80362370f565d1e201cc2d37c45ba4dcc4f9975aa",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {},
            "SandboxKey": "/var/run/docker/netns/e95fd62687e9",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "d3b1128ead012a85991473f2353d2b37e4f68a34f6eab5e05e240b1da1f5c2c4",
            "Gateway": "172.17.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.17.0.3",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:11:00:03",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "5bc82b228aae9d7965bd8db477b3d0e89a96195c68746ce1577852e3c7d6891c",
                    "EndpointID": "d3b1128ead012a85991473f2353d2b37e4f68a34f6eab5e05e240b1da1f5c2c4",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.3",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:03",
                    "DriverOpts": null
                }
            }
        }
    }
]

```

### 进入当前正在运行的容器

```shell
docker exec -it 容器id 


[root@localhost ~]# docker exec -it e4026153e96b /bin/bash
[root@e4026153e96b /]# ps -aux
USER        PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root          1  0.0  0.1  12008  2104 pts/0    Ss+  12:36   0:00 /bin/bash
root         14  0.0  0.1  12008  1948 pts/1    Ss+  13:41   0:00 /bin/sh
root         20  0.0  0.0  12008  1684 pts/2    Ss+  13:42   0:00 /bin/sh
root         25  0.3  0.1  12008  2108 pts/3    Ss   13:42   0:00 /bin/bash
root         38  0.0  0.1  47496  2060 pts/3    R+   13:42   0:00 ps -aux
# 方式二
docker attach 容器id 
docker attach  e4026153e96b
# docker exec # 进入容器后开启一个新的终端在里面操作
# docker attach  # 进入容器正在执行的终端 不会启动新的进程
```

### 从容器内拷贝东西到宿主机

```
[root@localhost ~]# docker attach e4026153e96b
[root@e4026153e96b /]# echo 123 >a.txt
[root@e4026153e96b /]# echo 123 a.txtread escape sequence
[root@localhost ~]# ls
anaconda-ks.cfg  initial-setup-ks.cfg
[root@localhost ~]# docker cp e4026153e96b:/a.txt ./
[root@localhost ~]# ls
anaconda-ks.cfg  a.txt  initial-setup-ks.cfg
[root@localhost ~]# more a.txt 
123
```



## 参考资料

[掘金](https://juejin.im/post/5b260ec26fb9a00e8e4b031a#heading-21) [官网](https://docs.docker.com/) [狂神说Docker](https://www.bilibili.com/video/BV1og4y1q7M4?)