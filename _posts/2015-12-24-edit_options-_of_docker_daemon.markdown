---
layout: post
title: 给 docker 添加启动项参数
date: '2015-12-24 03:33:23'
tags:
- bigdata
---

给 docker 添加启动项参数有多种方法。本文以使得 docker 在 2375 端口上访问为例，用户可以通过这个端口在远端访问 docker remote api，这里需要提醒大家的是把这个端口暴露在公网上是不安全的。

**使用 docker daemon 命令启动时直接在命令后加参数。**

  如 `docker daemon -H tcp://0.0.0.0:2375/ &`



 **支持 systemd 的系统（如 CentOS 7+、Ubuntu 16.04+、CoreOS
等）** : 在 `/usr/lib/systemd/system/docker.service` 文件的 `ExecStart`项改为：
```
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375
```
然后
```
systemctl daemon-reload
systemctl restart docker.service
```
即可。

**其他方式：**

**1) CoreOS** : `/etc/systemd/system/docker-tcp.socket`

内容：

```
[Unit]
Description=Docker Socket for the API

[Socket]
ListenStream=2375
BindIPv6Only=both
Service=docker.service

[Install]
WantedBy=sockets.target
```

应用该配置文件：
```
sudo systemctl enable docker-tcp.socket
sudo systemctl stop docker
sudo systemctl start docker-tcp.socket
sudo systemctl start docker
```
即可。

**2) 写在 docker 的配置文件中:**
 
 **配置文件一般位于**  `/etc/default/docker`

 编辑文件，添加（修改）行，如：

 ` DOCKER_OPTS="-H tcp://0.0.0.0:2375/`

 然后通过 `service docker restart` 命令重启 docker 即可生效。