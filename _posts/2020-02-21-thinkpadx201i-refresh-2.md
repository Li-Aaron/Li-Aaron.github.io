---
layout: post
title: 老电脑焕发第二春，Thinkpad X201i复活记（续）
date: 2020-02-21 18:00:00.000000000 +08:00
description: 老电脑系列第二篇，给Thinkpad X201i刷BIOS白名单，更换无线网卡。
author: aaron-li
categories: [玩机攻略, 老电脑]
tags: [thinkpad, thinkpad x201i, refresh]
---

继[去年复活了老Thinkpad X201i]({{site.url}}/2019/11/thinkpadx201i-refresh/)之后，用了一段时间，无论是写文章还是调试网络还是比较方便的。

特别是linux系统弥补了主力机上装vbox带来的不便捷。

但是网络连接上还是带来了很大的不便，这一次要给这台小机器换个网卡。

## 1. 刷白名单

Lenovo 2016年前上市的机器都有bios白名单限制，导致很多硬件无法更换，而内置的这个[Intel 112BNHMW](https://ark.intel.com/content/www/cn/zh/ark/products/59480/intel-centrino-wireless-n-1000-single-band.html)仅支持2.4g wifi (802.11n)，在现在基本是千兆网的内网环境下非常吃力，而且因为老化？经常会断线。

之前加装的Mi USB wifi虽然不会经常断线，但也是仅支持2.4g，网速比较差。

Intel 112BNHMW 与 MI wifi

![112b-and-mi](/assets/img/posts/2020-02-21-thinkpadx201i-refresh-2/112b-and-mi.jpg)

为了支持更多硬件，我们需要将bios中的白名单去掉，这里参考了[【转帖】转发一个从国外论坛弄来的x201 1.40bios(白名单及SLIC2.1)](https://forum.51nb.com/thread-1366013-1-1.html)文章中的压缩包，为了方便保存和下载，这里重新打包压缩了一下。

白名单下载：[Lenovo.ThinkPad.X201_1.40-.6quj18us._lenovo21_WList.zip](https://github.com/Li-Aaron/Li-Aaron.github.io/releases/download/0.0.1/Lenovo.ThinkPad.X201_1.40-.6quj18us._lenovo21_WList.zip)

刷白名单的方式很简单，需要使用压缩包中的`WinPhlash.exe`，但是这个软件只能在windows下运行，鉴于我是一个ubuntu系统，这里需要一个usb winPE来辅助一下。

使用其中的32bit软件（在我的winPE下64bit有兼容性问题），按照压缩包中`!_readme`文件中的提示，在`Advanced`中仅勾选以下两项  
```
[x] Verify block after programming  
[x] Disable Axx swaping automatic detection (if present)  
```
并在`"DMI" tab`中选择`"Update": Select "Update the BIOS and not DMI"`

然后刷新bios即可，并等待程序自动重启。

## 2. 更换网卡

鉴于拆过一次机了，这里不再展示拆机图片，原机自带的网卡是半高PCIE，更换同接口支持802.11ac和bluetooth的[Intel7260HMW](https://ark.intel.com/content/www/cn/zh/ark/products/75439/intel-dual-band-wireless-ac-7260.html)，某宝有很多版本，购买的时候**不要**买带FRU
的联想专用（贵，已经刷掉白名单不必要）。

Intel 7260HMW

![7260hmw](/assets/img/posts/2020-02-21-thinkpadx201i-refresh-2/7260hmw.jpg)

Intel 7260HMW 与 Intel 112BNHMW

![7260-112b](/assets/img/posts/2020-02-21-thinkpadx201i-refresh-2/7260-112b.jpg)

更换重启后，自动识别了网卡与bluetooth。

![bluetooth-setting](/assets/img/posts/2020-02-21-thinkpadx201i-refresh-2/bluetooth-setting.png)

Wifi链路速度，连接的是5G wifi。

![wifi-link-speed](/assets/img/posts/2020-02-21-thinkpadx201i-refresh-2/wifi-link-speed.png)

Speedtest，终于不是感人的十位数了。

![speedtest-result](/assets/img/posts/2020-02-21-thinkpadx201i-refresh-2/speedtest-result.png)

## 3. 后记

网不卡了，不掉线了，舒服。

## Reference
[【转帖】转发一个从国外论坛弄来的x201 1.40bios(白名单及SLIC2.1)](https://forum.51nb.com/thread-1366013-1-1.html)  
[X201T完美更换intel 7260AC无线网卡](https://forum.51nb.com/thread-1666112-1-1.html)  
