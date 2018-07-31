---
title: Pysam使用心得
date: 2018-07-27 11:13:50
categories: Bioinformatics
tags: [Bioinfomatics, python]
---

# pysam以及Samtools使用心得

## pysam简介

pysam 是一个基于 htslib 的 C++ API 进行封装的 python 模块,实现了对 SAM / BAM / CRAM 文件的便捷操作,可以简化 bam 文件处理的代码复杂度，同时也可以处理 VCF / BCF 等其他文件

## pysam 的安装

pysam 已经在 [pypi](https://pypi.org/project/pysam/) 中包含，可以直接使用 `pip` 命令安装

```bash
pip install pysam
```

<!-- more -->

## pysam 的使用

### 统计 bam 文件中总 reads 数目以及比对到参考基因组上的 reads 数目

利用 `pysam.AlignmentFile` 对象，我们可以使用 `pysam.AlignmentFile.count()` 方法快速统计其中的比对片段数目，然而由于存在一条 read 比对到基因组中多个位置的情况，因此直接统计的比对片段数目与 reads 数是不同的

可以使用 `read_callback` 参数，使用callback函数对比对片段进行筛选，具体细节请参考[pysam API](http://pysam.readthedocs.io/en/latest/api.html)


`read_callback (string or function)` –
> select a call-back to ignore reads when counting. It can be either a string with the following values:

`all` -
> skip reads in which any of the following flags are set: BAM_FUNMAP, BAM_FSECONDARY, BAM_FQCFAIL, BAM_FDUP

`nofilter` -
> uses every single read

使用`read_callback`结合自定义callback函数，我们可以方便地统计总 reads 数目以及比对到基因组上的reads数

**Note**: 默认情况下，`pysam.AlignmentFile.count()` 只统计比对上的片段，因此统计 total reads 数目时需要指定 `until_eof=True` 参数，才可以包含未比对上的片段信息

下面是一个使用 pysam 统计 total reads 与 mapped reads 数目的实例程序：

```python
import pysam


def main(bam_file):
    sam = pysam.AlignmentFile(bam_file, 'rb')
    total = sam.count(read_callback=total_callback, until_eof=True)
    unmapped = sam.count(read_callback=unmapped_callback)
    mapped = total - unmapped
    return total, mapped


def unmapped_callback(read):
    """
    callback function for mapped reads
    """
    return read.is_unmapped or read.mate_is_unmapped and not read.is_supplementary and not read.is_secondary


def total_callback(read):
    """
    callback function for total reads
    """
    return not read.is_supplementary and not read.is_secondary
```

### pysam 多进程处理 bam 文件

pysam中多进程处理需要注意一些事情

#### - 如何多进程处理bam文件

通过`pysam.AlignmentFile.fetch()`方法，可以提取出比对到指定染色体或者指定区域的比对片段，因此可以多进程同时处理多个染色体上的比对结果

#### - 注意事项

1. 利用同一个`pysam.AlignmentFile`句柄同时进行`fetch`操作可能发生冲突，因此每个进程需要重新打开自己的新句柄，并使用`fetch(multiple_iterators=True)`参数，使得句柄之间相互独立，可以避免文件读取时发生问题

2. `pysam.AlignedSegment`对象不能pickle化，因此无法在多进程中作为其他函数的参数，需要自己构建tuple/list用于参数传递

3. 如果每个子进程都需要相同的辅助数据，可以使用initializer进行全局对象的共享，效率比为每个进程提供相同参数要高很多

实例程序：

```python
import pysam
from multiprocess import Pool


AUX_DATA=None
def initializer(aux_data):
    global AUX_DATA
    AUX_DATA=aux_data


def worker(bam_file, chrom):
    sam = pysam.AlignmentFile(bam_file, 'rb')
    cnt = 0
    for read in sam.fetch(chrom, multiple_iterators=True):
        # use tuple to process AlignedSegment
        proc(read_query_name, read_is_mapped)
        cnt += 1
    # use aux_data
    print AUX_DATA
    return cnt


def proc(query_name, is_mapped):
    pass:


def main(bam_file):
    sam = pysam.AlignmentFile(bam_file, 'rb')
    header = sam.header['SQ']
    sam.close()

    aux_data = ['chr1', ;'chr2', 'chr3', ]

    pool = Pool(thread, initializer, (aux_data, ))
    jobs = []
    for chrom in header:
        jobs.append(pool.apply_async(worker, (bam_file, chrom, )))
    pool.close()
    pool.join()

    for job in jobs:
        ret = job.get()

```

## pysam与samtools结合使用

在`pysam.AlignmentFile()`打开bam文件时，一般需要bam文件已经排序并且已建立好index

可以使用`sort --threads`以及`index -@`多进程
```bash
samtools sort --threads 10 -o test.sorted.bam test.bam
samtools index -@ 10 test.sorted.bam
```

需要注意，`samtools index`多进程运行之后生成的index文件的时间戳会出现异常，在程序里需要等待几秒后再使用pysam继续处理，否则会提示`index file is older than data`

```python
import time
import subprocess
subprocess.call('samtools index -@ 10 test.sorted.bam', shell=True)
sam = pysam.AlignmentFile('test.sorted.bam')
```
