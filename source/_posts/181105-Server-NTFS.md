---
title: Linux 服务器挂载 NTFS / exFAT 格式硬盘
date: 2018-11-5 20:16:40
categories: Linux
tags: [Server, Linux, ]
---

Linux 下默认的硬盘格式为 ext4 / ext3，因此如果想要挂载平时更加常用的 NTFS / exFAT 格式移动硬盘，需要额外安装驱动

安装 NTFS / exFAT 支持需要**root权限**，服务器请联系管理员解决...

<!-- more -->

#### 系统环境

通过 `lsb_release -a` 命令查看当前系统版本

```bash
$ lsb_release -a
LSB Version:    :core-4.0-amd64:core-4.0-noarch:graphics-4.0-amd64:graphics-4.0-noarch:printing-4.0-amd64:printing-4.0-noarch
Distributor ID: RedHatEnterpriseServer
Description:    Red Hat Enterprise Linux Server release 6.3 (Santiago)
Release:        6.3
Codename:       Santiago
```


#### 下载驱动程序

下载 `ntfs-3g`, `fuse`, `fuse-exfat` 和 `exfat-utils` 的 RPM 安装包

```bash
$ wget http://download1.rpmfusion.org/free/el/updates/6/x86_64/exfat-utils-1.3.0-1.el6.x86_64.rpm
$ wget http://download1.rpmfusion.org/free/el/updates/6/x86_64/fuse-exfat-1.3.0-1.el6.x86_64.rpm
$ wget https://www.rpmfind.net/linux/epel/6/x86_64/Packages/n/ntfs-3g-2017.3.23-6.el6.x86_64.rpm
$ wget https://www.rpmfind.net/linux/centos/6.10/os/x86_64/Packages/fuse-2.8.3-5.el6.x86_64.rpm
```

#### 安装

注意**安装顺序**，先安装 `fuse-exfat` 再安装 `exfat-utils`

```bash
$ rpm -Uvh ntfs-3g-2017.3.23-6.el6.x86_64.rpm
$ rpm -Uvh fuse-2.8.3-5.el6.x86_64.rpm
$ rpm -Uvh fuse-exfat-1.3.0-1.el6.x86_64.rpm
$ rpm -Uvh exfat-utils-1.3.0-1.el6.x86_64.rpm
```

#### 出现的问题

安装过程中如果提示

```bash
/sbin/mount.ntfs-3g: symbol lookup error: /sbin/mount.ntfs-3g: undefined symbol: ntfs_xattr_build_mapping
```

`libntfs-3g` 动态库版本过低，可以通过 `ldd` 命令查看 `ntfs-3g` 依赖的动态库版本

```bash
$ ldd /bin/ntfs-3g
        linux-vdso.so.1 =>  (0x00007fffac8eb000)
        libdl.so.2 => /lib64/libdl.so.2 (0x00002ace10591000)
        libpthread.so.0 => /lib64/libpthread.so.0 (0x00002ace10796000)
        libntfs-3g.so.88 => /lib/libntfs-3g.so.88 (0x00002ad751769000)
        libc.so.6 => /lib64/libc.so.6 (0x00002ace10c02000)
        /lib64/ld-linux-x86-64.so.2 (0x00002ace1036e000)
```

我们刚刚安装的动态库位于 `/lib64/libntfs-3g.so.88`，而系统调用的仍然是 `/lib/libntfs-3g.so.88` 的老版本库，因此将 `/lib/libntfs-3g.so` 更名让程序无法调用即可

```bash
$ cd /lib
$ mv libntfs-3g.so.88 libntfs-3g.so.88.unwanted

$ ldd $(which ntfs-3g)
        linux-vdso.so.1 =>  (0x00007fffac8eb000)
        libdl.so.2 => /lib64/libdl.so.2 (0x00002ace10591000)
        libpthread.so.0 => /lib64/libpthread.so.0 (0x00002ace10796000)
        libntfs-3g.so.88 => /lib64/libntfs-3g.so.88 (0x00002ace109b3000)
        libc.so.6 => /lib64/libc.so.6 (0x00002ace10c02000)
        /lib64/ld-linux-x86-64.so.2 (0x00002ace1036e000)
```

可以看到 `ntfs-3g` 依赖的动态库已经指定为我们新安装的版本


#### 挂载

首先通过 `fdisk -l` 命令确定需要挂载的分区，例如 `/dev/sdc2`

创建挂载点文件夹
```bash
$ mkdir /NAS
```

将分区挂载到指定的挂载点

```bash
# NTFS 格式
$ mount -t ntfs-3g /dev/sdc2 /NAS

# exFAT 格式
$ mount.exfat /dev/sdc2 /NAS
```

#### 卸载硬盘

```
umount -l /NAS
```

***

> 参考链接：
> [1]. http://docs.gz.ro/node/162