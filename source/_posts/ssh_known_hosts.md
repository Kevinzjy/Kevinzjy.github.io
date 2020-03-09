---
title: ssh 跳过身份验证
date: 2020-03-09 19:30:42
categories: Linux
tags: [Linux, Server]
---

最近遇到了一个奇怪的情况，有用户反映从在服务器上无法用`scp`命令向其他所的服务器拷贝数据，报错提示 `REMOTE HOST IDENTIFICATION HAS CHANGED!`，可能是身份验证出现了问题。

<!-- more -->

#### 问题1

根据提示，报错原因是远程服务器 ECDSA key 发生了变化，和 `~/.ssh/known_hosts` 中记录的 key 不同。因此删除已记录的 key 之后，应该就可以重新登录。

```bash
[user1@manager ~]$ scp src_data.fq.gz CASPMI@remote_server:/remote/data/dir
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the ECDSA key sent by the remote host is
00:25:2c:82:e2:51:18:34:da:ab:56:a4:ad:cc:a6:19.
Please contact your system administrator.
Add correct host key in /home/user1/.ssh/known_hosts to get rid of this message.
Offending ECDSA key in /home/user1/.ssh/known_hosts:2
Password authentication is disabled to avoid man-in-the-middle attacks.
Keyboard-interactive authentication is disabled to avoid man-in-the-middle attacks.
Permission denied (publickey,gssapi-with-mic,password,hostbased).
lost connection
```

在删除了 `~/.ssh/known_hosts` 的第二行记录后，果然可以正常scp拷贝数据了

#### 问题2

过了几个小时，该用户发现又开始出现相同报错，同时反映需要登录的服务器是使用动态验证码+验证卡登录。因此猜测远程服务器可能在不断更新 Host key 。。。因此之前的方法只能暂时解决问题。

最终解决方案：编辑 ssh 配置文件 `~/.ssh/config`，跳过 Host key 的验证

```bash
Host remote_server
  StrictHostKeyChecking no
  UserKnownHostsFile=/dev/null
```

这样配置后，不管远程服务器怎么更新，都不需要进行 known_hosts 的检查了，成功解决。
