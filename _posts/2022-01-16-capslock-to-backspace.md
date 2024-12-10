---
layout: post
title: CapsLock修改为Backspace
date: 2022-1-16 17:00:00.000000000 +08:00
description: 本文介绍如何在Windows/Linux下，将CapsLock键修改为BackSpace。
author: aaron-li
categories: [工具指南]
tags: [capslock, backspace] 
---

## 1. Windows

在Command Prompt（命令提示行）下输入，然后敲回车。

```cmd
@echo off
reg add "hklm\system\currentcontrolset\control\keyboard layout" /v "ScanCode Map" /t REG_BINARY /d "0000000000000000020000000e003a0000000000" /f
echo "please reboot system"
pause
```

经测试Windows xp/Windows 10有效。

## 2. Linux

在Terminal下

```bash
vim ~/.Xmodmap
```

输入
```bash
!Turn Caps Lock into a second BackSpace key
remove Lock = Caps_Lock
keysym Caps_Lock = BackSpace
```

之后
```bash
xmodmap ~/.Xmodmap
```

经测试Ubuntu/Kali有效。
