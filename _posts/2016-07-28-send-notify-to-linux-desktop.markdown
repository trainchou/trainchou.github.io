---
layout: post
title: 在 Linux 上发送桌面通知
date: '2016-07-28 02:25:05'
tags:
- linux
- crontab
---

对于长期在电脑前工作的人，往往在不知不觉中盯着显示器过久，导致眼睛疲劳。
于是就产生了定期发送桌面通知来提醒自己休息一下的需求。
首先需要安装需安装 libnotify-bin 包，我的 Ubuntu 16.04 上面已经有了。
然后将以下命令写入 crontab：
```
export DISPLAY=:0.0 && export XAUTHORITY=/home/${yourname}/.Xauthority && sudo -u ${yourname} /usr/bin/notify-send "Have a rest!"
```

注意应当写入 root 的 crontab 中。

参考文献：

http://zankbo.com/archives/32.html

http://forum.ubuntu.org.cn/viewtopic.php?t=316753