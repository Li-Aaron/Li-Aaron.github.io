---
layout: post
title: Surface pro 安装 android x86/chrome OS
date: 2020-03-19 18:00:00.000000000 +08:00
---

收拾箱子的时候，发现老surface pro在箱底好多年了，这款14年买的初代surface pro已经不能满足现在windows 10的需求了，简直卡到爆炸。  
但是Linux系统在[去年复活了老Thinkpad X201i](_site_/2019/11/thinkpadx201i-refresh/)之后，已经有一台了，再多一台反而累赘。

![surface](/assets/images/2020-03-19-surface-pro-android/surface.jpg)

因为是触屏本首先想到的是安装android x86体验一下安卓平板，现在开始搞起。

## 1. 准备工作
到[官网](https://www.android-x86.org/)去下载Android x86的镜像，由于不知道适配的情况，这里分别下载了Android 8.1/9.0。  
（在尝试的时候8.1使用的是r3，目前已更新到r4)  
链接如下:  
[android-x86_64-8.1-r4](https://osdn.net/projects/android-x86/downloads/69704/android-x86_64-8.1-r4.iso/)  
[android-x86_64-9.0-r1](https://osdn.net/projects/android-x86/downloads/71931/android-x86_64-9.0-r1.iso/)  

使用UltraISO将image写入U盘，准备工作就结束了。

## 2. U盘启动
Surface Pro的U盘启动在Microsoft的网站上有详细的说明，反正就那么三个实体按键都用上了。  
[How to use Surface UEFI](https://support.microsoft.com/en-us/help/4023531/surface-how-to-use-surface-uefi)  
[Boot Surface from a USB device](https://support.microsoft.com/en-us/help/4023511/surface-boot-surface-from-a-usb-device)  

首先按住音量上开机，开机后进入UEFI，将TPM和Secure boot都关闭。
![uefi](/assets/images/2020-03-19-surface-pro-android/uefi.jpg)

这时Surface Pro的启动界面会变成丑陋的红色（只能忍了）
![redscreen](/assets/images/2020-03-19-surface-pro-android/redscreen.jpg)

然后按住音量下开机，系统会自动寻找U盘引导启动，这时便可以进入Android x86 Uefi Grub了。
![android-grub](/assets/images/2020-03-19-surface-pro-android/android-grub.jpg)

## 3. Android x86系统安装
在安装的过程中遇到了一些问题，首先正常安装Android系统后开机仍然会自动进入Uefi（无系统状态）。

这种情况是我在安装过程中，将所有的磁盘分区都删掉了，尝试各种办法没有解决，最后安装了一次ubuntu，再安装Android系统反而成功了，主要是ubuntu在安装的时候生成了一个vfat格式的分区，在安装android时不要删除这个vfat分区，直接安装在ext4分区就正常了。

安装ubuntu  
![ubuntu-grub](/assets/images/2020-03-19-surface-pro-android/ubuntu-grub.jpg)
安装成功后，重启系统，安装Android X86  
![ubuntu-login](/assets/images/2020-03-19-surface-pro-android/ubuntu-login.jpg)
选择安装在ext4分区  
![android-install](/assets/images/2020-03-19-surface-pro-android/android-install.jpg)
一路回车到底  
![android-install-finish](/assets/images/2020-03-19-surface-pro-android/android-install-finish.jpg)
现在可以体验Android系统了。  
![android-openscreen](/assets/images/2020-03-19-surface-pro-android/android-openscreen.jpg)
![android-webpage](/assets/images/2020-03-19-surface-pro-android/android-webpage.jpg)


## 4. Chrome OS系统安装
Android x86在surface上还是有诸多bug的，比如有时会莫名的旋转屏幕，有时触屏失灵，而且能安装的软件有限，对arm软件的支持也不是特别好。

我打算再安装一个Chrome OS试一下。由于官方的Chrome OS安装比较繁琐，这里用了[CloudReady OS](http://www.neverware.com/freedownload)。  
CloudReady制作U盘启动盘比较简单，下载Windows下的[USB maker](https://usb-maker-downloads.neverware.com/stable/cloudready-free/cloudready-usb-maker.exe)，插入U盘直接等待制作完成就可以了。  

U盘启动的方式一样，CloudReady 会直接Boot到Live OS中供体验，我们可以在Live OS中选择install OS，等待一段时间后Surface会自动关机，这时拔出U盘启动即可。  
![chromeos](/assets/images/2020-03-19-surface-pro-android/chromeos.jpg)
现在可以体验Chrome OS了。  
![chromeos-webpage](/assets/images/2020-03-19-surface-pro-android/chromeos-webpage.jpg)

## 5. 系统比较
关于这两种系统的三个版本的优缺点，进行一下简单的比较  
|项目|Android 8.1|Android 9.0|CloudReady|
|--|--|--|--|
|启动器|点击任务栏的图标会弹出不对应的应用|窗口最大化时崩溃|正常使用|
|安装应用|Google Play/三方市场/apk安装|Google Play/三方市场/apk安装|Chrome Web Store|
|旋转屏幕|有时好用|有时好用|需要设置|
|触屏功能|时好时坏|正常|正常|
|Type Cover|支持|支持|Chrome OS的键位与Windows键盘不同|
|屏幕键盘|支持|支持|需要设置/外接键盘时仍会弹出|


Android X86系统相对粗糙一些，UI并不怎么好看，而Chrome OS基本就是个浏览器。  
而且无论是Google Play还是Chrome Web Store对于国内网络环境都不是很友好。


## Reference
[android-x86.org](https://www.android-x86.org/)
[How to use Surface UEFI](https://support.microsoft.com/en-us/help/4023531/surface-how-to-use-surface-uefi)  
[Boot Surface from a USB device](https://support.microsoft.com/en-us/help/4023511/surface-boot-surface-from-a-usb-device)  
[CloudReady OS](http://www.neverware.com/freedownload)