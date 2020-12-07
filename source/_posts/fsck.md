---
title: 硬盘报错修复
date: 2020-12-07 16:28:43
categories: Linux
tags: [Linux, Server]
---

Linux 开机出现硬盘报错，自动进入恢复模式。可以使用 fsck 命令对硬盘错误进行修复

```bash
# 检查硬盘错误
fsck -y

# 重新挂载根目录
mount -o remount,rw /

# 重新启动
reboot
```
