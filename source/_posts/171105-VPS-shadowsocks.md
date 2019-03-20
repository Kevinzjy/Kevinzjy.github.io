---
title: Ubuntu VPS上搭建Shadowsocks IPV6代理
date: 2017-11-05 16:31:42
categories: Linux
tags: [Linux, Ubuntu, VPS]
---

最近把DigitalOcean上的VPS的ssh服务搞挂了。。。重新配置了一个Droplet，记录一下配置过程

更新: DigitalOcean因为下载盗版电影被封了。。。现在换了一个Vultr的

更新: Vultr VPS的IP被封了。。。国内电信IPV4环境下无法登陆，更换到阿里云美国节点ECS

<!-- more -->

#### VPS的系统配置

+ 系统：Ubuntu 16.04 LTS
+ 内存: 1024 MB Memory
+ 硬盘: 25 GB SSD
+ 流量：1000 GB
+ 费用: $5 / 月

#### Step1: 建立Server

~~在DigitalOcean上面新建一个Droplet~~
在Vultr中新建需要的server，这里选择的是日本的机房

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
# 在 /etc/sudoers 中添加root权限
# User privilege specification
zhangjy ALL=(ALL) ALL
```

下面的操作可以退出后使用新建的账户进行操作

#### Step3: 安装Shadowsocks服务

DigitalOcean上的VPS最大的作用就是可以同时连接ipv4和ipv6服务，在搭建Shadowsocks服务后，可以在校园网下用IPV6代理实现直接访问IPV4网址，不需要登录网关，同时避开了流量计费系统。

```bash
sudo apt intall python-pip
sudo pip install --upgrade pip
sudo pip install setuptools
#sudo pip install shadowsocks

sudo apt install libsodium-dev
pip install https://github.com/shadowsocks/shadowsocks/archive/master.zip -U
```

更新: 使用github上最新版本的shadowsocks，可以支持chacha20-ietf-poly1305加密方式，更加稳定

编辑`vim /etc/shadowsocks.json`，添加如下内容

```json
{
    "server": "::", # 监听IPV6地址
    "server_port": 443, # 使用443端口，更加“安全”
    "local_port": 1080,
    "password": "PASSWORD", # 密码
    "timeout": 600,
    "method": "aes-256-cfb"
}
```

在 openssl 版本更新后，需要修改一下 shadowsocks 的源代码

编辑 shadowsocks 中调用 openssl 的脚本
```
sudo vi /usr/local/lib/python2.7/dist-packages/shadowsocks/crypto/openssl.py
```

将 EVP_CIPHER_CTX_cleanup 替换为 EVP_CIPHER_CTX_reset

```
52c52
<     libcrypto.EVP_CIPHER_CTX_reset.argtypes = (c_void_p,)
---
>     libcrypto.EVP_CIPHER_CTX_cleanup.argtypes = (c_void_p,)
111c111
<             libcrypto.EVP_CIPHER_CTX_reset(self._ctx)
---
>             libcrypto.EVP_CIPHER_CTX_cleanup(self._ctx)
```

现在，可以手动开启shadowsocks服务了

```bash
ssserver -c /etc/shadowsocks.json -d restart
```

#### Step4: 配置shadowsocks服务开机自动启动

创建脚本 `/etc/init.d/shadowsocks`

```bash
#!/bin/sh
### BEGIN INIT INFO
# Provides:          shadowsocks
# Required-Start:    $remote_fs $syslog
# Required-Stop:     $remote_fs $syslog
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: start shadowsocks
# Description:       start shadowsocks
### END INIT INFO

ssstart(){
    /usr/local/bin/ssserver -c /etc/shadowsocks.json -d start
}

ssstop(){
    /usr/local/bin/ssserver-d stop
}

ssreload(){
    /usr/local/bin/ssserver -c /etc/shadowsocks.json -d restart
}

case "$1" in
start)
    ssstart
    ;;
stop)
    ssstop
    ;;
reload)
    ssreload
    ;;
*)
    echo "Usage: $0 {start|reload|stop}"
    exit 1
    ;;
esac
```

增加文件的执行权限，并加入rc.d中，实现开机启动

```
sudo chmod +x /etc/init.d/shadowsocks
sudo update-rc.d shadowsocks defaults
```

手动开启shadowsocks服务

```
sudo service shadowsocks start
```

#### Step5: shadowsocks自动重连

创建`autostart.sh`

```bash
#!/bin/bash
pids="$($_CMD pgrep ssserver)"
if [ ! $pids ]; then
        /usr/local/bin/ssserver -c /etc/shadowsocks.json -d start >> /root/ss.log
        newid="$($_CMD pgrep ssserver)"
        echo "Shadowsocks restarted at pid: $newid" >> /root/ss.log
fi
```

在crontab配置中，配置每天凌晨重启，以及每分钟检测断线重连

```bash
crontab -e
# 选择[3]: vim.basic
# 最后加入
0 4 * * * ssserver -c /etc/shadowsocks.json -d restart >> /root/ss.log
*/1 * * * * /root/autostart.sh
```

重启Crontab，使定时任务生效

```
service cron reload
```

#### 其他的配置

安装常用的oh-my-zsh, htop, glances等软件，参考[Ubuntu软件推荐](https://kevinzjy.github.io/2017/05/13/Ubuntu-softwares/)

