---
layout: post
title: 使用用户名和密码通过 SSH 登录 CoreOS 主机的方法
date: '2016-11-09 08:54:07'
tags:
- coreos
- ssh
- password
- key
- root
---

![](https://encrypted-tbn2.gstatic.com/images?q=tbn:ANd9GcQ5x8at5GAwFmUk4Vp8c_BxWmcSwYAG6vIdEJvmX8L-AvhSPidF)

CoreOS 默认情况下是只开通 core 这个用户的 SSH-key 的登录权限的。而我们在实际工作中遇到需要用密码登录以及以指定用户（包括 root）通过密码登录的需求，例如某些主机监控系统只支持用户名密码的认证方式。这时我们就需要更改 CoreOS 的设置来实现这一功能。**但注意一点，再复杂的密码，都是可以被破解的，所以能用 key 登录的话尽量不要开启密码登录。**

这一设置存在于 CoreOS 的 SSH 的配置文件(/etc/ssh/sshd-config)中。该文件是只读的，我们需要在 CoreOS 的 Cloud-config 中更改。
```
#cloud-config

write_files:
  - path: /etc/ssh/sshd_config
    permissions: 0600
    owner: root:root
    content: |
      # Use most defaults for sshd configuration.
      UsePrivilegeSeparation sandbox
      Subsystem sftp internal-sftp

      PermitRootLogin yes
      AllowUsers core root
      PasswordAuthentication yes
      ChallengeResponseAuthentication no
```
按以上配置：

**PermitRootLogin yes** 允许 root 用户通过 SSH 登录

**AllowUsers core root user1** 允许 core、root、user1 通过SSH 登陆

**PasswordAuthentication yes** 允许通过密码登录

即可。

若你指定的用户没有设置密码，则可通过命令：`passwd {username}`来设置密码。或者在 Cloud-config 中指定该用户的密码。

重新启动系统后，即可实现密码登录。


