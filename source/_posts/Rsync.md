---
title: rsync 拷贝指定拓展名文件
date: 2020-07-28 09:57:06
categories: Linux
tags: [Linux]
---

使用 rsync 命令拷贝目录下所有具有指定拓展名的文件（`*.fastq.gz`, `*.fq.gz`)

```bash
rsync -avP --include="*/" --include="*.fq.gz" --include="*.fastq.gz" --exclude="*" src_dir tgt_dir
```