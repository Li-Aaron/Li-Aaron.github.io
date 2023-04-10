---
layout: post
title: 在MBR (Legacy BIOS) 双系统下安装Kali实现三系统
date: 2022-1-13 19:00:00.000000000 +08:00
excerpt: 老电脑系列番外篇，给装好Ubuntu和Windows xp的Thinkpad X201i再安装Kali，其实安装Kali之后就实现了三系统，不过grub的启动顺序不是想要的，还是修复了一下。
---


最近想试试Kali系统，但是主力台式机上没有Wifi，于是又想起了这台[老电脑]({{site.url}}/2019/11/thinkpadx201i-refresh/)。

之前这台老电脑上已经装了两个系统：XP和Ubuntu，这次就讲一下怎么装三系统。

## 0. 准备工作

需要一个Ubuntu的安装U盘来修改磁盘分区（[Gparted](https://gparted.org/display-doc.php?name=moving-space-between-partitions)），以及修复引导（[Boot-Repair](https://help.ubuntu.com/community/Boot-Repair)）。  
需要一个Kali的安装U盘来安装[Kali](https://www.kali.org/)。  

用[Rufus](https://rufus.ie/zh_CN.html)生成Ubuntu/Kali的安装盘。（UltraISO生成的安装盘在Legacy BIOS下无法正常运行）  

![Rufus-image-gen](/assets/images/2021-01-11-install-xp-under-ubuntu/rufus-ubuntu.png)  

成功启动Live Ubuntu。  
![Live Ubuntu](/assets/images/2021-01-11-install-xp-under-ubuntu/ubuntu-live.png)  

注意：Rufus做Kali的安装盘需要用DD写入。  
![Rufus-image-gen-Kali-DD](/assets/images/2022-01-13-install-kali-on-mbr-with-two-os/rufus-kali.png)  


## 1. 磁盘分区

方法和Windows安装Ubuntu双系统是一样的，这里参考[在Ubuntu下安装Windows XP实现双系统]({{site.url}}/2021/01/install-xp-under-ubuntu/#tocAnchor-1-2)。  

在ubuntu的Live CD下，选择`Try Ubuntu`。  

![Try Ubuntu](/assets/images/2021-01-11-install-xp-under-ubuntu/try-ubuntu.png)  

Ubuntu 18.04.05 Live CD自带gparted，`ALT+T`打开terminal后，键入  
```Bash
sudo gparted
```

进入Gparted修改分区，移动Ubuntu分区到右侧，需要等待10+分钟（取决于文件的多少，使用的磁盘是HDD还是SSD）。  

![Gparted-size](/assets/images/2022-01-13-install-kali-on-mbr-with-two-os/gparted1.png)

并新建一个ext4分区（也可以在Kali的安装过程中新建）。  

![Gparted-size2](/assets/images/2022-01-13-install-kali-on-mbr-with-two-os/gparted2.png)

点击确定，等待分区结束。  

![Gparted-success](/assets/images/2022-01-13-install-kali-on-mbr-with-two-os/gparted3.png)


## 2. 安装Kali到第三个分区

由于在笔记本安装没有截图，这里在Virtural box虚拟机重新安装一次。  

将Kali安装盘插入USB，选择Start installer，（注意：Live不能直接安装）  

![kali-install-1](/assets/images/2022-01-13-install-kali-on-mbr-with-two-os/kali1.png)

这次直接装一个中文系统，Kali对中文的支持还不错。  

![kali-install-2](/assets/images/2022-01-13-install-kali-on-mbr-with-two-os/kali2.png)

手动选择硬盘分区。  

![kali-install-3](/assets/images/2022-01-13-install-kali-on-mbr-with-two-os/kali3.png)

选择刚刚gparted中建立的第三个分区。  

![kali-install-4](/assets/images/2022-01-13-install-kali-on-mbr-with-two-os/kali4.png)

设置文件系统为Ext4，挂载点为`/`。  
注：可启动标志为关，可启动标志要留给windows分区。  

![kali-install-5](/assets/images/2022-01-13-install-kali-on-mbr-with-two-os/kali5.png)

结束分区设定。  

![kali-install-6](/assets/images/2022-01-13-install-kali-on-mbr-with-two-os/kali6.png)

由于我的内存够用，这里不额外设置swap分区。  

![kali-install-7](/assets/images/2022-01-13-install-kali-on-mbr-with-two-os/kali7.png)

确定将改动写入磁盘。  

![kali-install-8](/assets/images/2022-01-13-install-kali-on-mbr-with-two-os/kali8.png)

等待系统安装完成。  

![kali-install-9](/assets/images/2022-01-13-install-kali-on-mbr-with-two-os/kali9.png)

安装Kali的Grub启动器。  

![kali-install-10](/assets/images/2022-01-13-install-kali-on-mbr-with-two-os/kali10.png)

安装完成后，Kali的Grub包含三个系统。  

![kali-grub](/assets/images/2022-01-13-install-kali-on-mbr-with-two-os/kali-grub.png)



## 3. 修改启动顺序
目的：将默认启动改回Ubuntu。  

### 3.1 直接修改Kali Grub
如果不想换grub，可以直接更改grub中的启动顺序。  

```Bash
sudo vim /etc/default/grub
```
将`GRUB_DEFAULT`修改为3（对应上面Kali-Grub中第四项Ubuntu）。  

![kali-grub-edit](/assets/images/2022-01-13-install-kali-on-mbr-with-two-os/kali-grub-edit.png)

```Bash
sudo update-grub2
```

![kali-grub-edit2](/assets/images/2022-01-13-install-kali-on-mbr-with-two-os/kali-grub-edit2.png)

修改后默认启动项改变。  

![kali-grub-edit3](/assets/images/2022-01-13-install-kali-on-mbr-with-two-os/kali-grub-edit3.png)



### 3.2 Boot-repair修复Ubuntu Grub
喜欢Ubuntu的grub话，可以使用[Boot-Repair](https://help.ubuntu.com/community/Boot-Repair)工具重建Ubuntu的grub。  

这里推荐Ubuntu官方的教程[Recovering Ubuntu After Installing Windows](https://help.ubuntu.com/community/RecoveringUbuntuAfterInstallingWindows)。

也可以参考之前的文章[在Ubuntu下安装Windows XP实现双系统]({{site.url}}/2021/01/install-xp-under-ubuntu/#tocAnchor-1-5)。  

由于Ubuntu还是可以启动的，可以直接在Ubuntu的Terminal下，键入如下命令：  
```bash
sudo add-apt-repository ppa:yannubuntu/boot-repair
sudo apt-get update
sudo apt-get install -y boot-repair && boot-repair
```

在[Boot-Repair](https://help.ubuntu.com/community/Boot-Repair)工具的图形界面中选择`Recommand repair`。  

![boot-repair界面](/assets/images/2021-01-11-install-xp-under-ubuntu/boot-repair.png)  

等待程序完成后，重启，自动进入新的Ubuntu Grub引导界面。  

![Ubuntu新的引导界面](/assets/images/2022-01-13-install-kali-on-mbr-with-two-os/ubuntu-grub.png)

## 4. 后记

可以开始体验Kali OS拉。  

![kali-os](/assets/images/2022-01-13-install-kali-on-mbr-with-two-os/kali-os.png)

![kali-os2](/assets/images/2022-01-13-install-kali-on-mbr-with-two-os/kali-os2.png)

## 5. 失败的尝试
### 5.1 Rufus生成Kali安装盘，使用ISO写入。

使用Rufus生成Kali的U盘安装盘选择了ISO写入，安装时出现CDROM无法挂载的情况。

![cdrom-detect_retry_0](/assets/images/2022-01-13-install-kali-on-mbr-with-two-os/cdrom-detect_retry_0.png)

进入Shell发现`/dev/sdb1`被挂载到`/media`上，移除这个挂载。

![debian-installer_shell-plugin_0.png](/assets/images/2022-01-13-install-kali-on-mbr-with-two-os/debian-installer_shell-plugin_0.png)

CDROM可以挂载了。

![cdrom-detect_success_0](/assets/images/2022-01-13-install-kali-on-mbr-with-two-os/cdrom-detect_success_0.png)

但是无法从CDROM中读取数据。

![retriever_cdrom_error_0.png](/assets/images/2022-01-13-install-kali-on-mbr-with-two-os/retriever_cdrom_error_0.png)

无法解决。

## Reference
[Kali](https://www.kali.org/)
[Boot-Repair](https://help.ubuntu.com/community/Boot-Repair)  
[Rufus 轻松创建USB启动盘](https://rufus.ie/zh_CN.html)  
[GNOME Partition Editor](https://gparted.org/display-doc.php?name=moving-space-between-partitions)  

## Useful Links
[Kali Linux 源使用帮助](https://mirrors.ustc.edu.cn/help/kali.html)  
[wpa-dictionary](https://github.com/conwnet/wpa-dictionary)  
[Tuna Mirror](https://mirrors.tuna.tsinghua.edu.cn/)  
[Kali Tuna Mirror Download](https://mirrors.tuna.tsinghua.edu.cn/kali-images/current/kali-linux-2021.4a-live-amd64.iso)  