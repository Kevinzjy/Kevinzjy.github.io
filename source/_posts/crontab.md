---
title: CentOS6 开机自动运行脚本
date: 2020-12-26 16:19:55
categories: Linux
tags: [Linux, Server]
---

为了让安装的 Mysql 和 Tomcat6 服务可以重启后自动运行，可以利用 `crontab` 命令指定开机后运行自定义脚本


**脚本 `/root/restart_mysql.sh` 内容**

```bash
#!/bin/sh
/etc/init.d/mysqld start
/usr/local/src/tomcat6/bin/startup.sh
```

**使用 `crontab -e` 命令对定时任务进行编辑，增加一行**

```
@reboot /root/restart_mysql.sh
```

这样可以实现每次重启后自动运行脚本中的任务，而不用手动开启服务
