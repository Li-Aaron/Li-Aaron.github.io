---
layout: post
title: 在Ubuntu下安装Windows XP实现双系统
date: 2021-1-11 18:00:00.000000000 +08:00
description: 老电脑系列番外篇，给Thinkpad X201i安装Windows XP，Windows下安装Ubuntu很方便，反过来还是要费一番心思的。
author: aaron-li
categories: [玩机攻略, 老电脑]
tags: [thinkpad, thinkpad x201i, refresh, ubuntu, windows xp]
---


继[去年给老Thinkpad X201i换了网卡]({{site.url}}/2020/02/thinkpadx201i-refresh-2/)之后，用了一段时间，网速快起来了。  

最近看了一些关于暗黑破坏神2的情怀文章，突然就想玩了，结果发现主力机Win10各种兼容性问题。  
![We-have-a-big-error](/assets/img/posts/2021-01-11-install-xp-under-ubuntu/we-got-a-big-error.png)

暗黑2使用Direct 3D 7，而Virtual box不支持Direct 3D 7，[只支持Direct 3D 8/9](https://www.virtualbox.org/manual/UserManual.html#guestadd-3d)。  
在Vritual box中只能检测到DirectDraw驱动，游戏过程非常卡顿。  
![Vbox-no-d3d7](/assets/img/posts/2021-01-11-install-xp-under-ubuntu/vbox-no-d3d7.png)

<!--
![Diablo-no-d3d](/assets/img/posts/2021-01-11-install-xp-under-ubuntu/diablo-no-d3d.png) 
-->

所以我选择在这台[老电脑]({{site.url}}/2020/02/thinkpadx201i-refresh/)上装一个XP双系统。  

## 0. 准备工作

需要一个Ubuntu的安装U盘来修改磁盘分区（[Gparted](https://gparted.org/display-doc.php?name=moving-space-between-partitions)），以及修复引导（[Boot-Repair](https://help.ubuntu.com/community/Boot-Repair)）。  
需要一个PE启动盘来安装GHOST的Windows XP。  
（U盘安装XP失败了，才出此下策，本人**不推荐**使用GHOST安装系统，会导致各种不稳定的问题）  

只有Legacy BIOS的老电脑，使用UltraISO生成的安装盘是无法正常boot的。  
![UltraISO-image-gen](/assets/img/posts/2021-01-11-install-xp-under-ubuntu/ultraISO.png)

Boot的时候提示`Failed to load ldlinux.c32`  
![Fail to boot](/assets/img/posts/2021-01-11-install-xp-under-ubuntu/ubuntu-ultraISO-fail.jpg)

这里用[Rufus](https://rufus.ie/zh_CN.html)生成安装盘。  
![Rufus-image-gen](/assets/img/posts/2021-01-11-install-xp-under-ubuntu/rufus-ubuntu.png)

成功启动Live Ubuntu。  
![Live Ubuntu](/assets/img/posts/2021-01-11-install-xp-under-ubuntu/ubuntu-live.png)


## 1. 磁盘分区

在Windows下安装Ubuntu双系统，在Ubuntu的live CD里面就有所支持，相对是比较简单的。  
而在Ubuntu下安装Windows的先例比较少，装XP这种古老的系统就更少了。  

最初尝试了在Win PE系统下使用DiskGenius缩小Ubuntu的ext4分区，但是失败了。  
而在Ubuntu下直接用Gparted，因为磁盘已经挂载，不能修改分区。  
![Gparted-fail](/assets/img/posts/2021-01-11-install-xp-under-ubuntu/gparted-locked.png)

图中可以看到`Resize`是灰色的。  

在这种情况下，只运行在内存里的ubuntu的Live CD下，Gparted就可以修改分区了。在这里选择`Try Ubuntu`。  
![Try Ubuntu](/assets/img/posts/2021-01-11-install-xp-under-ubuntu/try-ubuntu.png)

Ubuntu 18.04.05 Live CD自带gparted，`ALT+T`打开terminal后，键入
```Bash
sudo gparted
```

进入Gparted修改分区，移动分区到右侧，需要等待10+分钟（取决于文件的多少，使用的磁盘是HDD还是SSD）。  
![Gparted-size](/assets/img/posts/2021-01-11-install-xp-under-ubuntu/gparted-size.png)

并新建一个NTFS分区（也可以在PE的Disk Genius中新建），完成后如图所示。  
![Gparted-success](/assets/img/posts/2021-01-11-install-xp-under-ubuntu/gparted-after.png)


## 2. 激活NTFS分区

这一步实际上是为了测试WinNTSetup工具，参考的文章是[WinPE环境下WinNTSetup使用说明](https://my.oschina.net/hsds/blog/1841991)。  
虽然后续安装失败了，本人认为这一步骤还是有用的，至少可以保证从第一个盘启动。  
如果跳过这一步直接使用Ghost安装XP不保证能直接进入XP系统。  

首先在DiskGenius清除保留扇区。  
![DG-clean](/assets/img/posts/2021-01-11-install-xp-under-ubuntu/dg1.png)

取消Ubuntu分区的激活，并激活左侧新建的NTFS分区。  
![DG-activate](/assets/img/posts/2021-01-11-install-xp-under-ubuntu/dg2.png)

## 3. Ghost安装XP

用PE系统中的"手动安装GHOST"，在新建的NTFS分区中安装XP系统。

步骤如下：  
选择Local --> Partition --> From Image。  
![Ghost1](/assets/img/posts/2021-01-11-install-xp-under-ubuntu/ghost1.png)
找到需要安装的系统的GHO文件。  
![Ghost2](/assets/img/posts/2021-01-11-install-xp-under-ubuntu/ghost2.png)
Sorce Partition就是GHO文件中的镜像，直接OK。    
![Ghost3](/assets/img/posts/2021-01-11-install-xp-under-ubuntu/ghost3.png)
选择所要安装的磁盘。  
![Ghost4](/assets/img/posts/2021-01-11-install-xp-under-ubuntu/ghost4.png)
选择所要安装的分区，这里选择第一个NTFS的分区。  
![Ghost5](/assets/img/posts/2021-01-11-install-xp-under-ubuntu/ghost5.png)
最后确认。  
![Ghost6](/assets/img/posts/2021-01-11-install-xp-under-ubuntu/ghost6.png)

等待安装完成后，重新启动为XP系统，Ubuntu消失。  
![Windows-xp-startup](/assets/img/posts/2021-01-11-install-xp-under-ubuntu/windows-xp-startup.jpg)


## 4. 修复Ubuntu引导

要让Ubuntu系统回来，需要重建Ubuntu的grub。  
这里推荐Ubuntu官方的教程[Recovering Ubuntu After Installing Windows](https://help.ubuntu.com/community/RecoveringUbuntuAfterInstallingWindows)。  

在Live CD的Terminal下，键入如下命令：  
```bash
sudo add-apt-repository ppa:yannubuntu/boot-repair
sudo apt-get update
sudo apt-get install -y boot-repair && boot-repair
```

在[Boot-Repair](https://help.ubuntu.com/community/Boot-Repair)工具的图形界面中选择`Recommand repair`。  
![boot-repair界面](/assets/img/posts/2021-01-11-install-xp-under-ubuntu/boot-repair.png)

等待程序完成后，重启，自动进入新的Ubuntu Grub引导界面。  
![Ubuntu新的引导界面](/assets/img/posts/2021-01-11-install-xp-under-ubuntu/ubuntu-grub.jpg)

可以看到Ubuntu与XP双系统成功实现。

## 5. 后记

暗黑走起！  
![Diablo](/assets/img/posts/2021-01-11-install-xp-under-ubuntu/diablo-start.jpg)


## 6. 失败的尝试
### 6.1 Rufus生成XP安装盘，但Windows安装失败。
使用Rufus生成XP的U盘安装盘：  
![Rufus-image-gen-xp](/assets/img/posts/2021-01-11-install-xp-under-ubuntu/rufus-winxp.png)

安装时提示没有镜像，安装直接退出。  
![xpU盘安装失败](/assets/img/posts/2021-01-11-install-xp-under-ubuntu/windows-xp-install-fail.jpg)

### 6.2 WinNTSetup，进入安装界面时蓝屏。
尝试使用了PE下的工具WinNTSetup。  
![WinNtSetup](/assets/img/posts/2021-01-11-install-xp-under-ubuntu/WinNtSetup.png)

重启后，在Windows XP安装程序加载完毕后蓝屏，这里忘了拍照，网上找个相同错误码的图片，侵联删。  
![BSOD](/assets/img/posts/2021-01-11-install-xp-under-ubuntu/BSOD.png)

这个可能是由于BIOS设置中硬盘模式为AHCI的原因。  
改为Compatibility或IDE**可能**成功，如果这条路可以走通，便不再需要GHOST安装。
![AHCI](/assets/img/posts/2021-01-11-install-xp-under-ubuntu/AHCI.jpg)

### 6.3 EasyBCD 不支持 windows XP。
[EasyBCD](http://neosmart.net/EasyBCD/)是Legacy BIOS下可以在Windows Vista/7/8/10的Boot loader中添加启动项的好工具。  
但其可以在Vista+中添加XP，并不能在XP下添加其他系统。  
![EasyBCD](/assets/img/posts/2021-01-11-install-xp-under-ubuntu/EasyBCD.png)


## Reference
[Recovering Ubuntu After Installing Windows](https://help.ubuntu.com/community/RecoveringUbuntuAfterInstallingWindows)  
[Boot-Repair](https://help.ubuntu.com/community/Boot-Repair)  
[Rufus 轻松创建USB启动盘](https://rufus.ie/zh_CN.html)  
[WinPE环境下WinNTSetup使用说明](https://my.oschina.net/hsds/blog/1841991)  
[Hardware 3D Acceleration (OpenGL and Direct3D 8/9)](https://www.virtualbox.org/manual/UserManual.html#guestadd-3d)  
[GNOME Partition Editor](https://gparted.org/display-doc.php?name=moving-space-between-partitions)  
