---
title: ggplot2作图
date: 2017-03-24 23:04:04
categories: Bioinformatics
tags: [Bioinfomatics, R, ggplot2]
---

## ggsci颜色设置

```R
library("ggsci")
library("ggplot2")
library("gridExtra")

data("diamonds")

p1 = ggplot(subset(diamonds, carat >= 2.2),
       aes(x = table, y = price, colour = cut)) +
  geom_point(alpha = 0.7) +
  geom_smooth(method = "loess", alpha = 0.05, size = 1, span = 1) +
  theme_bw()

p2 = ggplot(subset(diamonds, carat > 2.2 & depth > 55 & depth < 70),
       aes(x = depth, fill = cut)) +
  geom_histogram(colour = "black", binwidth = 1, position = "dodge") +
  theme_bw()

p1_npg = p1 + scale_color_npg()
p2_npg = p2 + scale_fill_npg()
grid.arrange(p1_npg, p2_npg, ncol = 2)
```

<!-- more -->

{% asset_img ggsci_npg_scale.png %}

```R
mypal <- pal_npg("nrc", alpha=0.7)(9)

library(scales)
show_col(mypal)
```

{% asset_img ggsci_npg_pal.png %}

## ggplot legend

```R
get_legend <- function(p) {
    tmp <- ggplot_gtable(ggplot_build(p))
    leg <- which(sapply(tmp$grobs, function(x) x$name) == "guide-box")
    legend <- tmp$grobs[[leg]]
    return(legend)
}

p1_legend <- get_legend(p1)
plot(p1_legend)
```

{% asset_img p1_legend.png %}

## ggplot transparency

ggplot2 中，将 plot 存为 eps 格式可以更加方便地使用 AI 进行编辑

```R
ggsave("./plot.eps", width=5, height=5)
```

但是默认的 eps 输出格式不能保存透明度，会出现报错

```
Warning message:
In grid.Call.graphics(L_rect, x$x, x$y, x$width, x$height, resolveHJust(x$just,  :
  此装置不支持半透明：每一页将被报告一次
```

此时可以使用 ggsave 中指定设备为 cairo_ps

```R
ggsave("./F1B1.eps", device=cairo_ps, width = 6, height = 10)
```

或者直接使用 cairo_ps 保存图片

```R
cairo_ps("./F1B1.eps")
print(f1b1_plot)
dev.off()
```