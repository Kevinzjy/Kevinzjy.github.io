---
title: 无重复的RNA-Seq数据进行基因差异表达分析
date: 2017-05-20 23:08:22
categories: Bioinformatics
tags: [Bioinformatics, RNA-Seq, Differential Expression]
---

## GFOLD: a generalized fold change for ranking differentially expressed genes from RNA-Seq data

简单介绍一下无重复的RNA-Seq数据如何进行较为准确的差异表达分析

参考资料:

Feng J, Meyer CA, Wang Q, Liu JS, Liu XS, Zhang Y. GFOLD: a generalized fold change for ranking differentially expressed genes from RNA-seq data. **Bioinformatics** 2012

[全文链接](https://doi.org/10.1093/bioinformatics/bts515)

<!-- more -->

### Abstract

#### Motivation

+ RNA-Seq目前在转录组分析中，被广泛用来评价基因表达的水平
+ 虽然测序成本在快速降低，但是目前70%的人类RNA-Seq样本仍然没有生物学重复(此文发表在2012年)
+ 目前并没有令人满意的对没有生物学重复的数据进行差异表达分析的方法

#### Results

+ 文章中提出了GFOLD(Generalized fold change)算法，用来提出具有生物学意义的差异表达排序方法
+ GFOLD基于变化倍数的对数的后验分布，提出了一个可靠地统计量
+ GFOLD的方法克服了P-value和Fold change方法的缺陷，提出了更稳定和具有生物学意义的单样本基因差异表达排序的方法

#### Availability:

[Bitbucket](https://bitbucket.org/feeldead/gfold/), [项目主页](http://www.tongji.edu.cn/~zhanglab/GFOLD/index.html)

### Methods:

在忽略RNA-seq中的Techinical variance的情况下，一个基因中的Reads数目可以使用泊松分布进行估计, 在一个基因中观测到k条Reads的概率可以表示为:
$$P(k)=\frac{\lambda^ke^{-\lambda}}{k!},\lambda=n\times l\times x$$

其中\\(x\\)是基因的表达量(e.g. RPKM)，\\(n\\)是一个反应样本测序深度的标准化常数,\\(l\\)是基因的长度

根据贝叶斯理论，\\(\lambda\\)和\\(x\\)为随机变量，因此，\\(\lambda\\)的后验概率可以表示为:
$$Post(\lambda)\propto\frac{\lambda^ke^{-\lambda}}{k!}$$

是一个shape为\\(k+1\\)，scale为\\(1\\)的伽马分布，这里使用均匀分布作为\\(\lambda\\)的先验分布

根据这个分布，可以计算出表达量变化倍数的对数\\(log_2(x_2/x_1)\\)的后验分布。在计算过程中，涉及到\\(l\\)的计算，可以通过令\\(y_i=l\times x_i,i\in\\{1,2\\}\\)，计算\(z=log_2(y_2/y_1)\\)来避开。可以明显看出\\(log_2(y_2/y_1)\\)和\\(log_2(x_2/x_1)\\)拥有相同的分布。

为了充分利用到Fold change的后验分布的方差的信息，我们定义Generalized fold change(GFOLD)为:
$$ GFOLD(c)=\left\\{
\begin{aligned}
max(t,0)|P_Z(z\leq t)=c, if\ mean(Z)\geq 0 \\\\
min(t,0)|P_Z(z\geq t)=c, if\ mean(Z)<0
\end{aligned}
\right.
$$

其中\\(P_z\\)是\\(z=log_2(y_2/y_1)\\)的后验分布，\\(c\\)是一个自己定义的参数，默认值为0.01。如果该基因表达上调，则\\(mean(Z)\geq 0\\)，log2 fold change大于t的概率是\\(1-c\\)。那么，在默认的c下\\(t< mean(Z)\\)，\\(GFOLD(c)\\)计算时根据\\(max(t,0)\\)，将取值为负值的t变成0。

如果\\(GFOLD(c)\\)为0，说明该基因的表达量没有明显变化。而且mean(Z)<0的情况和mean(Z)>0时是对称的，因此可以通过\\(GFOLD(c)\\)来为基因的表达量变化进行降序排名，排名靠前的基因有明显的表达上调，而排名末尾的基因为表达量明显下调。

### Results

{% asset_img GFold.png %}

图中展示了，GFOLD和直接计算Log2 fold change以及使用Poisson test获得的p值的比较。可以看出，使用GFOLD进行差异表达打分的排名，同时考虑到了log2 fold change后验分布的均值和方差，排名更具有生物学意义。

### GFOLD的实现方法

在N足够大的时候，可以发现,log2 fold change的后验概率可以近似于正态分布。根据[gamma分布的性质](https://en.wikipedia.org/wiki/Gamma_distribution):

>If \\(X\sim Gamma(\alpha, \theta)\\) and \\(Y\sim Gamma(\beta, \theta)\\) are independently distributed, then \\(X/(X+Y)\\) has a beta distribution with parameters \\(\alpha\\) and \\(\beta\\).

可以计算出GFOLD的精确值，但是计算闭合形式的方程过于浪费时间，我们可以使用sampling方法进行粗略的估算。文章中，首先基于Gamma分布对于\\(y_1\\), \\(y_2\\)进行了随机抽样，进而获得\\(z\\)的分布，最终可以算出GFold(c)的值，虽然在小数点精度不是很高，但是与传统的P-value和直接计算Fold change的方法相比，GFOLD的排名应该仍然更可信。



