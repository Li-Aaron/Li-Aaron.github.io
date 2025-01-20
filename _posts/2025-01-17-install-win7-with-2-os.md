---
layout: post
title: 在MBR XP+Ubuntu下安装Win7实现三系统
date: 2025-1-17 19:00:00.000000000 +08:00
description: 老电脑系列番外篇，给装好Ubuntu和Windows xp的Thinkpad X201i再安装Win7，并修复Win7的引导。
author: aaron-li
categories: [玩机攻略, 老电脑]
tags: [thinkpad, thinkpad x201i, refresh, ubuntu, windows xp, windows 7]
---


最近在扫描照片，但是扫描仪的驱动只支持Win7，而这台[老电脑]({{site.url}}/2019/11/thinkpadx201i-refresh/)上面装了XP和Ubuntu。

XP里面有很多老游戏，不想直接升级成Win7，这次试一试怎么在XP和Ubuntu都装好的情况下装Win7。

## 0. 准备工作

需要一个Ubuntu的安装U盘来修改磁盘分区（[Gparted](https://gparted.org/display-doc.php?name=moving-space-between-partitions)）。

这一次推荐用[Ventoy](https://www.ventoy.net/cn/index.html)来安装，可以找一个大一点的U盘安装Ventoy，再把ISO放到Ventoy提供的分区里，在引导界面选择需要的ISO就可以自动进入引导，无需再制作Ubuntu的安装U盘。

安装升级非常简单：

![Ventoy-install1](/assets/img/posts/2025-01-17-install-win7-with-2-os/Snipaste_2025-01-17_13-52-47.png)

将ISO放进去：

![Ventoy-install2](/assets/img/posts/2025-01-17-install-win7-with-2-os/Snipaste_2025-01-17_13-56-20.png)

进入U盘启动后，直接就可以加载ISO了：

![Ventoy-install3](/assets/img/posts/2025-01-17-install-win7-with-2-os/101.jpg)

## 1. 磁盘分区

方法和安装Kali三系统是一样的，这里参考[在MBR (Legacy BIOS) 双系统下安装Kali实现三系统]({{site.url}}/2022/01/install-kali-on-mbr-with-two-os/#1-磁盘分区)。  

利用Ventory启动ubuntu的Live CD，选择`Try Ubuntu`。  

![Try Ubuntu](/assets/img/posts/2021-01-11-install-xp-under-ubuntu/try-ubuntu.png)  

Ubuntu 18.04.05 Live CD自带gparted，`ALT+T`打开terminal后，键入  
```bash
sudo gparted
```

进入Gparted修改分区，移动Ubuntu分区到右侧，需要等待10+分钟（取决于文件的多少，使用的磁盘是HDD还是SSD）。  

![Gparted-size](/assets/img/posts/2022-01-13-install-kali-on-mbr-with-two-os/gparted1.png)

并新建一个NTFS分区。  

![Gparted-size2](/assets/img/posts/2025-01-17-install-win7-with-2-os/gparted2.png)

点击确定，等待分区结束。  

![Gparted-success](/assets/img/posts/2025-01-17-install-win7-with-2-os/gparted3.png)

## 2. Ghost安装Win7

用PE系统中的"手动安装GHOST"，在新建的NTFS分区中安装Win7系统。

这里直接参考[Ghost安装XP]({{site.url}}/2021/01/install-xp-under-ubuntu/#3-ghost安装xp)，不再赘述。


## 3. 修复Win7引导

安装玩Ghost Win7是不能直接启动的，需要修复BCD。

这里采用了`ntbootautofix`这个tool。[下载链接](https://github.com/Li-Aaron/Li-Aaron.github.io/releases/download/0.0.2/ntbootautofix.zip)

先进入XP系统，打开`ntbootautofix`:

![ntbootautofix-1](/assets/img/posts/2025-01-17-install-win7-with-2-os/1.JPG)

选1 自动修复：

![ntbootautofix-2](/assets/img/posts/2025-01-17-install-win7-with-2-os/2.JPG)

修复成功。

在最初始的界面如果选2 高级：

![ntbootautofix-3](/assets/img/posts/2025-01-17-install-win7-with-2-os/3.JPG)

再选4 查看/管理 BCD 引导配置：

![ntbootautofix-4](/assets/img/posts/2025-01-17-install-win7-with-2-os/4.JPG)

可以看到BCD中有两个启动项了，这里也可以给它们改名字。


重启进系统，选择Grub中的XP系统（进入的是Windows Boot Manager）

![bcd-1](/assets/img/posts/2025-01-17-install-win7-with-2-os/102.jpg)

这里grub的名字是我手动改的，正常还是显示XP。

![bcd-2](/assets/img/posts/2025-01-17-install-win7-with-2-os/103.jpg)

可以看到Windows 7 和 Windows XP 都在。

选择 Windows 7 启动：

![bcd-3](/assets/img/posts/2025-01-17-install-win7-with-2-os/104.jpg)


## 4. 后记

开始用Win7啦。

![bcd-4](/assets/img/posts/2025-01-17-install-win7-with-2-os/desktop.png)


## 5. 失败的尝试
### 5.1 Win7直接安装到新分区失败

其实能不用GHOST系统最好不用，毕竟每个GHOST系统都有夹带。

自己在别的机器上安装再build GHOST image又麻烦。

最初我的尝试是用Ventoy引导Win7安装盘来安装。

但是卡在了这一步：

![error](/assets/img/posts/2025-01-17-install-win7-with-2-os/error.jpg)

不知道怎么解决。

### 5.2 Boot-repair后 Win7进入失败

如果用Boot-repair来修复Grub，会在Grub中生成Windows XP和Windows 7两个启动项。

但是启动Windows 7会提示找不到ntoskrnl.exe。

必须从Windows XP启动项来启动正确的Windows Boot Manager。

## Reference
[Gparted](https://gparted.org/display-doc.php?name=moving-space-between-partitions)  
[Ventoy](https://www.ventoy.net/cn/download.html)  
