---
title: Anaconda 构建本地 channel
date: 2019-05-21 15:49:53
categories: Linux
tags: [Linux, Server, CentOS]
---

> 根据 Anaconda 软件源上的说明，Anaconda 和 Miniconda 是 Anaconda, Inc. 的商标，任何未经授权的公开镜像都是不允许的。此方法只适用于个人用途,国内的清华镜像也因此已经停止服务 (https://mirrors.tuna.tsinghua.edu.cn/news/close-anaconda-service/)

为了缩短 Anaconda 各种第三方包的安装时间, 最近在服务器上构建了 free / main 两个 channel 的本地镜像, 可以安装大部分常用的模块

<!-- more -->

#### 安装 Anaconda 和 conda build

conda 的安装就不多说了, 直接安装 miniconda / Anaconda 都可以, 安装成功之后, 还需要 conda build 这个工具用来构建本地索引

```bash
conda install conda-build
```

#### 下载官方 Channel

```bash
wget -r --no-parent https://repo.continuum.io/pkgs/main/linux-64/
wget -r --no-parent https://repo.continuum.io/pkgs/free/linux-64/
```

下载完后, 目录结构为

```bash
/opt/repo.continuum.io
|-- pkgs
|   |-- free
|   |   |-- linux-64
|   |-- main
|   |   |-- linux-64
```

#### Conda index 构建索引

```bash
conda index /opt/repo.continuum.io/pkgs/free
conda index /opt/repo.continuum.io/pkgs/main
```

如果在 index 过程中, 出现 `conda index error: KeyError`, 提示某个包的名字没有找到, 此时删除

`/opt/repo.continuum.io/pkgs/main/linux-64/patch_instructions.json`

后,重新 `conda index` 即可

#### 使用本地 Channel 安装

```
conda install -c file:///opt/repo.continuum.io/pkgs/main --override-channels --offline tensorflow-gpu
```

即可成功使用本地 channel 安装 tensorflow-gpu 模块, 同时在成功配置过一次之后, 再次安装时可以不指定channel, 也会自动搜索本地目录里的模块内容

#### 使用 .condarc 进行配置

我们可以通过配置 ~/.condarc 中加入如下内容，使其覆盖原有的default channel（repo.continuum.io/pkgs/free/linux-64），并默认不再联网进行搜索

默认配置内容:

```bash
channels:
  - bioconda
  - conda-forge
  - defaults
  - r
```

修改为

```
channels:
  - file:///opt/repo.continuum.io/pkgs/free

 offline: True
```

这样就可以强制使用本地源进行包的安装了
