---
title: Pysam使用心得
date: 2018-07-27 11:13:50
categories: Bioinformatics
tags: [Bioinfomatics, python]
---

## Pysam的使用

pysam.AlignedSegment 对象不可以pickle化，在处理时需要传递具体的cigar_tuple或者query_name进行分析

<!-- more -->

## 代码仓库

```python
import pysam
from itertools import izip_longest
from string import maketrans

trantab = maketrans('ATCGN', 'TAGCN')


def grouper(iterable, n, fillvalue=None):
    args = [iter(iterable)] * n
    return izip_longest(*args, fillvalue=None)


def sam_to_fastq(query_sequence, query_qualities, query_qualities):
    seq = query_sequence
    qual = ''.join([chr(i + 33) for i in query_qualities])
    if query_qualities:
        seq, qual = reverse_comp(seq), reverse(qual)
    return seq, qual


def reverse(seq):
    return seq[::-1]


def reverse_comp(seq):
    return ''.join(reverse(seq.translate(trantab)))


def unmapped_reads(bam, r1, r2):
    args = ['samtools', 'view', '-u', '-F', '256', bam]
    proc = subprocess.Popen(args, stdout=subprocess.PIPE)
    fd_child = proc.stdout.fileno()

    sam = pysam.AlignmentFile(fd_child, 'rb')
    for pair_num, (read1, read2) in enumerate(grouper(sam, 2)):
      pass

    for read in sam.fetch('chr1'):
      pass


def main():
    start = time.time()
    chroms = ['chr' + str(i) for i in range(1, 23)] + ['chrX', 'chrY']
    cpu = 4

    bam_file = 'test.sorted.bam'
    read1 = 'read1.fq.gz'
    read2 = 'read2.fq.gz'

    with gzip.open(read1, 'wb') as r1, gzip.open(read2, 'wb') as r2:
        proc_bam(bam_file)

    end = time.time()
    print (end - start)


if __name__ == '__main__':
    main()

```
