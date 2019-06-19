---
title: CentOS7 配置 Torque + MAUI 作业调度系统
date: 2019-06-14 12:01:20
categories: Linux
tags: [Linux, Server, CentOS]
---

前两天曙光的 Gridview 管理系统配置出现了问题，导致管理节点无法正常使用。因此重新配置了一套 Torque 管理系统，正好一起配置新重装的 CentOS7 节点

<!-- more -->


### 服务器节点系统重装

#### CentOS 7.6 镜像位置

```
node71:/root/packages/CentOS-7-x86_64-DVD-1810.iso
```

#### 在 Linux 下制作 CentOS7 安装盘

在 node71 上插入安装使用的U盘或者移动硬盘，根据盘符将安装镜像写入设备

```bash
# 将系统安装镜像写入 /dev/sdc
sudo dd if=/root/packages/CentOS-7-x86_64-DVD-1810.iso of=/dev/sdc
```

在需要安装的节点上，插入安装盘。重启节点即可进入安装界面。

#### 给安装完成的 CentOS 配置网络

网卡配置文件在 /etc/sysconfig/network-scripts 文件夹，根据其他机器的配置，修改配置文件，一般为 ifcfg-eth 或者 ifcfg-enp 开头，对应不同的网卡设备名称

ifcfg-enp4s0f0
```
DEVICE=enp4s0f0
BOOTPROTO=static
ONBOOT=no
TYPE=Ethernet
```

ifcfg-enp4s0f1
```
DEVICE=enp4s0f1
BOOTPROTO=none
ONBOOT=yes
TYPE=Ethernet
IPV6INIT=no
IPADDR=11.11.11.3
NETMASK=255.255.0.0
```

ifcfg-eth0
```
TYPE=Ethernet
BOOTPROTO=static
ONBOOT=yes
IPADDR=192.168.0.143
NETMASK=255.255.255.0
```

修改完网卡配置后，重启网络服务，即可通过 ssh 登录计算节点进行后续的配置

```bash
service network restart
```

### 计算节点基础配置

#### 设置 host

将登录节点的 /etc/hosts 文件同步

```bash
# 登录节点进行操作
scp /etc/hosts root@node111:/etc/hosts
```

#### 设置 ssh 免密码登录

方法一： 可以复制登录节点的ssh配置文件，实现节点之间的登录

```bash
# 登录节点进行操作
rsync -avP /root/.ssh/* root@node143:/root/.ssh
```

方法二： 重新配置 ssh

```bash
# 登录节点进行操作
# 如果登录节点也是重新安装的系统，需要先生成 ssh 秘钥
ssh-keygen

# 将登录节点的秘钥配置上传到 node111 中
ssh-copyid node111
```

#### 设置中文支持

修改 `/etc/profile`

```bash
export LANG="zh_CN.UTF-8"
export LC_ALL="zh_CN.UTF-8"
```

#### 限制ssh 登录

目前采用的最简单的方法，直接从 sshd 服务配置上对用户进行限制

配置文件 `/etc/ssh/sshd_config` 根据需求增加如下内容


计算节点

```
AllowUsers root # 只允许 root 账户登录
```

管理节点

```
PermitRootLogin no # 禁止 root 账户登录
```

### 安装和配置存储服务

#### 在管理节点下载软件安装包

常用软件已经下载到 `/root/download`

```bash
# 将 git 和其依赖的软件包下载到指定文件夹
yumdownloader --destdir=/histor/public/rpm/git git

# 在指定节点批量安装软件
ssh root@node111 "rpm -ivh /histor/public/rpm/git/*.rpm"
```

通过shell脚本循环批量安装

```bash
# 在 node140-node160 上批量指定命令
for i in `seq 140 160`
do
	ssh root@node${i} "rpm -ivh /histor/public/rpm/git/*.rpm"
done
```

#### astor 挂载

目前没有使用 `icfs-fuse` 客户端，而是直接使用了 `nfs` 格式进行挂载，对于新装的系统，需要下载 `nfs-utils` 软件

编辑 `/etc/rc.local` 文件

```
mount -t nfs -o vers=3 192.168.0.226:/. /astor/
```

赋予执行权限，防止开机不能自动运行
```
chmod +x /etc/rc.d/rc.local
```

#### histor 挂载

驱动程序位于 `/root/Leo-xd-clt-as7.6-64-20180731`，挂载命令为 `/root/Leo-xd-clt-as7.6-64-20180731/LeoFS.192.168.0.245`

