---
title: Ubuntu Server 18.04 LTS 安装
date: 2019-03-04 14:22:57
categories: Ubuntu
tags: [Ubuntu, ]
---

实验室使用的 2011-mid 21.5 iMac 实在太卡，于是找老板更换了新电脑，打算装 Ubuntu 18.04 LTS，这样方便以后学习 ML，

<!-- more -->

#### 为何选择 Ubuntu Server 而不是 Desktop 版本

安装 Ubuntu server 的好处：可以使用 LVM 分区，方法分区大小的调整，在 Ubuntu Desktop 版本中，自动安装的 LVM 分区只能使用默认大小，无法自由设置

#### Ubuntu Server 的安装准备

从 [Canonical 官网](https://www.ubuntu.com/download/server) 下载 Ubuntu server 镜像，使用 [Rufus](https://rufus.ie/) 刻录到U盘作为启动盘

#### 系统分区方案

主机硬盘配置: i7 8700 / 16G / GTX1060 6G，512G N.vMe SSD + 3T HDD

为了保证系统速度，以及为了不会因为数据问题重新分区，采用了下列的分区方案

| MOUNT POINT | SIZE | TYPE | DEVICE TYPE |
| ------ | ----: | ------ | -----|
| / | 443.433G| ext4 | LVM logical volume (SSD)|
| /boot | 1.000G | ext4 | partition of local disk (SSD)|
| /boot/efi | 512.000M| fat32| partition of local disk (SSD) |
| /home| 2.729T| ext4| Spartition of local disk (HDD) |
| SWAP | 32.000G | swap | LVM logical volume (SSD)|

**在默认的 LVM 分区中，/ 根分区大小只有 4G，在SSD上还剩下400多G的空闲空间，可以分配给想要的分区 ( / 和 swap )**

#### 修改swap分区大小 (适用于Desktop环境)

在 Server 版本安装过程中，可以直接设定LVM分区的大小，不需要经过这一步

```
sudo swapon --show
free -h
df -lh
sudo lvs
ls /dev/mapper

swapoff -a # 关闭swap分区
lvreduce -L -31G /dev/mapper/ubuntu--vg-root # 根目录缩减空间
lvextend -L +31G /dev/mapper/ubuntu--vg-swap_1 # 将空闲空间分配给swap
swapon -a # 重新打开swap
```

**缩减根目录空间可能导致数据丢失，建议还是直接使用 Server 版本进行安装**

#### 网络连接配置

Ubuntu 18.04 LTS 中，默认使用 netplan 进行网络配置，然而在安装时，先暂时使用 ifconfig 命令配置联网，用来安装后续需要的软件

```bash
ifconfig -a # 列出网卡

ifconfig enp4s0 down # 关闭有线网卡
ifconfig enp4s0 hw ether 98:ee:cb:95:c9:0e # 更改硬件地址
ifconfig enp4s0 up # 重新打开有线网卡
ping www.baidu.com
```

在系统安装过程中，发现默认的网卡配置连不上网，可能上级路由配置问题，更改硬件 MAC 地址后，会重新分配到新 IP，就可以成功联网了

#### 安装 Grub 引导分区

```bash
fdisk -l
mkdir /mnt
mount /dev/ubuntu-vg/ubuntu-lv /mnt
mkdir /mnt/boot/efi
mount /dev/nvme0n1p1 /boot/efi
for i in /dev/ /dev/pts /proc /sys; do mount -B $i /mnt$i; done

apt install grub-efi-amd64
grub-install --no-nvram --root-directory=/mnt /dev/nvme0n1p1

sudo chroot /mnt
sudo update-grub
```

#### 进入 Grub 引导模式，配置UEFI启动

```
grub rescue > ls
ls (hd1,gpt2)/
set root=(hd1,gpt2)
insmod linux
linux (hd1,gpt2)/vmlinuz-4.15.0-29-generic
```

UEFI启动，开机时按ESC进入grub模式

```
mount -o rw,remount /
ls /home/

passwd root
reboot
```

在 grub 模式中挂载好分区，然后更改 root 密码，就可以成功进入系统了

#### 网络配置

配置静态ip地址，网关和DNS

```/etc/netplan/config.yaml
network:
    version: 2
    renderer: NetworkManager
    ethernets:
        enp4s0:
            addresses:
            - 10.4.0.140/24
            gateway4: 10.4.0.1
            nameservers:
                addresses: [8.8.8.8, 8.8.4.4]
                search: []
            optional: true
```

随机设置网卡 MAC 地址

```
ifconfig enp4s0 up
ifconfig enp4s0 10.4.0.246 netmask 255.255.255.0 broadcast 10.4.0.255
ip route add default via 10.4.0.1
openssl rand -hex 6 | sed 's/\(..\)/\1/g; s/.$//' | xargs ifconfig enp4s0 hw ether
```

安装无线网卡管理和桌面环境

```
apt install xfce4
apt install wicd
startx
```

#### 添加自己的用户

```
adduser zhangjy
vim /etc/sudoers # 在 sudoers 中，为账户配置 root 权限
zhangjy ALL=(ALL) ALL
```
