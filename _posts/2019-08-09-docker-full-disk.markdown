---
layout: post
title: Docker 防坑笔记—— docker 目录过大
date: '2019-08-09 15:29:13'
tags:
- docker
---

## Docker Root Dir 一定要设置在一个容量大的分区上

很多次故障都是因为 docker root dir 磁盘写满，或者 docker 把系统的根目录写满导致服务器宕机。  
杜绝这种事情发生，只有一个办法：把 docker root dir 设置在一个容量大的分区上。  
当然，如果你正准备安装 docker，那大可在安装之前就通过fstab将一个大的磁盘分区挂载到默认的 docker root dir （`/var/lib/docker/`)上。当然，修改 fstab 有一定的风险，如果你对该文件的格式不熟悉，很容易出错，导致系统启动失败。另外如果你已经在使用 docker，挂载分区对方法就有些繁琐，这块有时间我会专文讲解。这里主要介绍修改 docker root dir 的方法。  
本文有参考掘金上的[这篇文章](https://juejin.im/post/5badee89e51d450e6160312a)，不过修正了原文中的一些错误。

### 停止docker服务

    systemctl stop docker

### 创建新的docker目录，执行命令df -h,找一个大的磁盘

我的系统里 `/opt` 目录是单独挂载的，有500G的空间，因此，我在`opt`下面建了`/opt/docker/`目录

    mkdir -p /opt/docker

### 迁移/var/lib/docker目录下面的文件到/opt/docker

迁移后的完成docker路径：`/opt/docker/`

    rsync -avz /var/lib/docker/ /opt/docker/

### 配置 /etc/systemd/system/docker.service.d/devicemapper.conf

查看`/etc/systemd/system/docker.service.d`目录及`devicemapper.conf`是否存在。如果不存在，就新建

    mkdir -p /etc/systemd/system/docker.service.d/
    vi /etc/systemd/system/docker.service.d/devicemapper.conf

`devicemapper.conf`添加如下内容：

    [Service]
    ExecStart=
    ExecStart=/usr/bin/dockerd --graph=/opt/docker

这里注意第一个`ExecStart=`不能漏掉，不然启动 docker 会报错。

### 重启docker

    systemctl daemon-reload
    systemctl start docker

### 确认Docker Root Dir修改是否已经生效

    docker info
    ...
    Docker Root Dir: /opt/docker
    Debug Mode (client): false
    Debug Mode (server): false
    Registry: https://index.docker.io/v1/
    ...

### 启动成功后，再确认之前的镜像是否还在

    docker images
    REPOSITORY TAG IMAGE ID CREATED SIZE
    10.80.177.233/policy 2.1.2 64ac4e178cd2 2 hours ago 818 MB
    10.80.177.233/crm 2.1.3 d7636fbb7a29 2 hours ago 762 MB

### 确定容器没问题后删除`/var/lib/docker/`目录
<!--kg-card-end: markdown-->