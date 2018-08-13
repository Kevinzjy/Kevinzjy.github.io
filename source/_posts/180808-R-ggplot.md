---
title: ggplot2作图技巧
date: 2018-08-08 23:04:04
categories: Bioinformatics
tags: [Bioinfomatics, R, ggplot2 ]
---

# patchwork实现ggplot多图拼接

## patchwork介绍

ggplot2是大家使用R语言作图时常用的一个模块，而patchwork可以将多个ggplot图片通过简洁的语法进行拼接，类似功能的模块还有`gridExtra::grid.arrange`和`cowplot::plot_grid`，不过个人认为patchwork更好用一些，因为其语法更加贴近普通用户的使用习惯

Github项目地址: [https://github.com/thomasp85/patchwork](https://github.com/thomasp85/patchwork)

详细使用说明可以查看项目中`README.md`

<!-- more -->

## Patchwork的安装

1 - 下载并安装Rtools

首先，我们需要下载 `Rtools` ，一个windows系统下用来编译R模块的程序

rtools下载地址：[https://cran.r-project.org/bin/windows/Rtools/](https://cran.r-project.org/bin/windows/Rtools/)

下载并安装与系统R版本对应的exe文件即可（程序安装包较大，100MB+）

2 - 安装devtools

`devtools` 是用来帮助我们安装第三方开发的R模块的R package

项目地址: [https://cran.r-project.org/web/packages/devtools/index.html](https://cran.r-project.org/web/packages/devtools/index.html)

可以直接使用 `install.packages()` 安装

```R
install.packages("devtools")
```

3 - 升级rlang版本（非必须）

如果rlang版本较低，在安装 `patchwork` 时会出现以下报错

```
* installing *source* package 'patchwork' ...
** R
** preparing package for lazy loading
Error : object 'enexprs' is not exported by 'namespace:rlang'
ERROR: lazy loading failed for package 'patchwork'
```

升级rlang版本即可解决：

```R
devtools::install_github("r-lib/rlang", build_vignettes = TRUE)
```

4 - 安装Patchwork

```R
devtools::install_github("thomasp85/patchwork")
```

不要觉得安装过程过于麻烦，当你用上Patchwork的时候就会发现一切都是值得的...

## 使用diamonds测试数据集绘图

以diamonds数据集为例，我们可以简单地绘制两个figure

```R
library("ggplot2")
data(diamonds)

p1 <- ggplot(subset(diamonds, carat >= 2.2),
            aes(x = table, y = price, colour = cut)) +
    geom_point(alpha = 0.7) +
    geom_smooth(method = "loess", alpha = 0.05, size = 1, span = 1) +
    theme_bw() +
    theme(aspect.ratio = 1)

p2 <- ggplot(subset(diamonds, carat > 2.2 & depth > 55 & depth < 70),
            aes(x = depth, fill = cut)) +
    geom_histogram(colour = "black", binwidth = 1, position = "dodge") +
    theme_bw() +
    theme(aspect.ratio = 1)
```

{% asset_img p1.png %}

{% asset_img p2.png %}

我们可以看到，利用默认格式生成的图片配色是非常难看的...

### Tips: 利用ggsci进行颜色图片美化

`ggsci`总结了许多杂志常用的颜色，内置了多个调色板，该模块可以直接通过CRAN安装

```R
install.packages("ggsci")
```

使用ggsci中nejm配色作为自动填充后

{% asset_img p1_npg.png %}


```R
library(ggsci)

p1_npg <- p1 + scale_color_npg()

p2_npg <- p2 + scale_fill_npg()

```

{% asset_img p1_npg.png %}

可以看出，绘制出的图片色调明显更加协调了

{% asset_img p1_npg.png %}

ggsci提供了多个调色板，可以使用`show_col`命令进行查看

{% asset_img p1_npg.png %}


```R
mypal <- pal_npg("nrc", alpha=0.7)(9)

library(scales)
show_col(mypal)
```

{% asset_img ggsci_npg_pal.png %}


## 利用patchwork合并图片

在实际作图中，我们经常会遇到需要合并图片的需求，这时候之前安装的patchwork就派上用场了

```R
library(patchwork)

p <- p1_npg + p2_npg
```

{% asset_img p.png %}


我们可以看到p1和p2直接拼接在一起了

### Tips: 使用自定义函数提取图例

对于p1和p2,我们可以提取其中的图例并进行单独排版

```R
get_legend <- function(p) {
    tmp <- ggplot_gtable(ggplot_build(p))
    leg <- which(sapply(tmp$grobs, function(x) x$name) == "guide-box")
    legend <- tmp$grobs[[leg]]
    return(legend)
}

p1_legend <- get_legend(p1_npg)
```

{% asset_img p1_legend.png %}

在提取出图例之后，可以使用`plot_layout`指定布局列数和每个子图的宽度

```R
p_merged <- p1_npg + guides(color=FALSE) +
    p2_npg + guides(fill=FALSE) +
    p1_legend + plot_layout(ncol=3, widths = c(3,3,1))
```

{% asset_img p_merged.png %}

可以将p1,p2以及图例完美地拼接在一起

{% asset_img p_merged.png %}

{% asset_img p_merged.png %}

### Tips: ggplot保存文件

ggplot2 中，将 plot 存为 eps 格式可以更加方便地使用 AI 进行后续的编辑和修图

```R
ggsave("./plot.eps", p_merged)
```

但是默认的 eps 输出格式不能保存透明度，当图片使用了不同的透明度时，会出现报错

```
Warning message:
In grid.Call.graphics(L_rect, x$x, x$y, x$width, x$height, resolveHJust(x$just,  :
  此装置不支持半透明：每一页将被报告一次
```

此时可以使用 ggsave 中指定 `device=cairo_ps`,使用 cairo_ps 类型保存图片

```R
ggsave("./plot.eps", p_merged, device=cairo_ps)
```

同样，对于使用pdf格式保存的图片，当出现格式问题时，也可以使用cairo_pdf作为保存设备

```R
ggsave('./plot.pdf', p, device=cairo_pdf)
```
