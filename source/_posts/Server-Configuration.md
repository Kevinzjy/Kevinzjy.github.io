---
title: 服务器上软件安装和配置
date: 2017-08-21 15:44:15
categories: Linux
tags: Server
---

最近实验室服务器经常出现问题，忍不了了，于是更换了Home目录位置，许多软件和环境配置都不好使了，一怒之下，我决定重装。。。

<!-- more -->

### 服务器环境配置

#### Bash 环境配置

[项目主页](https://github.com/Bash-it/bash-it)

```bash
git clone --depth=1 https://github.com/Bash-it/bash-it.git /histor/zhao/zhangjy/software/bash_it
cd /histor/zhao/zhangjy/software/bash_it
./install.sh

# 修改.bashrc 中的theme
export BASH_IT_THEME='clean'
PS1="${no_color}\u${reset_color}@${purple}\h${reset_color}:${blue}\W/${reset_color} \[\$(scm_prompt_info)\]$ "
```

### 常用开发软件安装

#### 使用Anaconda安装Python

```
wget https://repo.continuum.io/archive/Anaconda2-4.4.0-Linux-x86_64.sh
bash Anaconda2-4.4.0-Linux-x86_64.sh
# 按空格翻页，输入yes同意terms
# 输入一个anaconda2安装位置，必须是未创建的目录 /histor/zhao/zhangjy/software/anaconda2
```

常用的python模块:

| Name | Description  |
| :------------------- | :---------- |
| pip (for python 2.7) | python-pip |
| pip (for python 3) | python3-pip |

#### Perl

```bash
wget http://www.cpan.org/src/5.0/perl-5.26.0.tar.gz
```

#### Tmux

Tmux的安装比较简单，先安装libevent, ncurses

然后安装Tmux

```
CFLAGS="-I/usr/local/libevent/include -I/usr/local/ncurses/include" LDFLAGS="-L/usr/local/libevent/lib -L/usr/local/ncurses/lib" ./configure --prefix=/usr/local/tmux
```

环境变量配置

```bash
export LD_LIBRARY_PATH="/histor/zhao/zhangjy/software/libevent/lib":$LD_LIBRARY_PATH
export PATH="/histor/zhao/zhangjy/software/tmux/bin":$PATH
alias tmux='tmux -2'
alias ta='tmux attach -t'
alias tad='tmux attach -d -t'
alias ts='tmux new-session -s'
alias tl='tmux list-sessions'
alias tksv='tmux kill-server'
alias tkss='tmux kill-session -t'
```

将下列内容存为~/.tmux.conf

```
set -g default-terminal "screen-256color"

set -g prefix C-x
unbind C-b
bind r source-file ~/.tmux.conf \; display "Reloaded"

# select pane
bind k selectp -U # above (prefix k)
bind j selectp -D # below (prefix j)
bind h selectp -L # left (prefix h)
bind l selectp -R # right (prefix l)

# resize pane
bind -r ^k resizep -U 10 # upward (prefix Ctrl+k)
bind -r ^j resizep -D 10 # downward (prefix Ctrl+j)
bind -r ^h resizep -L 10 # to the left (prefix Ctrl+h)
bind -r ^l resizep -R 10 # to the right (prefix Ctrl+l)
```


