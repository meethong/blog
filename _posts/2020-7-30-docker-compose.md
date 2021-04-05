---
layout: post
title: Docker Compose学习
categories: [Docker]
description: Docker
keywords: Docker
---

通过Compose，可以使用YML文件来配置应用程序需要的所有服务。然后，使用一个命令，就可以从YML文件配置置中创建并启动所有服务。Compose 是Docker官方的开源项目，需要安装！

## Docker Compose

## 安装

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.26.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose 
```

```bash
sudo chmod +x /usr/local/bin/docker-compose #授权运行
```

[入门手册](https://docs.docker.com/compose/gettingstarted)

1、应用 app.py

```python

import time

import redis
from flask import Flask

app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)


def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)


@app.route('/')
def hello():
    count = get_hit_count()
    return 'Hello World! I have been seen {} times.\n'.format(count)

```

2、Dockerfile 应用打包为镜像

```dockerfile
FROM python:3.7-alpine
WORKDIR /code
ENV FLASK_APP app.py
ENV FLASK_RUN_HOST 0.0.0.0
RUN apk add --no-cache gcc musl-dev linux-headers
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
EXPOSE 5000
COPY . .
CMD ["flask", "run"]
```

3、Docker-compose yaml文件（定义整个服务 需要的环境。 web、redis) 完成的上线服务！

```yaml
version: '3'
services:
  web:
    build: .
    ports:
      - "5000:5000"
  redis:
    image: "redis:alpine"
```

4、启动compose 项目（docker-compose up）

![docker_2020-07-30_17-47-39](https://cdn.jsdelivr.net/gh/meethong/images@latest/docker/docker_2020-07-30_17-47-39.png)

流程：

1、创建网络

2、执行Docker-compose yaml

3、启动服务。

**Compose** 中有两个重要的概念：
服务 ( service )： 一个应用的容器， 实际上可以包括若干运行相同镜像的容器实例。
项目 ( project )： 由一组关联的应用容器组成的一个完整业务单元， 在**docker-compose.yml** 文件中定义  

**Compose** 的默认管理对象是项目， 通过子命令对项目中的一组容器进行便捷地生命周期管理。
**Compose** 项目由 **Python** 编写， 实现上调用了 **Docker** 服务提供的 API 来对容器进行管理。 因此， 只要所操作的平台支持 **Docker API**， 就可以在其上利用**Compose** 来进行编排管理。  

**yaml 规则**

```yaml
#3层 ！

version: '' #版本
services: #服务
  服务1: web
	# 服务配置 docker
	images
	build
	network
	...
  服务2: redis
  #其他配置 网络/卷、全局规则
volumes:
networks:
configs:

```

## 常用命令

```shell
docker-compose up -d #后台启动
docker-compose up --build #重新构建
docker-compose ps #列出所有运行容器
```


