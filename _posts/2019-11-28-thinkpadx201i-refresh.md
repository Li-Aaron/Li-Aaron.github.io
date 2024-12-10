---
layout: post
title: 老电脑焕发第二春，Thinkpad X201i复活记
date: 2019-11-28 18:00:00.000000000 +08:00
description: 老电脑系列第一篇，介绍Thinkpad X201i拆机，清灰，更换风扇、硬盘、电池、以及增加内存的过程。
author: aaron-li
categories: [玩机攻略, 老电脑]
tags: [thinkpad, thinkpad x201i, refresh]  
---

最近搞网络设备的时候发现手头没有什么趁手的机器，surface pro放在腿上特别扭，又想躺在床上不用台式机，就想起前阵子给父上买了个小米Air，淘汰下来的这台老Thinkpad X201i了。

![机器外观](/assets/img/posts/2019-11-28-thinkpadx201i-refresh/front.jpg)

这台机器已经进不去系统了，在win7界面打转，电池也撑不了5分钟，风扇也哗啦啦的响。需要更换电池风扇，增加内存，更换SSD。

## 1. 拆机换风扇

首先是风扇，Thinkpad 都预留了更换内存、硬盘的开口，但更换风扇需要将主板整体拆下来。

![背面图片](/assets/img/posts/2019-11-28-thinkpadx201i-refresh/back.jpg)

要想拆主板，首先要拆键盘，Thinkpad 背面螺丝标注的还是很好的，把主板和键盘的螺丝都一次性拧下来，顺便把硬盘也拆掉。

![拆掉键盘](/assets/img/posts/2019-11-28-thinkpadx201i-refresh/remove_keyboard.jpg)

拆掉键盘和触摸板后，注意触摸板要先拆掉接线，主板就露出来了，X201i的主板比当年的E系列和SL系列做工看上去好一些，还包了一层防尘纸。

![暴力清灰](/assets/img/posts/2019-11-28-thinkpadx201i-refresh/remove_touchpad.jpg)

灰还是挺大的，懒人直接上吸尘器。

![主板螺丝](/assets/img/posts/2019-11-28-thinkpadx201i-refresh/remove_pcie.jpg)

继续拆。

![主板](/assets/img/posts/2019-11-28-thinkpadx201i-refresh/motherboard.jpg)

主板下来了，可以拆风扇了。

![散热片](/assets/img/posts/2019-11-28-thinkpadx201i-refresh/heatsink.jpg)

风扇拧下来，用吸尘器把风口好好吸一吸，之前都已经糊上了粘粘的东西。

![CPU](/assets/img/posts/2019-11-28-thinkpadx201i-refresh/cpu_chipset.jpg)

Intel的Chipset。

![内存条](/assets/img/posts/2019-11-28-thinkpadx201i-refresh/memory_chips.jpg)

直接把加装的内存条放上去，这里是用samsung 1333的ddr3，两条都是2G。

换上副厂的风扇（现在TB上也只有这种了），安装回去，开机没有声音了，清爽了许多。

## 2. 换硬盘，电池，网卡

把一块淘汰下来的128G samsung ssd换上去。

拆下来的硬盘盒两侧还有防震硅胶。

![硬盘](/assets/img/posts/2019-11-28-thinkpadx201i-refresh/ssd.jpg)

电池也换成副厂的。

![电池](/assets/img/posts/2019-11-28-thinkpadx201i-refresh/battery.jpg)

安装Ubuntu 18.04系统，开机基本秒开。

![Ubuntu界面](/assets/img/posts/2019-11-28-thinkpadx201i-refresh/ubuntu.jpg)


原来自带的网卡已经不太好用了，目前没找到能替换的，先插一个MI usb wifi。

![网卡](/assets/img/posts/2019-11-28-thinkpadx201i-refresh/networkcard.jpg)


现在可以正常使用了。

![网页](/assets/img/posts/2019-11-28-thinkpadx201i-refresh/blog.jpg)

## 3. 后记

用VLC打开视频，发现720P以上卡顿就很严重了，不过我的目的是用来远程调网络，顺便聊个天什么的，需求已经基本满足了，12寸的本本到现在还是很轻巧。

![VLC](/assets/img/posts/2019-11-28-thinkpadx201i-refresh/video.jpg)

![wireshark](/assets/img/posts/2019-11-28-thinkpadx201i-refresh/wireshark.jpg)