```bash
# 同步驱动
rsync -avP /root/Leo-xd-clt-as7.6-64-20180731 node111:/root/Leo-xd-clt-as7.6-64-20180731

# 在计算节点
cp -r /root/Leo-xd-clt-as7.6-64-20180731/LeoCluster /LeoCluster

# 配置自启动挂载
cp /root/Leo-xd-clt-as7.6-64-20180731/LeoFS.192.168.0.245 /etc/init.d/LeoFS.192.168.0.245
chkconfig LeoFS.192.168.0.245 start
```

#### NTFS 硬盘挂载

需要下载 `ntfs-3g` 和 `ntfsprogs` 软件

```
mkdir zhaotmp
mount -t ntfs /zhaotmp /dev/sdc1
```

#### 用户信息配置

用户配置文件包括四个:

```
/etc/passwd
/etc/group
/etc/shadow
/etc/gshadow
```

将四个文件中需要配置的内容拷贝到节点上对应的文件内即可

### PBS的安装和配置

#### 计算节点 PBS 安装

`torque` 安装文件位于 `/root/torque-4.2.9`

```
cd /root/torque-4.2.9

# 在计算节点创建安装文件夹
ssh node111 "mkdir -p /root/torque4"

# 拷贝安装文件到计算节点
scp torque-package-{mom,clients}-linux-x86_64.sh node111:/root/torque4
scp contrib/init.d/{pbs_mom,trqauthd} node1:/etc/init.d/

# 计算节点安装客户端
./torque-package-clients-linux-x86_64.sh --install  
./torque-package-mom-linux-x86_64.sh --install  

# 配置环境，编辑/var/spool/torque/mom_priv/config
$pbsserver node71
$logevent 225
$spool_as_final_name true # 日志直接保存到最终位置

# 计算节点启动服务
for i in pbs_mom trqauthd; do service $i start; done

# 配置开机启动
chkconfig pbs_mom on
chkconfig trqauthd on
```

将 torque 文件夹加入系统环境变量

编辑 `/etc/profile`，在文件末尾加入两行

```bash
TORQUE=/usr/local/torque-4.2.9
export PATH=$PATH:/usr/local/torque-4.2.9/bin:/usr/local/torque-4.2.9/sbin
```

#### 在管理节点增加新节点

配置文件位置 `/var/spool/torque/server_priv`

```
#指定节点名称，可用核心数，分组名称
node111 np=28 GPUfat
```

重启 PBS_server 使得配置信息生效

```
service pbs_server restart
```

可以使用 `pbsnodes` 命令查看已经配置好的节点列表

```
# 列出所有节点信息
pbsnodes

# 单独查看 node111
pbsnodes node111
```

```text
node111
     state = free
     np = 28
     properties = GPUfat
     ntype = cluster
     jobs = 0/504.node71
     status = rectime=1560483848,varattr=,jobs=504.node71,state=free,netload=783666799,gres=,loadave=0.04,ncpus=28,physmem=794154304kb,availmem=821029292kb,totmem=826922296kb,idletime=50234,nusers=1,nsessions=1,sessions=11595,uname=Linux node111 2.6.32-431.el6.x86_64 #1 SMP Sun Nov 10 22:19:54 EST 2013 x86_64,opsys=linux
     mom_service_port = 15002
     mom_manager_port = 15003
```

#### 管理节点增加队列

使用 `qmgr` 命令对队列信息进行配置

```bash
qmgr -c 'p s' # 输出当前队列配置
qmgr -c 'create queue middle' # 新增队列middle
qmgr -c 'set queue middle queue_type = Execution'
qmgr -c 'set queue middle resources_default.neednodes = middle' # 给队列分配分组为 middle 的节点
qmgr -c 'set queue middle resources_default.walltime = 1200:00:00' # 默认 walltime
qmgr -c 'set queue middle enabled = True' # 启用队列
qmgr -c 'set queue middle started = True' # 队列可以排队

qmgr -c "set queue middle acl_groups=zhaolab" # 限制特定用户组可以使用队列
qmgr -c "set queue middle acl_group_sloppy=true" # false 只查询用户的默认用户组; true: 查询用户所属的所有用户组
qmgr -c "set queue middle acl_hosts=node71" # 限制可以投递任务的节点
```

其他设置可以参考 http://docs.adaptivecomputing.com/torque/3-0-5/4.1queueconfig.php

#### MAUI 修改用户资源限额

配置文件地址 `/usr/local/maui/maui.cfg`

```
# 默认用户限制
USERCFG[DEFAULT]  MAXJOB=40  MAXPROC=80  MAXNODE=20

# 对 jipfnew 账户单独进行设置
USERCFG[jipfnew]  MAXJOB=100 MAXPROC=300 MAXNODE=20
```

修改完配置后，重启 MAUI 服务使配置生效

