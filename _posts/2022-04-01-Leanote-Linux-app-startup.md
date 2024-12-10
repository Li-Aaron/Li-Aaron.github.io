---
layout: post
title: Leanote APP in Linux startup menu with ICON
date: 2022-04-01 22:00:00.000000000 +08:00
description: 本文介绍之前没有解决的Leanote APP在Linux没有图标的问题。这里重新介绍一下添加启动器和图标的方法。
author: aaron-li
categories: [工具指南]
tags: [leanote, proxy] 
---

本文介绍之前没有解决的Leanote APP在Linux没有图标的问题。  
这里重新介绍一下添加启动器和图标的方法。  
关于Leanote其他部分介绍详见：[告别印象笔记，使用Leanote自建云笔记]({{site.url}}/2021/08/leanote-replace-evernote/)

# 1 下载与解压缩
官网下载地址： [http://app.leanote.com/](http://app.leanote.com/)

Github下载地址： [https://github.com/leanote/desktop-app/releases/](https://github.com/leanote/desktop-app/releases/)

解压缩到任意位置。

# 2 Linux 中添加启动器图标
打开一个terminal  
```bash
cd ~/.local/share/applications
vi leanote.desktop
```

输入并保存
```ini
[Desktop Entry]
Encoding=UTF-8
Version=1.5
Comment=cloud based note-taking application
Type=Application
Terminal=false
Name=Leanote
Exec=/path/to/leanote/Leanote
Icon=/path/to/leanote/leanote.png
StartupNotify=true
Categories=TextEditor;
SingleMainWindow=true
X-GNOME-UsesNotifications=true
X-GNOME-SingleWindow=true
StartupWMClass=leanote-desktop
```

效果如下：
![1](/assets/img/posts/2022-04-01-Leanote-Linux-app-startup/1.png)  

![2](/assets/img/posts/2022-04-01-Leanote-Linux-app-startup/2.png)
