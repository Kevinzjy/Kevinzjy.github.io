---
title: 在Linux服务器上安装R
date: 2017-03-24 20:08:28
categories: Linux
tags: [Linux, Server, Software]
---

## 设置环境变量

```bash
export R_HOME=/panfs/home/zhao/zhangjy/software/R
export R_SRC=/panfs/home/zhao/zhangjy/pacakges
```

## 安装Dependencies

### zlib

```bash
cd $R_SRC
wget https://nchc.dl.sourceforge.net/project/libpng/zlib/1.2.11/zlib-1.2.11.tar.gz
tar zxvf zlib-1.2.11.tar.gz
cd zlib-1.2.11 
./configure --prefix=$R_HOME
make && make install
```

<!-- more -->

### bzip2

```bash
cd $R_SRC
wget http://www.bzip.org/1.0.6/bzip2-1.0.6.tar.gz -O ./bzip2-1.0.6.tar.gz
tar zxvf bzip2-1.0.6.tar.gz
cd bzip2-1.0.6
make && make install PREFIX=$R_HOME
```

### liblzma

```bash
cd $R_SRC
wget --no-check-certificate http://tukaani.org/xz/xz-5.2.3.tar.gz 
tar zxvf xz-5.2.3.tar.gz
cd xz-5.2.3
./configure --prefix=$R_HOME
make && make install
```

### pcre

```bash
cd $R_SRC
wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.40.tar.gz
tar zxvf pcre-8.40.tar.gz
cd pcre-8.40
./configure --enable-utf8 --prefix=$R_HOME
make && make install
```

### libcurl

```bash
cd $R_SRC
wget --no-check-certificate https://curl.haxx.se/download/curl-7.53.1.tar.gz
tar zxvf curl-7.53.1.tar.gz
cd curl-7.53.1
./configure --prefix=$R_HOME
make && make install
```

## 配置R安装环境

```bash
export LD_LIBRARY_PATH=$R_HOME/include:$LD_LIBRARY_PATH
export PATH=$R_HOME/bin:$PATH
export CFLAGS="-I$R_HOME/include"
export LDFLAGS="-L$R_HOME/lib"
```

## 编译和安装R

```bash
cd $R_SRC
wget https://mirrors.tuna.tsinghua.edu.cn/CRAN/src/base/R-3/R-3.3.3.tar.gz
tar zxvf R-3.3.3.tar.gz
cd R-3.3.3
./configure --prefix=$R_HOME
make && make install
```

### 修改.zshrc或者.bashrc

在rc文件中增加如下内容

```
export PATH=/panfs/home/zhao/zhangjy/software/R/bin:$PATH
export LD_LIBRARY_PATH=/panfs/home/zhao/zhangjy/software/R/lib:$LD_LIBRARY_PATH
export LD_LIBRARY_PATH=/panfs/home/zhao/zhangjy/software/R/include:$LD_LIBRARY_PATH
```
