---
title: 在Windows下使用Github pages + Hexo搭建个人博客
date: 2017-03-24 21:16:34
categories: Windows
tags: [Windows, Software, Hexo]
---

## 依赖工具

在Windows下安装Git + Node.js + Cmder

### 安装Git

下载Git，[下载地址](https://git-scm.com/downloads)
安装时选择添加Git到$PATH


### 配置Cmder

安装[Cmder](http://cmder.net/), 在Cmder中添加{git bash}: 

``` bash
Task parameters: /icon "C:\Program Files\Git\git-bash.exe"
Commands: "C:\Program Files\Git\bin\sh.exe" --login -i
```

<!-- more -->

### 安装Node.js

下载Node.js, [下载地址](ttps://nodejs.org/en/)


## 安装Hexo

``` bash
npm install hexo-cli -g

mkdir -p /c/Users/kevin/Dropbox/git/Kevinzjy.github.io
cd /c/Users/kevin/Dropbox/git/Kevinzjy.github.io

hexo init
npm install
npm install hexo-deployer-git --save #安装git插件

hexo clean 
hexo g # generate 
hexo s # start

```

编辑_congif.yml

``` 
deploy:
  type: git
  repo: git@github.com:Kevinzjy/Kevinzjy.github.io.git
  branch: master
```

deploy到github

```
hexo d # deployment
```

当出现warning: LD will be replaced by CRLF in ...时，删除hexo文件夹下面的.deplo_git文件夹

```
git config --global core.autocrlf false
```

### Hexo主题配置

``` bash
git clone https://github.com/iissnan/hexo-theme-next themes/next
rm -rf ./themes/next/.git # 删除.git目录，防止github build时出错
```

编辑_config.yml，将theme改为next

### NexT主题配置

坑

## 附：Ubuntu中安装nodejs

```bash
wget https://nodejs.org/dist/v6.10.3/node-v6.10.3-linux-x64.tar.xz
tar xvf node-v6.10.3-linux-x64.tar.xz
mv node-v6.10.3-linux-x64 ../node-v6.10.3
```

编辑.zshrc文件，加入nodejs的路径

```
export PATH=/home/zhangjy/software/node-v6.10.3/bin:$PATH
npm install hexo-cli -g
```