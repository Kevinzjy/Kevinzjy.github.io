---
title: Bash on Ubuntu on Windows的安装和配置 (二) Ubuntu软件推荐
date: 2017-05-13 23:02:26
categories: Linux
tags: [Windows, Bash on Ubuntu, Software]
---

记录一些Ubuntu上工具的安装和使用

### Htop: 增强版的Top命令

```bash
sudo apt install htop
```

{% asset_img htop.png %}

### Glances: 全面监控你的系统状态

```bash
sudo pip install glances[action,browser,cloud,cpuinfo,chart,docker,export,folders,gpu,ip,raid,snmp,web,wifi]
```

<!-- more -->

### Axe: 好用的多线程下载工具

```bash
sudo apt install axel
axel -n 4 -o ./  url
```

### 附: 常用软件包在apt中的名称

| Description | Name  |
| :------------------- | :---------- |
| pip (for python 2.7)  | python-pip |
| pip (for python 3) | python3-pip |