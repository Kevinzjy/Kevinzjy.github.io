---
title: Linux解决根目录ls无响应问题
date: 2019-01-15 16:47:15
categories: Linux
tags: [Linux, ]
---

今天发现服务器某节点，在根目录执行ls时会卡死

首先用 `which ls` 命令查看 `ls` 命令的alias

```
alias ls='ls --color=auto'
	/bin/ls
```

直接执行 `/bin/ls` 命令可以获得结果，因此需要追踪 `strace ls --color=auto /` 在哪个目录出现了问题

查看输出结果可以发现，会卡在某个NFS文件夹，umount 之后就可以了

```
umount -l /data
```
