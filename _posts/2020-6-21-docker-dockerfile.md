---
layout: post
title: Docker笔记第三天
categories: [Docker]
description: Docker
keywords: Docker Docker
---

Dockerfile 就是用来构建 docker 镜像的构建文件！命令脚本！ 通过这个脚本可以生产镜像

## 初始Dockerfile

Dockerfile 构建步骤：

1、编写一个dockerfile 文件

2、docker build 构建成为一个镜像

3、docker run 运行镜像

4、docker push 发布镜像

```shell
#创建一个dockerfile
#文件内容如下
[root@localhost docker-file]# cat dockerfile1 
FROM centos

VOLUME ["volume01","volume02"] #匿名挂载

CMD echo "-------->end<------"

CMD /bin/bash
```

![docker-dockerfile-centos-ops](https://i.opsta.cn/docker/docker-dockerfile-centos-ops.png)



## DockerFile构建过程

**基础知识**：

1、每个保留关键字 (指令) 都是必须是大写字母

2、命令从上到下执行顺序

3、# 表示注释

4、每一个指令都会创建提交一个新的镜像层，并提交

5、常见指令

![docker-dockerfile](https://i.opsta.cn/docker/docker-dockerfile.png)

```shell
FROM		#基础镜像，一切从这里开始
MAINTAINER	#镜像是谁写的，姓名+邮箱（统一格式）
RUN			#镜像构建的时候需要运行的命令
ADD			#步骤，例如要搭建tomcat镜像，要加入tomcat压缩包
WORKDIR		#镜像的工作目录
VOLUME		#挂载的目录位置
EXPOST		#暴露端口配置
CMD			#指定这个容器启动的时候要运行的命令，只有最后一个会生效，即可被代替
ENTRYPOINT	#指定这个容器启动的时候要运行的命令，可以直接追加命令
ONBUILD		#当构建一个被继承当镜像，DockerFile这个时候就会执行ONBUILD的指令（触发指令）
COPY		#类似ADD，将文件拷贝到镜像中
ENV 		#构建的时候设置环境变量
```

## 名词解释

DockerFile：构建文件，定义了一切的步骤 源代码

DockerImages：通过 DockerFile 构建生成的镜像

Docker容器：容器就是镜像运行起来提供服务器

## 实战测试

编写一个简单的centos容器 带 vim 跟 net-tools 的

```shell
#1 编写dockerfile的文件
[root@localhost docker-file]# cat dockerfile-centos 
#以哪个镜像为基础
FROM centos
MAINTAINER ops<me@opsta.cn> #谁维护的

#设置变量
ENV MYPATH /usr/local
WORKDIR $MYPATH #用户进入容器的目录

#执行安装
RUN yum -y install vim
RUN yum -y install net-tools

#暴露端口
EXPOSE 80

#输出
CMD echo $MYPATH
CMD echo ----->opsta<-------
CMD /bin/bash
# 2 通过这个文件构建镜像
#命令 docker build -f dockerfile-centos mycentos:0.1 .
Successfully built 5a7c6e767b10
Successfully tagged mycentos:0.1
# 3 测试运行

```

列出本地进行的变更历史

```
docker history 5a7c6e767b10
```

> CMD 和 ENTRYPOINT 的区别

```shell
CMD #指定容器启动的时候要运行的命令,只有最后一个会生效 可被替代
ENTRYPOINT ##指定容器启动的时候要运行的命令,可以追加
```

测试CMD

```shell
#编写 dockerfile文件
[root@localhost docker-file]# more dockerfile-cmd-test 
FROM centos
CMD ["ls","-a"]

#构建镜像
[root@localhost docker-file]# docker build -f dockerfile-cmd-test -t cmdtest .

#run 运行
[root@localhost docker-file]# docker run -it cmdtest
.   .dockerenv	dev  home  lib64       media  opt   root  sbin	sys  usr
..  bin		etc  lib   lost+found  mnt    proc  run   srv	tmp  var

[root@localhost docker-file]# docker run -it cmdtest -l
docker: Error response from daemon: OCI runtime create failed: container_linux.go:349: starting container process caused "exec: \"-l\": executable file not found in $PATH": unknown.
ERRO[0000] error waiting for container: context canceled 

```

测试ENTRYPOINT 

```shell
[root@localhost docker-file]# more dockerfile-entrtpoint-test 
FROM centos
ENTRYPOINT  ["ls","-a"]
[root@localhost docker-file]# docker build -f dockerfile-entrtpoint-test -t entrtpoint-test .
Successfully built 73527121de0b
Successfully tagged entrtpoint-test:latest
[root@localhost docker-file]# docker run -it entrtpoint-test 
.   .dockerenv	dev  home  lib64       media  opt   root  sbin	sys  usr
..  bin		etc  lib   lost+found  mnt    proc  run   srv	tmp  var

[root@localhost docker-file]# docker run 73527121de0b -l 
total 0
-rwxr-xr-x.   1 root root   0 Jun 23 08:31 .dockerenv
lrwxrwxrwx.   1 root root   7 May 11  2019 bin -> usr/bin
drwxr-xr-x.   5 root root 340 Jun 23 08:31 dev
drwxr-xr-x.   1 root root  66 Jun 23 08:31 etc
drwxr-xr-x.   2 root root   6 May 11  2019 home
lrwxrwxrwx.   1 root root   7 May 11  2019 lib -> usr/lib
lrwxrwxrwx.   1 root root   9 May 11  2019 lib64 -> usr/lib64
drwx------.   2 root root   6 Jun 11 02:35 lost+found
drwxr-xr-x.   2 root root   6 May 11  2019 media
drwxr-xr-x.   2 root root   6 May 11  2019 mnt
drwxr-xr-x.   2 root root   6 May 11  2019 opt
dr-xr-xr-x. 217 root root   0 Jun 23 08:31 proc
dr-xr-x---.   2 root root 162 Jun 11 02:35 root
drwxr-xr-x.  11 root root 163 Jun 11 02:35 run
lrwxrwxrwx.   1 root root   8 May 11  2019 sbin -> usr/sbin
drwxr-xr-x.   2 root root   6 May 11  2019 srv
dr-xr-xr-x.  13 root root   0 Jun 21 03:22 sys
drwxrwxrwt.   7 root root 145 Jun 11 02:35 tmp
drwxr-xr-x.  12 root root 144 Jun 11 02:35 usr
drwxr-xr-x.  20 root root 262 Jun 11 02:35 var

```

## 制作 Tomcat 镜像

1、准备镜像文件 Tomcat 压缩包，jdk 的压缩包

2、编写Dockerfile ，官方命名 **Dockerfile** ，build会自动寻找这个文件 不需要 -f 指定

```shell
FROM centos
MAINTAINER opsta<me@opsta.cn>

ADD jdk-8u231-linux-x64.tar.gz /usr/local/
ADD apache-tomcat-8.5.55.tar.gz /usr/local/

RUN yum -y install vim

ENV MYPATH /usr/local/
WORKDIR $MYPATH

ENV JAVA_HOME  /usr/local/jdk-8u231
ENV CLASSPATH=$JAVA_HOME/bin
ENV CATALINA_HOME /usr/local/apache-tomcat-8.5.55
ENV CATALINA_BASH /usr/local/apache-tomcat-8.5.55
ENV PATH=.:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin:$PATH

EXPOSE 8080

CMD /usr/local/apache-tomcat-8.5.55/bin/startup.sh && tail -f  /usr/local/apache-tomcat-8.5.55/logs/catalina.out

```

3、构建镜像

```
docker build -t mytomcat .
```

4、运行 

```shell
 # -v /opt/tomcat/test/ 可要可不要
docker run -d -p 9090:8080 --name tomcat -v /opt/tomcat/test/:/usr/local/apache-tomcat-8.5.55/webapps/test -v /opt/tomcat/logs:/usr/local/apache-tomcat-8.5.55/logs mytomcat
```

5、测试访问

![docker-dockerfile-tomcat-quickstart](https://i.opsta.cn/docker/docker-dockerfile-tomcat-quickstart.png)

## 发布镜像

1、地址 https://hub.docker.com/ 注册自己的账号

2、登陆 

```
dokcer login
```

3、给自己要提交的镜像打标签

```shell
#docker tag 镜像id 新名字
docker tag a6fb4f06ee09 meethong/tomcat:1.0
```

4、提交

```shell
docker push meethong/tomcat:1.0
```

