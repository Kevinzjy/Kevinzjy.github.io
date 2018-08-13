---
title: 利用Sra-tools从NCBI下载SRA数据
date: 2017-05-12 15:56:45
categories: Bioinformatics
tags: [Bioinfomatics, Bioinformatic-Tools]
---

## Sra-tools的下载与安装

Note: 旧版的Sratoolkit已经更新为Sra-tools

参考NCBI官方教程[Wiki](http://ncbi.github.io/sra-tools/install_config.html)

```bash
# Downloading SRA Toolkit
wget "http://ftp-trace.ncbi.nlm.nih.gov/sra/sdk/current/sratoolkit.current-centos_linux64.tar.gz"
```

<!-- more -->

```bash
# Unpacking the Toolkit 
tar -zxvf sratoolkit.current-centos_linux64.tar.gz
cd sratoolkit.2.8.2-1-centos_linux64
```

```bash
# Add to $PATH
export PATH=[sratoolkit_path]:$PATH
```

```
# Test sratoolkit
fastq-dump -X 5 -Z SRR390728
```

```
# Configure 
vdb-config -i
```

在vdb-config的图形界面里更改[Default Import Path]，最好不要放在用户$HOME目录下，否则SRA文件可能会导致用户目录占用空间
当
某些情况下，vdb-config可能不能打开图形化界面，可以手动在配置文件中指定SRA文件的储存位置

```bash
# 手动指定sra_path
echo '/repository/user/main/public/root = "[sra_path]"' > $HOME/.ncbi/user-settings.mkfg
```

## 如何下载SRA文件

一般情况下，我们下载的是paired-end reads，而默认的fastq-dump会把read1,read2解压在同一文件中，我们需要手动指定--split-3来分别获得read1和read2

```bash
# Download DRX033310_1.fastq.gz, DRX033310_2.fastq.gz
fastq-dump --split-3 --gzip SRR390728
```