---
title: CentOS7 调整 lvm 分区大小
date: 2019-07-22 16:06:03
categories: Linux
tags: [Linux, Server, CentOS]
---

Centos7 系统在安装时，会默认使用 lvm 创建三个逻辑卷

| Volume| FS | Mount Point | Size |
| :------ | ----: | :------ | ---: |
| /dev/centos/home | xfs | /home | 157.20 GB |
| /dev/centos/root | xfs | /root | 50.00 GB |
| /dev/centos/swap |  |  | 4.00 GB |

默认分配给挂载点 /home 的空间大小较大，而由于在集群中，用户的目录都在存储阵列上，并且 root 账户拥有 /root 目录，因此 /home 目录几乎不会使用到，因此需要在安装时调整默认的分区大小

同时我们可以看到，交换空间分区 swap 的大小只有 4G， 可能是安装时考虑到节点内存较大 (128GB)，因此默认没有更大的分配交换空间。然而经过实际使用证明，swap 分区的大小是远远不够的，很多软件会在使用内存的同时使用swap空间，并且当交换空间占满后，系统的IO性能会有所降低。

由于之前节点重装时，没有考虑到以上两点，因此现在需要对 lvm 分区大小进行调整。所以我准备使用 lvm 管理命令结合 system-storage-manager 软件，实现对 /home 分区空间的缩减，并重新分配到 Swap 交换空间以及 / 根目录中

<!-- more -->

***在执行下文所述操作之前，需要确定节点上没有正在运行的任务***

## 准备工作

#### 先将节点下线，防止有任务出现

```
pbsnodes -o node5
```

#### 下载 system-storage-manager

```
yumdownloader --resolve --downloadonly --downloaddir=/dir/to/download system-storage-manager-0.4-8.el7.noarch.rpm
```

#### 安装 system-storage-manager

```
rpm -ivh system-storage-manager-0.4-8.el7.noarch.rpm
```

```
警告：system-storage-manager-0.4-8.el7.noarch.rpm: 头V3 RSA/SHA256 Signature, 密钥 ID f4a80eb5: NOKEY
准备中...                          ################################# [100%]
正在升级/安装...
   1:system-storage-manager-0.4-8.el7 ################################# [100%]
```

#### 关闭 SWAP 分区

```
swapoff -a
```

#### 卸载 /home 分区

***如果 /home 分区内有用户目录，则需要先备份到其他位置，等调整完毕后，拷贝回 /home 目录即可***

```
umount /home
```

#### 使用 `system-storage-manager` 查看 lvm 分区情况
ssm list

```
----------------------------------------------------------------
Device           Free       Used      Total  Pool    Mount point
----------------------------------------------------------------
/dev/loop0                        100.00 GB
/dev/loop1                          2.00 GB
/dev/md126p1                      200.00 MB          /boot/efi
/dev/md126p2                        1.00 GB          /boot
/dev/md126p3  0.00 KB  211.20 GB  211.20 GB  centos
/dev/sda      0.00 KB  212.40 GB  223.57 GB
/dev/sdb      0.00 KB  212.40 GB  223.57 GB
----------------------------------------------------------------
----------------------------------------------------
Pool    Type  Devices     Free       Used      Total
----------------------------------------------------
centos  lvm   1        0.00 KB  211.20 GB  211.20 GB
----------------------------------------------------
---------------------------------------------------------------------------------------
Volume            Pool    Volume size  FS       FS size       Free  Type    Mount point
---------------------------------------------------------------------------------------
/dev/centos/root  centos     50.00 GB  xfs     49.98 GB   48.67 GB  linear  /
/dev/centos/swap  centos      4.00 GB                               linear
/dev/centos/home  centos    157.20 GB  xfs    157.12 GB  157.12 GB  linear  /home
/dev/md126        md        212.40 GB                               raid1
/dev/md126p1                200.00 MB  vfat                                 /boot/efi
/dev/md126p2                  1.00 GB  xfs   1014.00 MB  875.35 MB          /boot
---------------------------------------------------------------------------------------
```

## 释放 /home 空间

#### 删除 /dev/centos/home 逻辑卷

```
ssm remove /dev/centos/home
```

```
Do you really want to remove active logical volume centos/home? [y/n]: y
  Logical volume "home" successfully removed
```

## 增加 SWAP 空间容量

#### 增加 /dev/centos/swap 逻辑卷容量

```
ssm resize -s +28G /dev/centos/swap
```

```
  Size of logical volume centos/swap changed from 4.00 GiB (1024 extents) to 32.00 GiB (8192 extents).
  Logical volume centos/swap successfully resized.
```

#### 重设 SWAP 空间大小

```
mkswap /dev/centos/swap
```

```
mkswap: /dev/centos/swap: warning: wiping old swap signature.
正在设置交换空间版本 1，大小 = 33554428 KiB
无标签，UUID=f235320f-7bfd-4f58-8a0c-15dddb935624
```

#### 启用 SWAP

```
swapon -a
```

#### 查看 SWAP 空间容量

```
free -m
```

```
              total        used        free      shared  buff/cache   available
Mem:         128652        2411      124346          26        1894      125328
Swap:         32767           0       32767
```

## 增加根目录 / 容量

#### 增加 /dev/centos/root 逻辑卷容量

```
lvresize -l +100%FREE /dev/centos/root
```

```
  Size of logical volume centos/root changed from 50.00 GiB (12800 extents) to <179.20 GiB (45875 extents).
  Logical volume centos/root successfully resized.
```

#### / 使用的 XFS 文件系统，需要手动扩容，否则分配的容量仍然无法使用

```
xfs_growfs /dev/centos/root
```

```
meta-data=/dev/mapper/centos-root isize=512    agcount=4, agsize=3276800 blks
         =                       sectsz=4096  attr=2, projid32bit=1
         =                       crc=1        finobt=0 spinodes=0
data     =                       bsize=4096   blocks=13107200, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal               bsize=4096   blocks=6400, version=2
         =                       sectsz=4096  sunit=1 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 13107200 to 46976000
```

## 修改其他配置

#### 调整默认挂载选项

编辑 `/etc/fstab`，删除文件内这一行

```
/dev/mapper/centos-home /home                   xfs     defaults        0 0
```

#### 测试挂载文件内容是否正确

```
mount -a
```

#### 节点上线

```
pbsnodes -c node5
```
___

> 参考资料
> [1]. https://blog.csdn.net/shenyue_sam/article/details/77175637
> [2]. https://blog.csdn.net/yexiangCSDN/article/details/83182259