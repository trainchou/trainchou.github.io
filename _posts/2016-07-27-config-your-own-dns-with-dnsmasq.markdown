---
layout: post
title: 使用 Dnsmasq 自建 DNS
date: '2016-07-27 02:48:57'
tags:
- dns
- dnsmasq
---

Dnsmasq 的功能比较多，本文介绍它的 DNS 功能。
有些 Linux 发行版中预装了 Dnsmasq， 如果没有可以通过软件管理器安装。
Dnsmasq 的配置文件一般在 `/etc/dnsmasq.conf`。
几个关键的参数：

* `port` DNS的端口
* `resolv-file` 指向另一个文件，里面定义了上游 DNS，当遇到本地的 DNS 无法解析的域名，则会向上游 DNS 进行查询
* `listen-address` 这里填写监听的 ip，如果要应用于所有的网卡可设为`0.0.0.0`，或者设为指定的 ip 也可

最为重要的参数就是 `address` 了。

这个参数即为注册的 DNS 记录。
例如：
`address=/test.domain.com/192.168.1.1`

Dnsmasq 比 hosts 牛逼的一点就是支持泛域名解析，例如：
`address=/.domain.com/192.168.1.1` 
会将 `domain.com` 下的所有子域名都解析到指定的 ip 上，另有配置的除外。

配置好以后重启一下 Dnsmasq 服务即可生效。

Dnsmasq 可以用于本机作为高级的 hosts 使用，也可搭建在国外的 VPS 上，防止 DNS 污染。

-----------------------
更新：CentOS7 系统中会有一个默认开机由 libvirtd 启动的 dnsmasq，和一个systemd 管理的 dnsmasq 服务。建议使用 `virsh net-autostart --disable default` 命令关闭由 libvirtd 启动的dnsmasq，通过 `systemctl enable dnsmasq.service` 实现开机启动。