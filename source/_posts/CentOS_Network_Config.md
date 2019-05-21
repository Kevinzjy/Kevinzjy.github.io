---
title: CentOS 配置网络环境
date: 2019-05-21 15:49:53
categories: Linux
tags: [Linux, Server, CentOS]
---

在 CentOS 安装过程中,如果选择了 minimal 安装,默认是没有安装 net-tools 的,因此 iwconfig 等命令无法正常使用,配置网络环境十分麻烦,查阅了很多资料,终于发现其实自带了一个命令行的网络管理工具 `nmtui`

使用 `nmtui` 配置好网络链接后, 输入 `service network start` 就可以正常使用网络了,然后再使用 `yum install net-tools` 安装网络工具之后,常用的命令就都能用了