```
service maui restart
```

### 安全设置

#### 管理节点禁止其他用户切换到 root 账户

添加管理员到 wheel 用户组

```
usermod -G wheel admin
id admin
```

使用 `id` 命令可以查看 admin 账户的 id 和 group 信息

```
uid=1000(admin) gid=1000(admin) groups=1000(admin),10(wheel)
```

修改 /etc/pam.d/su，将这行的注释去掉，使得只有 wheel 组的账户可以使用 su 命令切换到 root 账户

```
auth		required	pam_wheel.so use_uid
```

#### 管理节点禁止其他用户登录

编辑PAM配置文件 `/etc/security/access.conf`

```bash
+ : root : 192.168.0. 11.11.11. # 允许内网 root 账户登录
- : root : ALL # 禁止其他节点 root 账户登录
- : ALL  : 192.168.0. # 禁止内网其他用户登录
```

#### Firewall-cmd 防火墙配置

通过查看 `/var/log/secure` 文件，发现有几个ip在大量尝试暴力破解root密码，因此决定增加防火墙防止被攻击

```bash
# 安装防火墙服务
yum install firewalld

# 查看防火墙状态
firewall-cmd --state

# 设置开机自动启动
systemctl enable firewalld

# 开启防火墙
systemctl start firewalld
```

由于 Torque + MAUI 的配置中，管理节点需要和计算节点进行通信，因此需要开放几个端口

```bash
firewall-cmd --add-port=15001/tcp --permanent
firewall-cmd --add-port=15002/tcp --permanent
firewall-cmd --add-port=15003/tcp --permanent
firewall-cmd --add-port=42559/tcp --permanent
firewall-cmd --add-port=8080/tcp --permanent

# 这样配置比较麻烦，设置直接信任内网网卡
firewall-cmd --zone=trusted --change-interface=enp1s0 --permanent

# 重启防火墙使得配置生效
firewall-cmd --reload
```

对于之前发现的ip，进行了手动封禁ip

```bash
# 查看尝试次数比较多的ip列表
grep "Failed password" /var/log/secure | awk '{print $11}' | sort | uniq -c | sort -n -r -k 1 | head

# 手动封禁
firewall-cmd --permanent --add-rich-rule="rule family='ipv4' source address='120.133.22.121' reject"
```

如果需要解封，可以编辑 `/etc/firewalld/zones/public.xml` 配置文件，删除ip对应的字段

```xml
  <rule family="ipv4">
    <source address="178.128.149.100"/>
    <reject/>
  </rule>
```

#### Fail2ban 自动ban ip

手动封禁还是比较麻烦的，因此使用了 Fail2ban 对短时间大量尝试的ip进行自动封禁

```bash
# 安装服务与开机自启
yum install fail2ban
systemctl enable fail2ban
```

配置文件位于 `/etc/fail2ban/jail.local`

```
[DEFAULT]
ignoreip  = 127.0.0.1/8 172.16.0.0/8 127.0.0.1 10.0.0.0/8 192.168.0.0/16 1.2.3.4
bantime   = 86400
findtime  = 600
maxretry  = 10
banaction = firewallcmd-ipset
action    = %(action_mwl)s

[sshd]
enabled = true
filter  = sshd
port    = 22
action  = %(action_mwl)s
logpath = /var/log/secure
```

这样配置后，可以禁止在10分钟内 ssh 错误超过10次的ip继续尝试登录

```bash
# 启动服务
systemctl start fail2ban

# 查看目前运行状态，可以看到被ban的ip
fail2ban-client status sshd

# 解封ip
fail2ban-client set sshd unbanip 172.16.100.1
```

### 其他设置

#### 曙光节点重新配置

**经过测试，这样直接配置可以进行任务投递，但是任务结束时无法获取 Complete 的状态，可能是曙光使用的 Torque 版本号不同所导致，需要对于所有的计算节点重新配置版本号对应的 Torque-client，*但是仍然需要关闭Gridview服务的自启***

~~曙光服务位置在 `/opt/gridview`~~

~~通过配置文件 `/opt/gridview/pbs/dispatcher/mom_priv/config` 可以为曙光节点重新指定 pbs 管理节点~~

```
$pbsserver node71
$restricted *.node71
```

~~修改成功后，重启 `pbs_mom` 服务~~

```bash
service pbs_mom restart
```

禁用 `gridview` 相关服务的自启动

```
# 查看所有自启动服务
chkconfig --list

# 关闭 gridview_platform 服务自启
chkconfig gridview_platform off

# 这两项服务是 pbs 需要的，不用关闭
chkconfig pbs_mom on
chkconfig trqauthd on
```
