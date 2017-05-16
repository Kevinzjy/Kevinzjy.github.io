---
title: RNA-Seq中基因的差异表达分析
date: 2017-05-15 12:49:52
categories:
tags:
---

## 如何进行基因的差异表达分析

### Introduction

+ RNA-Seq produces gene expression profiles with much smaller technical variance
+ Gene expression level is measured by the number of reads mapped back to this gene

<!-- more -->

### Variance

+ Random sampling of reads
+ Other sources of variation including techinical and biological noise

#### Variance of Random sampling

Approximated by `Poisson distribution`

Consider biological variance: `over-dispersed Poisson distribution`

Models: 

+ Negative binomial (NB) distribution: 

    edgeR, DESeq, baySeq

+ Other models:

    - log fold changes obey the normal distribution

        * MA-based method: M(log fold change) at given A(log intensity sum) obey the normal distribution

        * two-step, generalized Poisson model to fir nt-wide read distribution

        * delta method: estimate variance of log-fold change (Cufflinks)

### DEGSeq: MA-based method

$$\lambda = x * n * l$$

