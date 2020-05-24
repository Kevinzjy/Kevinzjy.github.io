---
title: Proxmox VE 环境安装
date: 2020-05-24 11:48:23
categories: Linux
tags: [Linux, Server]
---

这两天在家里尝试了下 Proxmox 的安装，记录一下需要注意的内容

安装过程参考: https://pve.proxmox.com/wiki/Installation，需要注意网络配置静态IP

<!-- more -->

#### 更换系统自带的 vim-tiny 为 vim

```
apt remove vim-tiny
apt install vim
```

#### 更换免费源

注释掉 /etc/apt/sources.list.d/pve-enterprise.list 中 enterprise 源内容

```txt
#deb https://enterprise.proxmox.com/debian/pve buster pve-enterprise
```

修改 /etc/apt/sources.list ，加入免费源

```
# PVE pve-no-subscription repository provided by proxmox.com,
# NOT recommended for production use
deb http://download.proxmox.com/debian/pve buster pve-no-subscription
```

更新源

```bash
apt update
apt upgrade
```

#### 去除付费提示

```bash
cd /usr/share/javascript/proxmox-widget-toolkit
cp proxmoxlib.js proxmoxlib.js.bak
vim proxmoxlib.js
```

将 data.status !== 'Active' 替换为 false

```
393c393
< 		if (data.status !== 'Active') {
---
> 		if (false) {
```

重启 PVE 网页服务

```
systemctl restart pveproxy
```

Comman + Shift + R 刷新页面缓存之后，订阅提示就会消失了

#### 其他设置

```bash
apt install ntp ntpdate
ntpdate -u ntp.aliyun.com
```

#### 虚拟机 ISO 文件存放位置

```
/var/lib/vz/template/iso
```
