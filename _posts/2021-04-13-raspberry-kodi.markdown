---
layout: post
title: 树莓派 Raspberry Pi 安装 Kodi 影音系统
date: 2021-04-13 18:00:00.000000000 +08:00
---

## 0. 前言
树莓派 Raspberry Pi 3B+ 也用了两年多了，期间一直主要用做 Ubuntu Server。  
最近 bundle upgrade jekyll 的时候，gcc 编译会out of memory，没有办法，换了台D2550的x86板子做Server。  
这台闲置下来，就想看看能做点什么，发现[Raspberry Pi](https://www.raspberrypi.org/software/operating-systems/)官网上有对Kodi支持的系统（[LibreElec](https://libreelec.tv/downloads_new/)）。  
遂下来玩一玩。

![raspberrypi](/assets/images/2021-04-13-raspberry-kodi/raspberrypi.jpg)  

## 1. 系统安装
[LibreElec 下载](https://libreelec.tv/downloads_new/)  
LibreElec 提供了LibreELEC USB-SD Creator这个App，下载烧录二合一。  
![createusbsd1](/assets/images/2021-04-13-raspberry-kodi/createusbsd1.png)
不过这个App内直接下载image的速度很慢，LibreElec还提供了manual download的地址。  
[LibreElec RASPBERRY PI 3 / 3+](https://libreelec.tv/downloads_new/raspberry-pi-3-3/)  
[LibreElec RASPBERRY PI 4](https://libreelec.tv/raspberry-pi-4/)  

下载后直接烧录SD卡就可以了。  

![createusbsd2](/assets/images/2021-04-13-raspberry-kodi/createusbsd2.png)

SD卡插回Raspberry Pi，开机：  

![Kodi_startup](/assets/images/2021-04-13-raspberry-kodi/Kodi_startup.gif)

首次开机需要选择语言，这里选择English避免出现乱码bug：  

![kodi_setting1](/assets/images/2021-04-13-raspberry-kodi/kodi_setting1.jpg)

设置Hostname：  

![kodi_setting2](/assets/images/2021-04-13-raspberry-kodi/kodi_setting2.jpg)

连接WIFI（或者使用有线连接）：  

![kodi_setting3](/assets/images/2021-04-13-raspberry-kodi/kodi_setting3.jpg)

打开SSH并设置密码：  

![Kodi_ssh](/assets/images/2021-04-13-raspberry-kodi/Kodi_ssh.gif)

查看系统信息：  

![kodi_sys](/assets/images/2021-04-13-raspberry-kodi/kodi_sys.jpg)


## 2. 配置中文
直接安装中文（或者在开机选择中文）会出现乱码BUG：  

![kodi_setting4](/assets/images/2021-04-13-raspberry-kodi/kodi_setting4.jpg)

在安装前，要在Setting -- Skin -- Fonts 把Skin Default改成Arial based，  
再切换成中文就不会出现乱码bug了：  

![Kodi_chinese](/assets/images/2021-04-13-raspberry-kodi/Kodi_chinese.gif)


## 3. 安装IPTV插件
很多时候Kodi是作为一个IPTV存在的，下面介绍下怎么在Kodi上安装IPTV插件。  

### 3.1 关于插件镜像源（目前未解决）

参考[ustc的XBMC/Kodi 镜像使用帮助](https://mirrors.ustc.edu.cn/help/xbmc.html)  
>可以使用此镜像服务器来访问 Kodi 官方插件库，避免因网络访问的问题而无法正常安装使用插件。需要编辑 Kodi 安装目录中的 addons/repository.xbmc.org/addon.xml 文件。将其中所有 http://mirrors.kodi.tv/ 替换为 http://mirrors.ustc.edu.cn/xbmc/。  

通过SSH找到了LibreElec的addon.xml文件  

```bash
LibreELEC:~ # find / -name addon.xml | grep repository.xbmc.org
/usr/share/kodi/addons/repository.xbmc.org/addon.xml
```

但这是一个只读文件，所以没有办法通过vi或则sed替换。  
找到好的方法再更新这里。  
好在Kodi里面还有一个LibreElec源，这个源安装插件实测还是很快的。  

![kodi_setting6](/assets/images/2021-04-13-raspberry-kodi/kodi_setting6.jpg)


### 3.2 安装IPTV并配置
一图流  

![Kodi_PVR](/assets/images/2021-04-13-raspberry-kodi/Kodi_PVR.gif)  

频道地址：  
```
https://raw.githubusercontent.com/EvilCult/iptv-m3u-maker/master/http/tv.m3u
```

重启后可以在电视中找到所有频道  

![kodi_iptv1](/assets/images/2021-04-13-raspberry-kodi/kodi_iptv1.jpg)  

随便打开一个频道  

![kodi_iptv2](/assets/images/2021-04-13-raspberry-kodi/kodi_iptv2.jpg)  

## 4. 导入影视库
Kodi最主要的功能还是看电影：  

![kodi_movie](/assets/images/2021-04-13-raspberry-kodi/kodi_movie.jpg)

下面讲一讲如何配置这个电影库  

### 4.1 SMB文件共享（Windows，Ubuntu）
**Windows**下共享文件非常简单  
右键--属性--共享--高级共享--共享此文件夹--确定  

![smb1](/assets/images/2021-04-13-raspberry-kodi/smb1.png)

之后可以用【*Windows的用户名*】和【*登录密码*】登录SMB服务了。

**Ubuntu**下需要安装smb服务, 在[用树莓派 Raspberry Pi 远程下载 (aria2)/SMB服务（可选）]({{site.url}}/2019/01/aira2-on-raspberry/#tocAnchor-1-16)文中也有提及

```bash
sudo apt-get install samba samba-common
mkdir /home/myshare
chmod 777 /home/myshare
vi /etc/samba/smb.conf
```
在smb.conf文件末尾添加
```ini
[share]
  path = /home/myshare
  available = yes
  browseable = yes
  writeable = yes
  #public = yes
```
public注释打开后则不需要输入密码  
```bash
touch /etc/samba/smbpasswd
smbpasswd -a {当前用户名}
{输入密码}
service smbd restart
```
之后可以用【*当前用户名*】和【*上面输入的密码*】登录SMB服务了。

### 4.2 NFS文件共享（群晖）
**群晖**推荐使用NFS分享文件，速度比SMB稳定，缺点是不支持加密授权，仅依靠IP段控制访问。  
在控制面板--文件服务--SMB/AFP/NFS中启用NFS服务：  

![nfs1](/assets/images/2021-04-13-raspberry-kodi/nfs1.png)

在共享文件夹--选择文件夹--编辑--NFS权限中新增NFS规则：  

![nfs2](/assets/images/2021-04-13-raspberry-kodi/nfs2.png)

IP地址填写【内网网段】，其他如下设置：  

![nfs3](/assets/images/2021-04-13-raspberry-kodi/nfs3.png)

即可通过内网IP地址直接访问NFS服务了。


### 4.3 Kodi 连接SMB/NFS
一图流演示Kodi连接NFS+设置刮削器为Local Information（通过下文的TinyMediaManager刮削）：

![Kodi_create_movie](/assets/images/2021-04-13-raspberry-kodi/Kodi_create_movie.gif)  

SMB添加相似，不再过多介绍。

### 4.4 通过TinyMediaManager刮削电影
[TinyMediaManager](https://www.tinymediamanager.org/)是一个非常优秀的Kodi刮削软件。  
注意要下载v3版本，v4版本免费只能刮削50个电影。  

![TMM](/assets/images/2021-04-13-raspberry-kodi/TMM.png)

在电影--媒体库目录中添加电影数据源后，更新媒体库即可获得所有的电影：  

![TMM2](/assets/images/2021-04-13-raspberry-kodi/TMM2.png)

选择影片并刮削：  

![TMM3](/assets/images/2021-04-13-raspberry-kodi/TMM3.png)

在刮削器中选择对应的info：  

![TMM4](/assets/images/2021-04-13-raspberry-kodi/TMM4.png)

一些建议设置（中文版）：  
电影--刮削器--元数据刮削 选择themobiedb.org，并在host文件中添加  
```
13.225.62.15 api.themoviedb.org
```
电影--刮削器--NFO设置  

![TMMsetting1](/assets/images/2021-04-13-raspberry-kodi/TMMsetting1.png)

电影--图片--艺术图片文件名  

![TMMsetting2](/assets/images/2021-04-13-raspberry-kodi/TMMsetting2.png)

这样可以避免同一文件夹下多个电影之间产生冲突。


## Reference
[NAS 共享访问协议 — NFS、SMB、FTP、WebDAV 各有何优势？](https://cloud.tencent.com/developer/article/1651445)  
[用树莓派 Raspberry Pi 远程下载 (aria2)]({{site.url}}/2019/01/aira2-on-raspberry/)  