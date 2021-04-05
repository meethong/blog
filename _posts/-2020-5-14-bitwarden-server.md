





依赖安装

```
https://docs.docker.com/compose/install/
```

```
https://www.docker.com/products/container-runtime#/download
```

```
sudo curl -L "https://github.com/docker/compose/releases/download/1.25.5/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```



```
 curl -s -o bitwarden.sh     https://raw.githubusercontent.com/bitwarden/server/master/scripts/bitwarden.sh     && chmod +x bitwarden.sh
```

```
 yum install docker
 systemctl start docker
```

```
docker run -d \
    --name bitwarden \
    -p 8080:80 \
    -p 3012:3012 \
    --restart=always \
    -e SIGNUPS_ALLOWED=true \
    -e WEB_VAULT_ENABLED=true \
    -e DOMAIN=https://passwd.opsta.cn \
    -v /data/bitwarden/data:/data \
    bitwardenrs/server:latest
```

```
docker run -d --name bitwarden -v /data/:/data/ -p 80:80 bitwardenrs/server:latest
```





```
docker run -d --name bitwarden \    -e SIGNUPS_ALLOWED=false \    -e INVITATIONS_ALLOWED=false \    -e ADMIN_TOKEN=step2_generated_token \    -e ROCKET_TLS='{certs="/path/to/docker/ssl/fullchain.cer",key="/path/to/docker/ssl/yourdomain.com.key"}' \    -e DOMAIN=https://yourdomain.com \    -e LOG_FILE=/path/to/log \    -e LOG_LEVEL=warn -e EXTENDED_LOGGING=true \    -e DATA_FOLDER=/path/to/data/folder \    -p 443:80 \    -v /path/to/host/ssl/:/path/to/docker/ssl/    -v /path/to/host/data/folder:/path/to/docker/data/folder \    bitwardenrs/server:latest
```

```
授权码：44WI16OMTO
产品密钥:UAPFE-9452B-FEE49-29210-73831-71C07-B4CC6
IP:192.168.0.125
制作证书
时间三个月
用户20
```

```

===========配置参数============
地址：passwd.opsta.cn
端口：443
uuid：db74360f-1583-4bb5-bf81-80e05efceae8
额外id：32
加密方式：aes-128-gcm
传输协议：ws
别名：WP博客+V2ray
路径：/5168
底层传输：tls

```

