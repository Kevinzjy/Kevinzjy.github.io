---
title: Ubuntu VPS上搭建Shadowsocks IPV6代理
date: 2017-05-19 14:43:36
categories: Linux
tags: [Linux, Ubuntu, VPS]
---

最近把DigitalOcean上的VPS的ssh服务搞挂了。。。重新配置了一个Droplet，记录一下配置过程

<!-- more -->

#### VPS的系统配置

+ 系统：Ubuntu 16.04 LTS
+ 内存: 512 MB Memory
+ 硬盘: 20 GB Disk

#### Step1: 建立Droplet

在DigitalOcean上面新建一个Droplet

#### Step2: 登录VPS进行配置

修改Root密码

```bash
ssh root@vps_ip # 登录VPS
sudo passwd root #修改ROOT密码
```

> Enter new UNIX password: # 输入新密码
> 
> Retype new UNIX password: # 再次输入
> 
> passwd: password updated successfully

修改时区

```bash
dpkg-reconfigure tzdata #修改时区，选择Asia/Shanghai
```

> Current default time zone: 'Asia/Shanghai'
> 
> Local time is now:      Fri May 19 15:19:57 CST 2017.
> 
> Universal Time is now:  Fri May 19 07:19:57 UTC 2017.

更新APT源

```
apt update # 更新apt源
apt upgrade # 升级
```

推荐新建一个具有Root权限的用户，方便VPS的管理和操作

```bash
sudo adduser zhangjy # 新建用户
sudo usermod -G admin zhangjy # 将用户添加到root
cp /root/.ssh /home/zhangjy # 复制登录秘钥，开启新用户的远程登录
```

下面的操作可以退出后使用新建的账户进行操作

#### Step3: 安装Shadowsocks服务

DigitalOcean上的VPS最大的作用就是可以同时连接ipv4和ipv6服务，在搭建Shadowsocks服务后，可以在校园网下用IPV6代理实现直接访问IPV4网址，不需要登录网关，同时避开了流量计费系统。

```bash
sudo intall python-pip
pip install --upgrade pip
sudo pip install shadowsocks
```

编辑`vim /etc/shadowsocks.json`，添加如下内容

```json
{
    "server": "::", # 监听IPV6地址
    "server_port": 18286, # 服务器端口
    "local_port": 1080,
    "password": "PASSWORD", # 密码
    "timeout": 600,
    "method": "aes-256-cfb"
}
```

编辑`vim /root/shadowsocks.sh`，内容

```bash
ssserver -c /etc/shadowsocks.json -d restart
```

配置每天重启

```bash
crontab -e 
# 选择[3]: vim.basic
# 最后一行加入
0 4 * * * /root/shadowsocks.sh >> /root/cron.log
```

重启Crontab，使定时任务生效

```
service crontab restart 
```

#### Step4: 其他的配置

安装常用的oh-my-zsh, htop, glances等软件，参考[Ubuntu软件推荐](https://kevinzjy.github.io/2017/05/13/Ubuntu-softwares/)

