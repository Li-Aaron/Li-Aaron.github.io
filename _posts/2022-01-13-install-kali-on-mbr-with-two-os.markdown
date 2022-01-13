---
layout: post
title: 在MBR (Legacy BIOS) 双系统下安装Kali实现三系统
date: 2022-1-14 20:00:00.000000000 +08:00
excerpt: 老电脑系列番外篇，给装好Ubuntu和Windows xp的Thinkpad X201i再安装Kali，其实安装Kali之后就实现了三系统，不过grub的启动顺序不是想要的，还是修复了一下。
---


最近想试试Kali系统，但是主力台式机上没有Wifi，于是又想起了这台[老电脑]({{site.url}}/2020/02/thinkpadx201i-refresh/)。

之前这台老电脑上已经装了两个系统：XP和Ubuntu，这次就讲一下怎么装三系统。

## 0. 准备工作

需要一个Ubuntu的安装U盘来修改磁盘分区（[Gparted](https://gparted.org/display-doc.php?name=moving-space-between-partitions)），以及修复引导（[Boot-Repair](https://help.ubuntu.com/community/Boot-Repair)）。
需要一个Kali的安装U盘来安装[Kali](https://www.kali.org/)。

用[Rufus](https://rufus.ie/zh_CN.html)生成Ubuntu/Kali的安装盘。（UltraISO生成的安装盘在Legacy BIOS下无法正常运行）
![Rufus-image-gen](/assets/images/2021-01-11-install-xp-under-ubuntu/rufus-ubuntu.png)

成功启动Live Ubuntu。
![Live Ubuntu](/assets/images/2021-01-11-install-xp-under-ubuntu/ubuntu-live.png)

注意：Rufus做Kali的安装盘需要用DD写入。
![Rufus-image-gen-Kali-DD](/assets/images/)


## 1. 磁盘分区

方法和Windows安装Ubuntu双系统是一样的，这里参考[在Ubuntu下安装Windows XP实现双系统]({{site.url}}/2021/01/install-xp-under-ubuntu/#tocAnchor-1-2)。

在ubuntu的Live CD下，选择`Try Ubuntu`。
![Try Ubuntu](/assets/images/2021-01-11-install-xp-under-ubuntu/try-ubuntu.png)

Ubuntu 18.04.05 Live CD自带gparted，`ALT+T`打开terminal后，键入
```Bash
sudo gparted
```

进入Gparted修改分区，移动Ubuntu分区到右侧，需要等待10+分钟（取决于文件的多少，使用的磁盘是HDD还是SSD）。
![Gparted-size](/assets/images/)

并新建一个ext4分区（也可以在Kali的安装过程中新建），完成后如图所示。
![Gparted-success](/assets/images/)


## 2. 安装Kali到第三个分区

这次直接装一个中文系统，Kali对中文的支持还不错。


## 3. 修复Ubuntu引导

这一步可以不修复，但我需要默认启动ubuntu而不是Kali，并且喜欢Ubuntu的grub（个人审美）。

如果不想换grub，可以直接更改grub。

要让Ubuntu系统回来，需要重建Ubuntu的grub。
这里推荐Ubuntu官方的教程[Recovering Ubuntu After Installing Windows](https://help.ubuntu.com/community/RecoveringUbuntuAfterInstallingWindows)。

由于Ubuntu还是可以启动的，可以直接在Ubuntu的Terminal下，键入如下命令：
```bash
sudo add-apt-repository ppa:yannubuntu/boot-repair
sudo apt-get update
sudo apt-get install -y boot-repair && boot-repair
```

在[Boot-Repair](https://help.ubuntu.com/community/Boot-Repair)工具的图形界面中选择`Recommand repair`。
![boot-repair界面](/assets/images/2021-01-11-install-xp-under-ubuntu/boot-repair.png)

等待程序完成后，重启，自动进入新的Ubuntu Grub引导界面。
![Ubuntu新的引导界面](/assets/images/2021-01-11-install-xp-under-ubuntu/ubuntu-grub.jpg)

可以看到三系统顺序发生了改变。

## 4. 后记

暗黑走起！
![Diablo](/assets/images/2021-01-11-install-xp-under-ubuntu/diablo-start.jpg)


## 5. 失败的尝试
### 5.1 Rufus生成Kali安装盘，使用ISO写入。
使用Rufus生成Kali的U盘安装盘：
![Rufus-image-gen-Kali-ISO](/assets/images/)

安装时提示。
![安装失败](/assets/images/)

## Reference
[Boot-Repair](https://help.ubuntu.com/community/Boot-Repair)
[Rufus 轻松创建USB启动盘](https://rufus.ie/zh_CN.html)
[GNOME Partition Editor](https://gparted.org/display-doc.php?name=moving-space-between-partitions)