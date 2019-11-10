---
title: CentOS7 安装 pdftk
date: 2019-11-10 18:54:15
categories: Linux
tags: [Linux, Server, CentOS]
---

我们在文章投稿的时候，会需要提供一个包括所有 Figure 的 pdf 文件。此时，`pdftk` 这个工具可以帮助我们快速合并已有的 pdf，并生成一个无压缩的合并文件。

<!-- more -->

#### 为什么使用 pdftk

一开始，我是使用了 `ghostscript` 进行 pdf 合并

```bash
gs -dNOPAUSE \
   -dColorConversionStrategy=/LeaveColorUnchanged \
   -dEncodeColorImages=false \
   -dEncodeGrayImages=false \
   -dEncodeMonoImages=false \
   -sDEVICE=pdfwrite \
   -dPDFSETTINGS=/prepress \
   -sOUTPUTFILE=converted.pdf \
   -dBATCH \
   Fig1.pdf Fig2.pdf Fig3.pdf Fig4.pdf Fig5.pdf Fig6.pdf
```

然而，从文件的大小来和合并之后的画质来看， `ghostscript` 仍然对我的图像进行了压缩，效果比较糟糕。因此，我选择了 `pdftk` 来直接合并文件

#### pdftk 安装

在 Centos 6 系统中，`pdftk` 直接提供了 RPM 的安装方式。然而我使用的是 WSL 中的 CentOS 7 因此需要额外进行一些操作，才可以成功使用 `pdftk`

首先，我们需要安装一些 `pdftk` 需要的依赖库

```bash
sudo yum install gcc gcc-c++ libXrandr gtk2 libXtst libart_lgpl
```

pdftk 需要的 libgcj 并不在标准库中，因此我们需要下载 rpm 文件到本地安装

```bash
wget http://li.nux.ro/download/nux/dextop/el7/x86_64//libgcj-4.8.2-16.el7.i686.rpm
sudo rpm -ivh ./libgcj-4.8.2-16.el7.i686.rpm
```

最后，安装 pdftk 的本体

```bash
sudo yum localinstall https://www.linuxglobal.com/static/blog/pdftk-2.02-1.el7.x86_64.rpm
```

#### 使用 pdftk 合并 pdf

```bash
pdftk ./Fig1.pdf ./Fig2.pdf ./Fig3.pdf ./Fig4.pdf ./Fig5.pdf ./Fig6.pdf cat output converted.pdf
```

经过对比发现，ghostscript 文件压缩之后的大小为 7M 左右，而 pdftk 合并之后的文件大小在 14M，足足差了一倍。可能在 ghostscript 也可以通过增加一些参数指定不进行压缩，但是在简单的 pdf 合并任务上，pdftk 还是更加合适的。