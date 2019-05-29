---
title: 服务器使用 NFS 挂载移动硬盘
date: 2019-05-29 14:30:34
categories: Linux
tags: [Linux, Server, CentOS]
---

最近在服务器上, 经常需要挂载硬盘进行数据拷贝的工作. 然而由于 home 目录 USB 接口数量有限,每次只能通过将硬盘插入计算节点的方式进行挂载. 因此每次需要使用 qsub 提交交互式任务到计算节点上进行数据拷贝, 十分不方便. 因此打算使用 NFS 的方法, 把计算节点上挂载的数据目录挂载在主节点, 进行数据拷贝的工作.

<!-- more -->

***由于服务器已经部署了 NFS, 因此不记录 NFS 和 rpcbind 服务的安装和配置过程***

#### 启动 NFS 和 rpcbind 服务

```
/etc/init.d/rpcbind start
/etc/init.d/nfs start
```

#### 在计算节点挂载移动硬盘

```
mount -t ntfs /dev/sdc1 /media/data
```

#### 将挂载文件夹加入 NFS 发布目录

```
vi /etc/exports
/media/data     192.168.0.0/24(rw,no_root_squash)
exportfs -r #使配置生效
exportfs -v #查看已生效配置
```
#### 在主目录上挂载数据文件夹

```
mount node150:/media/data /media/data
```

#### 卸载硬盘

```
# On PBS manager
umount /media/data

# On node150
echo '' > /etc/exports # clear NFS export
exportfs -r
exportfs -v
umount /media/data
```

***

> 参考资料:
> [1]. https://blog.51cto.com/yejiankang/885484