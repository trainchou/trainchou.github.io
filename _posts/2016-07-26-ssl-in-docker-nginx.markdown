---
layout: post
title: 让 docker 环境中的 NGINX 使用 Let's Encrypt 的 SSL 证书
date: '2016-07-26 03:18:48'
tags:
- ssl
- nginx
- docker
- lets-encrypt
---

之前在国外的 VPS 上使用过 Let's Encrypt 的 SSL 证书，但是此次是在国内网络下使用 docker 环境中的 NGINX，操作略有区别。

理论上来说，可以把生成证书这一步放在 docker 中完成，但是为了图省事，我没有尝试这样做。而是在宿主机上生成 SSL 证书。

生成证书的方法：

```
git clone https://github.com/letsencrypt/letsencrypt  
cd letsencrypt  
./letsencrypt-auto certonly --standalone --email admin@mail.com -d www.domain.com
```
其中， admin@mail.com 替换为管理员的电子邮箱地址， www.domain.com 替换为使用 SSL 证书的域名。Let's Encrypt 最多支持添加100个域名，但不支持通配符。
这里要注意提前将本机的80端口和443端口空出来。

在运行最后一步命令的时候可能因为网络的问题，部分依赖包无法下载到，此处建议添加 Google 的 DNS ：

```
8.8.8.8
8.8.4.4
```

当看到返回 Congratulations 的字样则表示证书生成成功了。

生成的证书存地址会在返回的文本中显，一般会存放在 `/etc/letsencrypt/live/$domain` 目录下，注意该目录下的文件其实是链接，文件的真实的地址在`/etc/letsencrypt/archive/$domain`（由于 Let's Encrypt 每三个月要续约一次，续约后会生成新的证书，存放在 archive 文件夹下，然后自动更新 live 文件下的链接。）。因此，我们在把证书挂载到 docker 里时，应注意要挂载真实的文件，而不可挂载链接，不然会导致找不到文件。

将文件挂载入镜像后，修改 docker 中的 NGINX 的配置文件，将证书的地址加上即可。

例如：
```
server {  
    listen 443 ssl;
    server_name domain.com;
    ssl_certificate         /etc/letsencrypt/live/domain.com/fullchain.pem;
    ssl_certificate_key     /etc/letsencrypt/live/domain.com/privkey.pem;
    location / {

        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_pass         http://127.0.0.1:8080;
    }
}
```

这样设置以后启动 docker 容器，应该就可以通过 https 访问你的网站了。续期的时候可以写个脚本，将新生成的证书更新到 docker 容器中。

下附我的 NGINX 的 docker-compose.yml, 供各位参考：

```
nginx:
    image: "nginx"
    ports:
        - 80:80
        - 443:443
    volumes:
        - ./nginx.conf:/etc/nginx/nginx.conf:ro
        - ./html:/usr/share/nginx/html:ro
        - ./letsencrypt:/etc/letsencrypt/live/domain.com:ro
```
续期脚本（续期前应先关闭80和443端口）：

```
#!/bin/bash
SERVER="https://acme-v01.api.letsencrypt.org/directory" # 目前Let's Encrypt的签发服务器
DOMAINS=(okwoo.com tieba.okwoo.com bbs.okwoo.com) # 需要认证的域名，用空格分隔
# 生成域名参数列表，类似"-d xxx.yourdomain.com -d xx2.youdomain.com"
for i in ${DOMAINS[*]}
do
 domarg="$domarg -d $i"
done
cd /root/letsencrypt
git pull origin master
./letsencrypt-auto --server $SERVER $domarg --renew --agree-dev-preview certonly
```
把证书拷贝到 docker-compose 目录下的相应位置，docker-compose up 即可完成续期。

--------------
Update(2016.7.27)：
更简单的方式是使用这个项目：https://github.com/SteveLTN/https-portal