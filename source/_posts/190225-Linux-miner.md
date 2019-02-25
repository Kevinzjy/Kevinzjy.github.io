---
title: Linux 服务器中异常排查
date: 2019-02-25 18:57:47
categories: GitHub
tags: [GitHub, ]
---

今天，在登录服务器时，发现非常卡顿，并且计算中心提醒服务器最近占用了大量的下行带宽，因此怀疑服务器上可能存在异常的程序

#### 检查网络流量

```bash
iftop
```

{% asset_img iftop.png %}

最右侧三列为 2s/4s/10s 内占用平均带宽

#### 通过 top 查看系统 CPU 和内存占用

```bash
top
```

#### 检查异常进程相关文件

```bash
lsof -p ${pid}
ls -al /proc/${pid}
ls -al /proc/${pid}/fd
cat /proc/${pid}/fd/255 > /tmp/${pid}.255
cat /proc/${pid}/exe > /tmp/${pid}.exe
```

#### 检查计划任务

```bash
ls /var/spool/cron
ls /etc/cron.hourly
less -S /etc/crontab
```

#### 检查自动启动的服务

```bash
service --status-all | grep running
chkconfig --list | grep :on
less -S /etc/rc.local
```

#### 检查自动启动脚本

```bash
ls -lt /etc/init.d | head
less -S /etc/rc.local
ls /etc/rc.d
```

#### 清除文件名异常的文件

经过检查，发现在 /tmp 文件夹下存在一个名称为 ".       " 的文件，然而一开始在 CentOS 的 shell 里面只看到了 "."，因此一直无法删除。最后只好使用文件的 index number 进行操作

```
➜  tmp ls -al
total 2556
drwxrwxr-x 2 zhangjy zhangjy    4096 Feb 25 19:13  .
-rwxrwxr-x 1 zhangjy zhangjy 1981324 Nov  2 06:36 '.      '
drwxrwxr-x 3 zhangjy zhangjy    4096 Feb 25 18:59  ..
-rwxr-xr-x 1 zhangjy zhangjy  625867 Feb 25 19:13  libudev.so
```

获取 index number

```bash
➜  tmp ls -ial
total 2556
118621432 drwxrwxr-x 2 zhangjy zhangjy    4096 Feb 25 19:13  .
118621437 -rwxrwxr-x 1 zhangjy zhangjy 1981324 Nov  2 06:36 '.      '
118621454 drwxrwxr-x 3 zhangjy zhangjy    4096 Feb 25 18:59  ..
118621435 -rwxr-xr-x 1 zhangjy zhangjy  625867 Feb 25 19:13  libudev.so
```

根据 index number 删除文件

```
find . -inum 118621437 -exec rm -i {} \;
```

参考资料: https://blog.51cto.com/qicheng0211/1928738