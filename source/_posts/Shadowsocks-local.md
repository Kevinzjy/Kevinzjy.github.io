---
title: shadowsocks教程
date: 2017-05-12 15:56:45
categories: Bioinformatics
tags: [Windows, OS X, Linux]
---

## Windows下使用Shadowsocks

### 配置Shadowsocks客户端

下载[Shadowsocks-4.0.6.zip](https://github.com/shadowsocks/shadowsocks-windows/releases/download/4.0.6/Shadowsocks-4.0.6.zip)

下载完成后，可以在习惯的位置创建一个新的文件夹，将`Shadowsocks.exe`解压到该目录下

右键**以管理员身份运行**`Shadowsocks.exe`，会在当前目录下自动生成下列文件（所以单独放在一个新文件夹下面比较好）

    - Shadowsocks.exe
    - statistics-config.json
    - user-wininet.json
    - ss_win_temp
    - gui-config.json (后面配置好节点信息后生成)

在服务器配置页面填入你的shadowsocks节点地址，端口，密码等信息，保存后点击添加，将节点配置保存到gui-config.json，记住配置中的代理端口(默认为1080)

{% asset_img Review-Invitation.png %}

保存后，右键任务栏中的shadowsocks图标（小飞机），启用系统代理，并设置为开机启动

系统代理模式 -> PAC模式，PAC -> 使用本地PAC，从GFWlist更新本地PAC

### 为浏览器设置代理

Internet Explorer 和 Microsoft Edge (Win10) 无需进行额外设置，如果使用Chrome浏览器，可以使用[SwithyOmega](https://github.com/FelisCatus/SwitchyOmega/releases/download/v2.5.6/SwitchyOmega_Chromium.crx)进行代理切换配置






