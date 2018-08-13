---
title: PBS作业管理系统使用技巧
date: 2018-8-13 14:16:14
categories: Bioinformatics
tags: [Bioinfomatics, ]
---

PBS是一种十分常用的集群作业调度系统，主要包括openPBS, PBS Pro和Torque三种分支。本文以Torque为例，简单介绍一下PBS作业调度系统的使用技巧

## Torque介绍

什么是TORQUE：

> Terascale Open-source Resource and QUEue Manager (TORQUE) 是一个分布式计算节点和批处理作业的资源调度系统，常在高性能计算集群中用于计算作业的提交和管理。用户可以使用TORQUE将计算作业提交到不同计算队列，常用的命令包括qsub(提交作业), qstat(作业状态查询)和qdel(删除作业)

官网地址： http://www.adaptivecomputing.com/support/documentation-index/torque-resource-manager-documentation/

<!-- more -->

## Torque使用基础

### 作业提交和管理命令

| 命令 | 功能 | 使用说明 | 示例 |
|-----|-----|-----|-----|
| `qsub` | 提交pbs作业 | qsub [script] | `$ qsub job.pbs`
| `qdel` | 删除pbs作业 | qdel [job_id] | `$ qdel 12345`
| `qhold` | 挂起pbs作业 | qhold [job_id] | `$ qhold 12345`
| `qrls` | 释放挂起的pbs作业 | qrls [job_id] | `$ qrls 12345`

### 作业状态查询命令

| 命令 | 说明 |
| ---- | ----|
| `qstat -q` | 列出所有队列 |
| `qstat -a` | 列出所有作业 |
| `qstat -u user_id` | 列出user_id的所有作业 |
| `qstat -r` | 列出所有正在运行的作业 |
| `qstat -f job_id` | 列出作业job_id的信息 |
| `qstat -fQ queue` | 列出队列queue的信息 |
| `qstat -B` | 列出所有作业状态的汇总 |
| `pbsnodes` | 列出所有节点的详细信息 |
| `pestat` | 列出所有节点的状态 |

### PBS作业参数

```bash
#!/bin/bash
#PBS -l nodes=1:ppn=16
#PBS -l walltime=1000:00:00
#PBS -q high
#PBS -N Job_Name
#PBS -oe

your_commands_goes_here
```

### 提交交互式作业

我们可以通过`-I`选项提交一个交互式的作业，效果类似直接ssh登录到计算节点

`qsub -I -N stdin -l nodes=1:ppn=16 -l walltime=1000:00:00 -q high`

## 批量提交作业

在很多情况下，我们需要提交大量相似的任务，比如大量样本的转录组数据分析，每个样本数据的分析流程都是一致的，提交的脚本中一般只有样本名字不同，那么如果对每个样本产生产生一个shell脚本，是非常麻烦的。在这种情况下，我们可以通过任务模板与`qsub -v`命令，进行作业提交时的参数传递。

模板任务文件：`template.sh`

```bash
#!/bin/bash
#PBS -j oe
#PBS -q high
#PBS -l nodes=1:node:ppn=4
#PBS walltime=1000:00:00

hisat2 -p 4 -x hg19 -1 ./reads/${sample}_1.fq.gz -2 ./reads/${sample}_2.fq.fz -t | samtools view -bS > ./align/${sample}.bam
```

任务参数文件：`job_params.csv`
```
sample1
sample2
sample3
```

任务投递脚本：`qsub.py`
```python
#!/usr/bin/env python
import csv
import subprocess
import time

param_file = './job_params.csv'
cwd = '/home/kevinzjy/RNA-seq'

with open(param_file, 'r') as f:
    reader = csv.reader(f)
    for sample in reader:
        qsub_cmd = 'qsub -N {0} -d {1} -v SAMPLE={0} template.sh'.format(sample, cwd)
        # print qsub_cwd

        exit_status = subprocess.call(qsub_cmd, shell=True)
        if exit_status is 1:
            print 'Job "{}" failed to submit'.format(qsub_cwd)
        time.sleep(1)

print "Done submitting jobs!"
```

通过`qsub.py`，我们可以实现对`job_params.csv`里面每一行对应的样本数据

## 批量删除作业

如果要删除一个用户所有的作业，可以使用qselect结合xargs命令，进行作业编号的提取和对指定编号作业的删除

`qselect -u user_id | xargs qdel`

利用`qselect`命令，我们可以增加参数进一步缩减选择的作业范围，详细信息可以参考`man qselect`中的说明

| 参数 | 说明 |
| ---  | --- |
| -N | 指定作业名字 |
| -s | 指定状态 [EHQRTW] |
| -u | 指定用户列表 |

删除某用户所有正在排队的任务：

```bash
qselect -u user_name -s Q | xargs qdel
```

xargs命令的作用是读入标准输入的内容并分行，对其中每一行的内容都执行相同的命令，在这里xargs读入了前面管道中qselect命令得到的某用户的所有作业编号，对于每个编号执行了qdel命令，起到了删除该用户名下所有作业的效果