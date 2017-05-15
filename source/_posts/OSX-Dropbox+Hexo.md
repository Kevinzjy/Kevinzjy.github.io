---
title: MAC下使用Dropbox同步更新Hexo博客
date: 2017-05-14 15:25:13
categories: OS X
tags: [MAC, Hexo]
---

在Mac上许多配置与Linux系统不大相同

<!-- more -->

## OS X上Hexo博客的搭建

本人使用Dropbox同步Github pages目录，这样可以随时随地在设备上编辑博客并更新

在实际使用中，Ubuntu / Windows (Git bash) / Windows (WSL) 中使用Hexo十分完美，但是在OS X上，遇到了几个小问题

### Dropbox在OS X中无法实时同步

在Mac上修改文件，Windows下面会立即同步文件的更新, 但是反过来，Win下修改文件后，Mac下的Dropbox没有任何反应，必须要关闭后重新打开Dropbox才能开始同步

目前从官网下载了最新版本，暂时还没重现这个问题，版本号`v25.4.28`

### Hexo deploy的时候提示`Error: Cannot find module './build/Release/DTraceProviderBindings'`

在Mac上，使用Nodejs安装了Hexo之后`hexo deploy`时，会出现上述报错，可能是因为Hexo版本过低，或者插件没有安装完全

```bash
npm install hexo --no-optional
# 如果仍然不行
npm uninstall hexo-cli -g
npm install hexo-cli -g
```
