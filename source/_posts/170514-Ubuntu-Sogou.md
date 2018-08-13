---
title: Ubuntu 16.04 LTS 安装搜狗拼音输入法
date: 2017-05-14 20:00:10
categories: Linux
tags: [Linux, Ubuntu]
---

记录一下Ubuntu 16.04 LTS中如何安装搜狗拼音输入法

参考:

+ [如何在Sublime Text 3中输入中文](http://sklcc.github.io/blog/2014/11/21/how-to-input-chinese-in-sub/)

+ [点滴记录——在Ubuntu 14.04中使SublimeText 3支持中文输入法](http://blog.csdn.net/cywosp/article/details/32350899)

<!-- more -->

### 下载安装Sougou输入法

在搜狗输入法[官方页面](http://pinyin.sogou.com/linux/?r=pinyin)，下载适合系统的deb安装文件，安装

```bash
sudo dpkg -i sogoupinyin_2.1.0.0086_amd64.deb
# 如果出现依赖未安装
sudo apt install -f
sudo dpkg -i sogoupinyin_2.1.0.0086_amd64.deb
```

### 设置Fcitx输入法

在Ubuntu中，打开System Settings -> Language Support，根据提示安装完整的语言支持，并将Keyboard input method system设为fcitx

### 在Fcitx中添加搜狗输入法

打开Terminal(快捷键 Ctrl + Alt + T)，输入`fcitx-configtool`，点击添加，去掉勾选的Only Show Current Language，添加Sougou Pinyin

在System Settings -> Keyboard -> Shortcuts -> Typing中，将Switch to next source和Switch to previous source的选项，都设为Diabled(点击选项后，按Backspace)。这样可以禁用系统的切换输入法快捷键，搜狗拼音输入法的默认快捷键为`Ctrl + ,`

### 在Sublime Text 3中添加搜狗输入法的支持

Sublime Text是一款跨平台下十分好用的文本编辑软件，然而在Ubuntu下，默认安装的Sublime Text 3不支持搜狗中文输入法，需要手动编译一个共享库

#### 安装依赖软件

安装编译共享库的依赖'build-essential'和'libgtk2.0-dev'

```bash
sudo apt install build-essential libgtk2.0-dev
```

#### 编写sublime-imfix.c文件

在随意文件添加一个sublime-imfix.c，内容为

```c
/*
 * sublime-imfix.c
 * Use LD_PRELOAD to interpose some function to fix sublime input method support for linux.
 * By Cjacker Huang <jianzhong.huang at i-soft.com.cn> *
 *
 * gcc -shared -o libsublime-imfix.so sublime_imfix.c  `pkg-config --libs --cflags gtk+-2.0` -fPIC
 * LD_PRELOAD=./libsublime-imfix.so sublime_text
 */
#include <gtk/gtk.h>
#include <gdk/gdkx.h>

typedef GdkSegment GdkRegionBox;

struct _GdkRegion
{
    long size;
    long numRects;
    GdkRegionBox *rects;
    GdkRegionBox extents;
};

GtkIMContext *local_context;

void
gdk_region_get_clipbox (const GdkRegion *region,
                        GdkRectangle    *rectangle)
{
    g_return_if_fail (region != NULL);
    g_return_if_fail (rectangle != NULL);

    rectangle->x = region->extents.x1;
    rectangle->y = region->extents.y1;
    rectangle->width = region->extents.x2 - region->extents.x1;
    rectangle->height = region->extents.y2 - region->extents.y1;
    GdkRectangle rect;
    rect.x = rectangle->x;
    rect.y = rectangle->y;
    rect.width = 0;
    rect.height = rectangle->height;

    //The caret width is 2;
    //Maybe sometimes we will make a mistake, but for most of the time, it should be the caret.
    if (rectangle->width == 2 && GTK_IS_IM_CONTEXT(local_context)) {
        gtk_im_context_set_cursor_location(local_context, rectangle);
    }
}

//this is needed, for example, if you input something in file dialog and return back the edit area
//context will lost, so here we set it again.

static GdkFilterReturn event_filter (GdkXEvent *xevent, GdkEvent *event, gpointer im_context)
{
    XEvent *xev = (XEvent *)xevent;

    if (xev->type == KeyRelease && GTK_IS_IM_CONTEXT(im_context)) {
        GdkWindow *win = g_object_get_data(G_OBJECT(im_context), "window");

        if (GDK_IS_WINDOW(win)) {
            gtk_im_context_set_client_window(im_context, win);
        }
    }

    return GDK_FILTER_CONTINUE;
}

void gtk_im_context_set_client_window (GtkIMContext *context,
                                       GdkWindow    *window)
{
    GtkIMContextClass *klass;
    g_return_if_fail (GTK_IS_IM_CONTEXT (context));
    klass = GTK_IM_CONTEXT_GET_CLASS (context);

    if (klass->set_client_window) {
        klass->set_client_window (context, window);
    }

    if (!GDK_IS_WINDOW (window)) {
        return;
    }

    g_object_set_data(G_OBJECT(context), "window", window);
    int width = gdk_window_get_width(window);
    int height = gdk_window_get_height(window);

    if (width != 0 && height != 0) {
        gtk_im_context_focus_in(context);
        local_context = context;
    }

    gdk_window_add_filter (window, event_filter, context);
}
```

在该文件所在目录下，编译

```bash
gcc -shared -o libsublime-imfix.so sublime-imfix.c  `pkg-config --libs --cflags gtk+-2.0` -fPIC
```

编译后可以得到libsublime-imfix.so这个共享库

#### 添加共享库到Sublime Text 3

首先，测试一下是否在Sublime Text 3中可以输入中文

```bash
LD_PRELOAD=./libsublime-imfix.so subl
```

如果可以正常使用搜狗输入法，进行接下来的配置

将libsublime-imfix.so拷贝到系统库的默认路径下：

```bash
sudo cp libsublime-imfix.so /usr/lib
```

修改Sublime的执行文件`/usr/bin/subl`

```
#!/bin/sh
LD_PRELOAD=/usr/lib/libsublime-imfix.so exec  /opt/sublime_text/sublime_text "$@"
```

修改Sublime的快捷方式`/usr/share/applications/sublime_text.desktop`:

```bash
# Exec=/opt/sublime_text/sublime_text %F
# 修改为
Exec=bash -c 'LD_PRELOAD=/usr/lib/libsublime-imfix.so /opt/sublime_text/sublime_text' %F

# Exec=/opt/sublime_text/sublime_text -n
# 修改为
Exec=bash -c 'LD_PRELOAD=/usr/lib/libsublime-imfix.so /opt/sublime_text/sublime_text' -n
```

这样就可以在命令行`subl`和系统Application中的Sublime Text 3中输入中文了
