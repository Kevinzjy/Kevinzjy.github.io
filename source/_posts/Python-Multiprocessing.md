---
title: Python中的多进程
date: 2017-03-29 12:55:17
categories: Python
tags: [Python, Multiprocessing]
---

Python中的多进程使用

## Multiprocessing进程池

```python
from multiprocessing import Pool

def run_func(msg):
    print msg

pool = Pool(4)
jobs = []
for i in range(4):
    jobs.append(pool.apply_async(run_func, (i, )))
pool.close()
pool.join()
```

## Multiprocessing线程池

```python
from multiprocessing.dummpy import Pool

def run_func(msg):
    print msg

pool = Pool(4)
jobs = []
for i in range(4):
    jobs.append(pool.apply_async(run_func, (i)))
pool.close()
pool.join()
```
