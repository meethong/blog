

```shell
docker create -v /var/lib/postgresql/data --name PostgresData alpine
#默认用户名postgres
docker run -p 5432:5432 --name postgres  --restart=always -e POSTGRES_PASSWORD=postgres -d --volumes-from PostgresData postgres 
```



```shell
docker run --name redis -p 6379:6379 -v /D/cache/Docker/app/redis/conf/redis.conf:/etc/redis/redis.conf -v /D/cache/Docker/app/redis/data:/data/ -d redis:latest   redis-server --appendonly yes 

```





```shell
#先启动 再把配置文件copy出来
docker run -p --name demo 61616:61616 -p 8161:8161  rmohr/activemq #
docker cp 4ad2247707ef:/opt/apache-activemq-5.15.6/data/ D:/cache/Docker/app/activemq/ #copy
docker cp 4ad2247707ef:/opt/apache-activemq-5.15.6/conf/ D:/cache/Docker/app/activemq/ #copy
docker run -p 61616:61616 -p 8161:8161 -v /D/cache/Docker/app/activemq/conf:/opt/activemq/conf -v /D/cache/Docker/app/activemq/data:/opt/activemq/data rmohr/activemq #启动
```





```shell
docker run -it --name rss --restart=always -e SELF_URL_PATH=http://192.168.72.50/ -e DB_HOST=192.168.72.50 -e DB_PORT=5432  -e DB_USER=postgres -e DB_PASS=postgres  -p 80:80 -d wangqiru/ttrss
```





```bash
docker run -p 5432:5432 --name postgres  --restart=always -e POSTGRES_PASSWORD=postgres -d -v /opt/ops/data/postgresql:/var/lib/postgresql/data  postgres 
```

