---
title: Ubuntu下部署NAS
date: 2017-04-30 16:18:15
categories: Linux
tags: [Linux, Ubuntu, NAS]
---

## 安装Ubuntu NAS

<!-- more -->

### 开机自动挂载移动硬盘

```bash
sudo fdisk -l
sudo umount /dev/sdb1
sudo mkfs -t ext4 /dev/sdb1
```

### 使用bash-it 或者 zsh + oh-my-zsh
```bash
sudo apt update
sudo apt install zsh
sudo apt install git
```

