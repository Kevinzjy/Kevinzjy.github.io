---
title: 服务器计算节点重装 CentOS7
date: 2019-06-05 22:51:19
categories: Linux
tags: [Linux, Server, CentOS]
---

由于计算需求, 服务器需要安装 CentOS7,正好有一些节点 down 掉之后系统无法正常进入, 趁机重装一下, 单独分一个队列出来

<!-- more -->

### 安装前准备

#### 在 Linux 下制作 CentOS7 安装盘

```bash
sudo dd if=/home/zhangjy/Data/resources/linux/CentOS-7-x86_64-DVD-1810.iso of=/dev/sda
```

### 机房操作

#### 给安装完成的 CentOS 配置网络

网卡配置 /etc/sysconfig/network-scripts

```
DEVICE=enp4s0f0
BOOTPROTO=static
ONBOOT=no
TYPE=Ethernet
```

```
DEVICE=enp4s0f1
BOOTPROTO=none
ONBOOT=yes
TYPE=Ethernet
IPV6INIT=no
IPADDR=11.11.11.3
NETMASK=255.255.0.0
```

```
TYPE=Ethernet
BOOTPROTO=static
ONBOOT=yes
IPADDR=192.168.0.143
NETMASK=255.255.255.0
```

### 管理节点操作

#### 设置host

```bash
rsync -avP /etc/hosts root@node143:/etc/hosts
```

#### 设置ssh免密码登录

```bash
rsync -avP /root/.ssh root@node143 /
ssh-copy-id root@node143
ssh node143
ssh-copy-id root@node143
```

#### 设置中文环境

修改 /etc/profile

```bash
export LANG="zh_CN.UTF-8"
epxort LC_ALL="zh_CN.UTF-8"
```

#### 配置 Deploy_tool
```bash
cd /public/sourcecode/Gridview3.2_Release/setup/deploy_tool
```

/etc/ssh/sshd_config
```
PermitRootLogin no
service sshd restart
```

#### 在可以连接外网的节点上下载所需安装包

```
[root@manager ~]#  yum install --downloadonly --downloaddir=/histor/software/yum_download pam-devel zlib-devel readline-devel openssl-devel libxml2-devel dos2unix libgcc glibc
```

#### 修改节点列表
/opt/node_list

```
node143
```

#### 安装采集计算节点和任务调度系统

```bash
cd /opt/gridview/collect_agent
sh ./install_jobscheduler_node.sh 192.168.0.78 /opt/node_list

cd /opt/gridview/jobscheduler
./batch_install_collect_node.sh 192.168.0.78 /opt/node_list
```

```bash
qmgr -c "create queue dev queue_type=execution"
qmgr -c "set queue dev started=true"
qmgr -c "set queue dev enabled=true"
qmgr -c "set queue dev resources_default.nodes=1"
qmgr -c "set queue dev resources_default.walltime=100000"

qmgr -c "create node node143"
qmgr -c "set node node143 np=28"

qmgr -c "set queue dev acl_groups=zhaolab"
qmgr -c "set queue dev acl_group_sloppy=true"
qmgr -c "set queue dev acl_hosts=node143"

```

/opt/gridview/pbs/dispatcher/server_priv/nodes
```
node143 np=28
```

#### 配置NIS

配置 manager 上 /etc/sysconfig/network, (需要重启)

```
HOSTNAME=manager
NETWORKING=yes
NISDOMAIN=TS10K
```

临时生效: `nisdomainname TS10K`

配置: /etc/ypserv.conf

```
# Host                     : Domain  : Map              : Security
#
# *                        : *       : passwd.byname    : port
# *                        : *       : passwd.byuid     : port
127.0.0.1/255.0.0.0        : *       : *                : none
192.168.0.0/255.255.255.0  : *       : *                : none
*                          : *       : *                : deny
```

```
service ypserv start
```

```
[root@manager yp]# /usr/lib64/yp/ypinit -m

At this point, we have to construct a list of the hosts which will run NIS
servers.  manager is in the list of NIS server hosts.  Please continue to add
the names for the other hosts, one per line.  When you are done with the
list, type a <control D>.
	next host to add:  manager
	next host to add:
The current list of NIS servers looks like this:

manager

Is this correct?  [y/n: y]  y
We need a few minutes to build the databases...
Building /var/yp/TS10K/ypservers...
Running /var/yp/Makefile...
gmake[1]: Entering directory `/var/yp/TS10K'
Updating passwd.byname...
failed to send 'clear' to local ypserv: RPC: Program not registeredUpdating passwd.byuid...
failed to send 'clear' to local ypserv: RPC: Program not registeredUpdating group.byname...
failed to send 'clear' to local ypserv: RPC: Program not registeredUpdating group.bygid...
failed to send 'clear' to local ypserv: RPC: Program not registeredUpdating hosts.byname...
failed to send 'clear' to local ypserv: RPC: Program not registeredUpdating hosts.byaddr...
failed to send 'clear' to local ypserv: RPC: Program not registeredUpdating rpc.byname...
failed to send 'clear' to local ypserv: RPC: Program not registeredUpdating rpc.bynumber...
failed to send 'clear' to local ypserv: RPC: Program not registeredUpdating services.byname...
failed to send 'clear' to local ypserv: RPC: Program not registeredUpdating services.byservicename...
failed to send 'clear' to local ypserv: RPC: Program not registeredUpdating netid.byname...
failed to send 'clear' to local ypserv: RPC: Program not registeredUpdating protocols.bynumber...
failed to send 'clear' to local ypserv: RPC: Program not registeredUpdating protocols.byname...
failed to send 'clear' to local ypserv: RPC: Program not registeredUpdating mail.aliases...
failed to send 'clear' to local ypserv: RPC: Program not registeredgmake[1]: Leaving directory `/var/yp/TS10K'

manager has been set up as a NIS master server.

Now you can run ypinit -s manager on all slave server.
```

计算节点

```
[root@node143 ~]# cat /etc/sysconfig/network
HOSTNAME=node143
NETWORKING=yes
NISDOMAIN=TS10K
```

/etc/nsswitch.conf
```
passwd:     files nis
shadow:     files nis
group:      files nis
hosts:      files nis dns
```

NFS 挂载
```
mount -t nfs -o vers=3 192.168.0.226:/. /astor/
```

***

> 参考资料:
> [1]. https://blog.51cto.com/yejiankang/885484