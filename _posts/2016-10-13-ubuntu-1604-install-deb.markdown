---
layout: post
title: Ubuntu 16.04 无法直接安装 deb包的解决方法
date: '2016-10-13 08:11:37'
tags:
- ubuntu
- deb
---


[![Ubuntu 16.04 无法直接安装 deb包的解决方法](http://www.ubuntudoc.com/wp-content/uploads/2015/12/UBUNTU-16.04-LTS-810x623-1.png)](https://blog.kittypanic.com/ubuntu-1604-install-deb/)
Ubuntu 16.04 LTS 可以使用以下命令安装 deb 包：

`sudo dpkg -i 文件名.deb`

如果安装程序被锁住了，一般会报以下错误：

`dpkg: error: dpkg status database is locked by another process`

使用命令：

`sudo rm /var/lib/dpkg/lock`

若安装失败，则使用命令：

`apt-get install -f`

再执行，`sudo dpkg -i 文件名.deb`，即可。

另外 dpkg 命令卸载应用的方法是：

`sudo dpkg -r package_name`

此处注意 `package_name` 不是安装包的文件名，而是应用程序的名称。