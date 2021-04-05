---
layout: post
title: 在Centos上用Docker安装tt-rss
categories: [Docker, TT-RSS]
description: Docker TT-RSS
keywords: Docker, TT-RSS

---

Tiny Tiny RSS是一种基于Web的免费开放源新闻订阅（RSS / Atom）阅读器和汇总器

## 前提条件

- 切换Yum源为阿里云仓库

   [阿里云仓库](https://developer.aliyun.com/mirror/centos?spm=a2c6h.13651102.0.0.3e221b11fjkN0F)

- 安装Docke

  [安装文档](https://docs.docker.com/engine/install/centos/#prerequisites)

- 安装Docker Compose

  [安装文档](https://docs.docker.com/compose/install/)
## 安装

```sh
git clone https://git.tt-rss.org/fox/ttrss-docker-compose.git ttrss-docker && cd ttrss-docker
```

复制`.env-dist`到`.env`和编辑你需要改变任何相关的变量。

- `SELF_URL_PATH`在网络浏览器中打开时，您可能必须更改其名称，使其等于标准的tt-rss URL。如果此字段设置不正确，您可能会在tt-rss致命错误消息中看到正确的值。

注意：容器重启时，会自动`SELF_URL_PATH`在生成的tt-rss中更新`config.php`。您无需为此进行`config.php`手动修改。

- 默认情况下，容器绑定到**本地主机**端口**8280**。如果你想在容器中在网上访问，而无需使用反向代理共享相同的主机，你将需要删除`127.0.0.1:`的`HTTP_PORT`变量`.env`。

#### 构建并启动容器

```sh
docker-compose up --build
```

访问页面

![](https://i.opsta.cn/tt-rss/tt-rss-install-ok.png)

```
默认管理员密码:admin/password
```

登陆自己导入一些订阅源 舒服了舒服了

![](https://i.opsta.cn/tt-rss/tt-rss-install-load.png)

可以把`Postgresql`从`Docker`中映射出来方便备份恢复 修改`docker-compose.yml`

![](https://i.opsta.cn/tt-rss/tt-rss-postgresql-up.png)

#### 更新容器脚本

1. 停止容器： `docker-compose down && docker-compose rm`
2. 从git：更新脚本，`git pull origin master`并对`.env`，等进行必要的修改。
3. 重建并启动容器： `docker-compose up --build`

## 参考

[tt-rss安装文档](https://tt-rss.org/wiki/InstallationNotes)