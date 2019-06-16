---
title: CentOS7 集群配置 Docker
date: 2019-06-15 16:57:39
categories: Linux
tags: [Linux, Server, CentOS]
---

集群中 Docker 的安装和配置

<!-- more -->

#### Docker 安装

```bash
# 安装 Docker-CE
rpm -ivh --force /histor/public/yum/docker/*
```

#### Docker 的启动

```bash
# 启动 Docker
systemctl start docker

# 查看 docker 配置
docker info

# 运行测试程序
docker run hello-world
```

#### Docker更改安装位置

```
# 停止 Docker 服务
systemctl stop docker

# 创建配置文件的目录
mkdir -p /etc/systemd/system/docker.service.d

# 编辑配置文件
vi /etc/systemd/system/docker.service.d/docker-storage.conf
```

通过配置文件指定 docker 存储文件夹

```bash
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H fd:// --data-root /histor/public/docker/node142
```

编辑 `/etc/docker/daemon.json`，更改 Docker 使用的存储驱动

```json
{
  "storage-driver": "devicemapper"
}
```

保存后，重新启动 Docker 服务

```
systemctl daemon-reload
systemctl start docker
```

运行 `docker-info` 即可看到 Docker 的安装位置已经发生了变化

```
Docker Root Dir: /histor/public/docker/login2
```

#### 给非root用户授权，可以使用 Docker

```bash
usermod -aG docker zhangjy
```

#### 计算节点离线安装镜像

```bash
# 下载镜像
docker pull tensorflow/tensorflow

# 将下载的镜像打包到文件
docker save -i /histor/public/docker/save/tensorflow.tar tensorflow/tensorflow

# 切到计算节点，进行安装
docker load /histor/public/docker/save/tensorflow.tar
```