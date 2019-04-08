---
title: Anaconda3 环境配置
date: 2019-04-08 16:02:34
categories: Linux
tags: [Linux, Server]
---

*背景:* 最近院里集群存储出现了一点问题，导致 .bashrc 文件无法打开，因此需要重新配置环境。

在配置环境的过程中，先安装了 anaconda3，然而配置好环境变量之后，发现每次登录服务器需要等待 4-5 秒后才会出现命令行提示符，十分难受。经过检查，发现是在 anaconda 安装时配置的环境变量有点问题，将其更改为手动添加后，登录服务器的过程又变得十分顺畅了。

#### Anaconda3 提供的配置

```bash
# added by Anaconda3 4.5.12 installer
# >>> conda init >>>
# !! Contents within this block are managed by 'conda init' !!
__conda_setup="$(CONDA_REPORT_ERRORS=false '$HOME/software/anaconda3/bin/conda' shell.bash hook 2> /dev/null)"
if [ $? -eq 0 ]; then
    \eval "$__conda_setup"
else
    if [ -f "$HOME/software/anaconda3/etc/profile.d/conda.sh" ]; then
        . "$HOME/software/anaconda3/etc/profile.d/conda.sh"
        CONDA_CHANGEPS1=false conda activate base
    else
        \export PATH="$HOME/software/anaconda3/bin:$PATH"
    fi
fi
unset __conda_setup
# <<< conda init <<<
```

#### 手动添加的配置

```bash
export PATH="$HOME/software/anaconda3/bin":$PATH
```
