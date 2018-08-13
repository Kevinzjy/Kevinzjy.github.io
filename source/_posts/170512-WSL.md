---
title: Bash on Ubuntu on Windows的安装和配置 (一) WSL的安装和配置
date: 2017-05-12 23:26:40
categories: Windows
tags: [Windows, Bash on Ubuntu]
---

记录基本的 Bash on Ubuntu on Windows (WSL) 的安装和配置过程
参考[官方说明](https://msdn.microsoft.com/en-us/commandline/wsl/about)

## Step1: Bash on Ubuntu on Windows的下载和安装

### 更新系统版本

在 设置 -> 系统 -> 关于 中，查看系统版本号，系统必须为64位，版本号至少为14393.0，如果版本过低，检查更新至14393以上版本

### 打开开发者模式

在 设置 -> 更新和安全 -> 针对开发人员 中，选中开发人员模式, 重启电脑

在 控制面板 -> 程序 -> 程序和功能 -> 启动或关闭Windows功能 中，勾选适用于Linux的Windows子系统

打开命令行，输入`bash`, 阅读条款后输入"y"确认条款，下载安装Ubuntu sub system。安装完成后输入用户名和密码后，安装成功

<!-- more -->

## Step2: 更新Ubuntu版本

检查Windows创意者更新，在Windows升级至1703版本后，WSL会自动升级至16.04 LTS

```bash
# 手动查看Ubuntu版本号
lsb_release -a
sudo do-release-upgrade # Manual upgrade to Ubuntu 16.04 
```

## Step3: 安装Ubuntu后进行的设置，[参考](https://cxfer.cn/2016/79.html):

* 新升级至Ubuntu 16.04 LTS后，打开Bash会提示 'To run a command as administrator (user "root"), use "sudo <command>". See "man sudo_root" for details.'，随便执行一个sudo命令，即可去掉该提示

```bash
sudo -i
```

* 在Ubuntu升级后，sudo apt update可能会出现 "N: Ignoring file '50unattended-upgrades.ucf-dist' in directory '/etc/apt/apt.conf.d/' as it has an invalid filename extension" 错误，将该旧版配置文件删除后 `sudo apt update` 即可解决

```bash
sudo rm /etc/apt/apt.conf.d/50unattended-upgrades.ucf-dist && sudo apt update
```

* 默认系统中提示音非常烦人，可以通过配置 $HOME/.inputrc 去除

```bash
echo "set bell-style none" > $HOME/.inputrc
```

* Ubuntu中默认安装的Vim版本有BUG在Insert模式中不能识别方向键,可以通过卸载并重新安装Vim解决

```bash
sudo apt remove vim
sudo apt install vim
```

* 去掉Vim中烦人的提示音

```bash
echo ":set vb t_vb=" > $HOME/.vimrc
```

* 在执行需要联网的命令如apt时会提示 "sudo: unable to resolve host DESKTOP-2Q1PJM7", 同时，在sudo apt update的时候会出现getaddrinfo.c错误: "../sysdeps/posix/getaddrinfo.c:2591: getaddrinfo: Assertion `(__extension__ ({ const struct in6_addr *__a = (const struct in6_addr *) (sin6->sin6_addr.__in6_u.__u6_addr32); __a->__in6_u.__u6_addr32[0] == 0 && __a->__in6_u.__u6_addr32[1] == 0 && __a->__in6_u.__u6_addr32[2] == __bswap_32 (0xffff); }))' failed."

```
cat /etc/hostname # 查看本机hostname
sudo vi /etc/hosts
127.0.0.1 DESKTOP-2Q1PJM7 # 改为你的hostname
```

Note：目前我还没解决第二个Assertion报错，但是遇到报错时重复运行多次命令可以暂时解决

* WSL系统中的文件在Windows下无法读取，但是Windows的磁盘分区已经自动挂载，如 C:/ -> /mnt/c ，可以通过创建软链接的方式便于访问Windows下的文件

```bash
ln -s /mnt/c C
ln -s /mnt/d D
```

### 安装Git

```bash
sudo apt install git
```

这里我在Github上面设置了ssh-key登录

```bash
ssh-keygen # 生产ssh-key后把pub key内容复制到github设置中
git config --global user.email "kevinzjy1997@gmail.com" # 配置github账户
```

### 安装Cmder

从[官方网站](http://cmder.net/)下载Cmder，并在[ConEmu](https://www.fosshub.com/ConEmu.html)下载ConEmu的更新，安装后覆盖至Cmder/vendor/conemu-maximus5，完成对ConEmu的更新

在Cmder中，直接运行Bash.exe时无法使用方向键。需要点击右下角 Settings -> Startup -> Tasks，新建任务

```
C:\Windows\System32\bash.exe ~ -cur_console:p:n # 直接进入~目录
```

即可在Bash中使用方向键, 如果使用 `-cur_console:p1` 会导致Vim中无法使用方向键

### 安装Bash-it对Bash环境进行美化

在Windows 1703 创意者更新后，Cmder + zsh 环境配置下会出现光标前出现空格的Bug，目前暂未解决，因此使用Bash-it对默认的Bash环境进行美化

```bash
git clone https://github.com/revans/bash-it.git ~/.bash_it
~/.bash_it/install.sh
```

更改主题: 在 ~/.bashrc 中，把theme设为"simple"

```bash
export BASH_IT_THEME='simple' # 主题配置

export SCM_CHECK=true # Git插件配置
export SCM_GIT_SHOW_DETAILS=true 
export SCM_GIT_SHOW_MINIMAL_INFO=true # 防止Git提示符乱码

export TERM=xterm-256color # 设置Terminal类型，支持256色
LC_ALL=en_US.UTF-8
```

### 安装X-server用来运行图形化程序

默认情况下，WSL不能直接运行图形化程序，可以通过安装X-server并监听端口的方式解决

```bash
export DISPLAY=:0 # 写入 .bashrc, 可以省去每次打开Bash后输入的麻烦
```

在Windows下，安装VcXsrv，运行VcXsrv监听:0端口

```bash
Rstudio # 即可在Windows下启动Rstudio的图形化界面
```

不过用这种方法，打开的图形化界面太丑陋了，不能直视啊。。。

### Tips: WSL系统内可以直接运行Windows命令

```bash
sublime_text.exe # 启动sublime text
```

但是不能在命令行直接对Linux下的文件进行编辑，应该是因为传递的文件名参数问题。这点其实可以通过一些方法解决，但是目前还没有这方面需求，懒得折腾了。。。

安装完这些，基本就可以在命令行和Bash on Ubuntu on Windows愉快地玩耍了~ Enjoy!
