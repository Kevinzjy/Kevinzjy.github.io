---
title: 在Linux服务器上安装GCC，Boost和Cmake
date: 2017-03-24 21:46:27
categories: Linux
tags: [Server, Linux, Software]
---

## GCC的安装

### 下载gcc-6.3.0

```bash
cd /panfs/home/zhao/zhangjy/software/src/gcc_src/
wget http://ftp.tsukuba.wide.ad.jp/software/gcc/releases/gcc-6.3.0/gcc-6.3.0.tar.gz
tar zxvf gcc-6.3.0.tar.gz
cd gcc-6.3.0
```

### 安装依赖的mpfr, gmp和mpc等依赖工具

```bash
./contrib/download_prerequisites
```

<!-- more -->

### 在新的目录下编译

```bash
mkdir -p /panfs/home/zhao/zhangjy/software/src/gcc_build
cd /panfs/home/zhao/zhangjy/software/src/gcc_build
/panfs/home/zhao/zhangjy/software/src/gcc_src/gcc-6.3.0/configure --prefix=/panfs/home/zhao/zhangjy/software/gcc --disable-multilib
```

### 修改环境变量

```bash
echo $LD_LIBRARY_PATH
echo $LIBRARY_PATH
```

如果输出的`$LD_LIBRARY_PATH`或`LIBRARY_PATH`是以`:`结尾,比如:


```
/public/software/intel/composer_xe_2011_sp1.7.256/compiler/lib/intel64:/public/software/intel/composer_xe_2011_sp1.7.256/mkl/lib/intel64:
```

需要去掉结尾处的`:`，否则`make`时候会报错


```bash
export LIBRARY_PATH=/public/software/intel/composer_xe_2011_sp1.7.256/compiler/lib/intel64:/public/software/intel/composer_xe_2011_sp1.7.256/mkl/lib/intel64
```

### 安装GCC

```bash
make -j 10 # make in 10 threads
make install
```

### 添加新安装的GCC到环境变量

**Note**: `$LD_LIBRARY_PATH`中要加入两个目录

```bash
export PATH=/panfs/home/zhao/zhangjy/software/gcc/bin:$PATH
export LD_LIBRARY_PATH=/panfs/home/zhao/zhangjy/software/gcc/lib/gcc/x86_64-pc-linux-gnu/6.3.0:$LD_LIBRARY_PATH
export LD_LIBRARY_PATH=/panfs/home/zhao/zhangjy/software/gcc/lib64:$LD_LIBRARY_PATH
```

### 当安装其他软件时，指定调用新版的GCC
```bash
export CC=/panfs/home/zhao/zhangjy/software/gcc/bin/gcc
export CXX=/panfs/home/zhao/zhangjy/software/gcc/bin/g++
cmake $YOUR_PATH -DCMAKE_INSTALL_PREFIX=$INSTALL_DIR
```

## Boost的安装

### 下载[Boost](http://www.boost.org/)

```bash
wget https://nchc.dl.sourceforge.net/project/boost/boost/1.63.0/boost_1_63_0.tar.gz
tar zxvf boost_1_63_0.tar.gz
cd boost_1_63_0
```

### 安装Boost

```bash
sh ./bootstrap.sh
```

### 指定boost安装目录和include, lib文件夹的位置

```bash
./bjam -j 4 --prefix=/panfs/home/zhao/zhangjy/software/boost --includedir=/panfs/home/zhao/zhangjy/software/boost --libdir=/panfs/home/zhao/zhangjy/software/boost install
```

### 添加到环境变量

```bash
export PATH=/panfs/home/zhao/zhangjy/software/boost/boost:$PATH
export LD_LIBRARY_PATH=/panfs/home/zhao/zhangjy/software/boost/lib:$LD_LIBRARY_PATH
```

## 安装Cmake

### 下载Cmake

```bash
wget https://cmake.org/files/v3.7/cmake-3.7.2.tar.gz --no-check-certificate
tar zxvf cmake-3.7.2.tar.gz
cd cmake-3.7.2
```

### 安装Cmake

```bash
./bootstrap --prefix=/panfs/home/zhao/zhangjy/software/cmake
gmake
```
