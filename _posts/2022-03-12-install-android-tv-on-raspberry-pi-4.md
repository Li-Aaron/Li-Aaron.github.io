---
layout: post
title: 树莓派 Raspberry Pi 安装 Android TV 与遥控器键位修改
date: 2022-3-12 08:30:00.000000000 +08:00
description: 本文介绍在树莓派上安装Android TV的方法，一些基本的ADB指令，以及遥控器键位修改的方法。
author: aaron-li
categories: [玩机攻略, 树莓派]
tags: [android, raspberry pi, linageos, adb]
---

## 0. 前言
老婆大人一直不肯在书桌办公，原因是书桌没有电视看，而手头又没有什么高性能的电视盒子接我的显示器。  

前一阵子把树莓派 Raspberry Pi 3B+ 换成了 4B，这下可以在上面装Android TV了，赶快搞起来让老婆在每个屋子都有电视看。  

![](/assets/img/posts/2022-03-12-install-android-tv-on-raspberry-pi-4/1.JPG)  


## 1. 安装Android TV
这部分其实网上也有一些教程了，写一下方便别人也是方便自己。  

首先我选择的是[konstakang](https://konstakang.com/)提供的[基于LineageOS 18.1的Android TV 11操作系统](https://konstakang.com/devices/rpi4/LineageOS18-ATV/)。  

Image可通过其[教程](https://konstakang.com/devices/rpi4/LineageOS18-ATV/)下载。  

或直接从[https://www.androidfilehost.com/?fid=17825722713688273838](https://www.androidfilehost.com/?fid=17825722713688273838)下载。

用[Rufus](https://rufus.ie/zh_CN.html)烧录到SD卡。  

![](/assets/img/posts/2022-03-12-install-android-tv-on-raspberry-pi-4/1646976296850_2.png)  


烧录后SD卡中全部分区为7G多。  

![](/assets/img/posts/2022-03-12-install-android-tv-on-raspberry-pi-4/1646976315095_3.png)  

剩余的SD卡空间可通过DiskGenius，或通过其教程提供的Tool扩展。

如图所示为DiskGenius扩展分区的方式，在userdata分区上右键选择调整分区容量。

![](/assets/img/posts/2022-03-12-install-android-tv-on-raspberry-pi-4/1646976340563_4.png)  

![](/assets/img/posts/2022-03-12-install-android-tv-on-raspberry-pi-4/1646976362454_5.png)  

将SD卡装回树莓派即可启动Android TV 11。

首次启动时会有一些初始化选项，我们需要至少外接一个键盘来完成后续操作。

## 2. 一些初始化配置

Konstakang的[教程](https://konstakang.com/devices/rpi4/LineageOS18-ATV/)中有关于初始化的部分，因为是英文的，这里简单讲一下。  

键盘按键对应功能：
```cmd
F1 = 主页
F2 = 返回
F3 = 多任务
F4 = 菜单
F5 = 待机
F11 = 音量-
F12 = 音量+
```

### 语言设置

![](/assets/img/posts/2022-03-12-install-android-tv-on-raspberry-pi-4/lang.gif)  

### 开发者模式

![](/assets/img/posts/2022-03-12-install-android-tv-on-raspberry-pi-4/dev.gif)  

### 打开LAN ADB与SSH

![](/assets/img/posts/2022-03-12-install-android-tv-on-raspberry-pi-4/rpi.gif)  

### 隐私设置（安装第三方软件）

![](/assets/img/posts/2022-03-12-install-android-tv-on-raspberry-pi-4/privacy.gif)  

### SSH登录的方法（取自[教程](https://konstakang.com/devices/rpi4/LineageOS18-ATV/)）：

安卓没有带密码的用户账户，需要通过ADB获取private rsa key，通过rsa key登录ssh。

```bash
adb connect 192.168.1.xxx
adb root
adb pull /data/ssh/ssh_host_rsa_key my_private_key
ssh -i my_private_key root@192.168.1.xxx
```
### 时区设置

这个系统貌似有BUG，设置时区无法保存，可以通过如下命令设置。
```base
adb shell "setprop persist.sys.timezone Asia/Shanghai"
```

测试一下时区：
```bash
adb shell date
```

## 3. 基本的ADB指令

这一段提供一些基本的ADB指令
```bash
# 一般adb出现问题的时候强制关掉server
adb kill-server

# 获取adb root权限
adb root

# 通过LAN连接客户端
adb connect 192.168.1.xxx

# 安装xxx.apk
adb install xxx.apk

# 覆盖安装xxx.apk (一般升级用)
adb install -r xxx.apk

# 启动shell (开启ssh后可以直接用ssh)
adb shell

# 获取客户端的文件
adb pull /xxx/xx.xx

# 向客户端上传文件
adb push xx.xx /xxx/xx.xx
```

## 4. 遥控器键位修改

网购了个买了个USB 2.4G的遥控器，结果返回键不好用，这下怎么办。

下面介绍下如何修改遥控器键位在Android TV上的映射。

### 获取遥控器按键输入值

首先通过如下命令获取遥控器按键的输入：

```bash
adb root
adb shell
getevent
```

按一下遥控器的按键，查看输入的结果：

```bash
add device 7: /dev/input/event9
  name:     "YSTEK Laser Pen_V2.1 System Control"
add device 8: /dev/input/event8
  name:     "YSTEK Laser Pen_V2.1 Consumer Control"
add device 9: /dev/input/event7
  name:     "YSTEK Laser Pen_V2.1 Mouse"
add device 10: /dev/input/event6
  name:     "YSTEK Laser Pen_V2.1"

/dev/input/event6: 0004 0004 00070029
/dev/input/event6: 0001 0001 00000001
/dev/input/event6: 0000 0000 00000000
/dev/input/event6: 0004 0004 00070029
/dev/input/event6: 0001 0001 00000000
/dev/input/event6: 0000 0000 00000000
```

我们可以通过以上结果确认两个信息：  
设备为`/dev/input/event6`，即`"YSTEK Laser Pen_V2.1"`。  
返回键按钮的值为`0x070029`。  

### 获取遥控器对应配置文件

再通过如下命令  

```bash
adb shell # 没有退出shell可以不输入
dumpsys input
```

获取到如下信息：  

```
  7: YSTEK Laser Pen_V2.1
      Classes: 0x80000083
      Path: /dev/input/event6
      Enabled: true
      Descriptor: 556e197041405d0e6cf75f8a91bd36753588615b
      Location: usb-0000:01:00.0-1.4/input0
      ControllerNumber: 0
      UniqueId: 
      Identifier: bus=0x0003, vendor=0x276d, product=0x1101, version=0x0111
      KeyLayoutFile: /vendor/usr/keylayout/Generic.kl
      KeyCharacterMapFile: /system/usr/keychars/Generic.kcm
      ConfigurationFile: 
      HaveKeyboardLayoutOverlay: false
      VideoDevice: <none>
```

我们可以通过以上结果确认该设备对应的Layout文件为`/vendor/usr/keylayout/Generic.kl`

### 修改遥控器配置文件

退出adb shell，通过  
```bash
adb pull /vendor/usr/keylayout/Generic.kl
```
命令获取该文件，并在文件中增加  
```text
key usage 0x070029 BACK
```

再上传文件并重启  
```bash
adb remount
adb push Generic.kl /vendor/usr/keylayout/Generic.kl
adb reboot
```

这样，返回键的映射就设置好了。

其他按键的名称可通过[https://source.android.com/devices/input/key-layout-files?hl=zh-cn](https://source.android.com/devices/input/key-layout-files?hl=zh-cn)查询，或参考`/vendor/usr/keylayout/Generic.kl`文件中定义。

## 5. 后记

老婆大人觉得不错。

![](/assets/img/posts/2022-03-12-install-android-tv-on-raspberry-pi-4/2.JPG)  

![](/assets/img/posts/2022-03-12-install-android-tv-on-raspberry-pi-4/3.JPG)  

## 2024更新

LineageOS 19之后HW decoding好了，但是Kodi播放4K的视频还是爆卡，可以播1080p的。

LinageOS自带的ntp也不能正常同步，可以通过如下命令更改NTP服务器。

```
adb root
adb shell "settings put global ntp_server ntp.ntsc.ac.cn"
```

修改之后重新开关一下自动同步时间就好了。

## Reference
[LineageOS 18.1 Android TV (Android 11)](https://konstakang.com/devices/rpi4/LineageOS18-ATV/)  
[Android 调试桥 (adb)](https://developer.android.com/studio/command-line/adb?hl=zh-cn)   
[按键布局文件](https://source.android.com/devices/input/key-layout-files?hl=zh-cn)   
[adb与遥控器按键相关的指令-爱代码爱编程](https://icode.best/i/88550535743279)   
