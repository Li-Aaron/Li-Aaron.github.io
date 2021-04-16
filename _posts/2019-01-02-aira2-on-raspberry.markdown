---
layout: post
title: 用树莓派 Raspberry Pi 远程下载 (aria2)
date: 2019-01-02 18:00:00.000000000 +08:00
excerpt: 用树莓派做远程下载机，可以在外出时方便下载，回家直接看电影。本文详细介绍如何在树莓派上安装并配置Aria2，配置网页服务器并部署AriaNg管理界面，获取公网IP并公网映射或实现内网穿透，以及配置硬盘挂载和SMB服务。
---

## 1. 系统安装

安装好系统的可以略过。  

>**2021.04.11更新**：目前ubuntu 20.04 LTS 已经支持 Raspberry Pi 4/3b， 可以直接从官方源下载：  
>[https://ubuntu.com/download/raspberry-pi](https://ubuntu.com/download/raspberry-pi) -- 下载页面  
>[https://ubuntu.com/tutorials/how-to-install-ubuntu-on-your-raspberry-pi#1-overview](https://ubuntu.com/tutorials/how-to-install-ubuntu-on-your-raspberry-pi#1-overview) -- 安装说明  
>感谢[chainsx](https://github.com/chainsx)  

这里选择的系统是[chainsx](https://github.com/chainsx)提供的64位系统[ubuntu-18.04-arm64](https://github.com/chainsx/ubuntu64-rpi/releases/tag/ubuntu-18.04-arm64)。  
这个系统相比官方的armhf系统优势在于配置简单，官方提供的ubuntu只默认支持Raspberry Pi 2，3B与3B+均需要根据官方的文档进行更改。  
且此系统默认的apt源是清华的源，国内下载速度快。  

SD卡烧录工具：[Win32DiskImager](https://sourceforge.net/projects/win32diskimager/files/latest/download)。  
以下说明均按照ubuntu系统，centos系统目前还没有尝试过。  

注：默认用户名为`ubuntu`

## 2. Aria2安装

```bash
sudo apt-get update
sudo apt-get install aria2
```


## 3. Aria2配置

### 3.1 Aria2配置文件

创建配置文件

```bash
mkdir -p ~/.config/aria2/
touch ~/.config/aria2/aria2.session
vim ~/.config/aria2/aria2.config
```

配置文件中设置如下（注意不要中文注释）：

```bash
# set your own path
dir=[yourpath]
disk-cache=32M
file-allocation=trunc
continue=true

max-concurrent-downloads=10

max-connection-per-server=16
min-split-size=10M
split=5
max-overall-download-limit=0
#max-download-limit=0
#max-overall-upload-limit=0
#max-upload-limit=0
disable-ipv6=false

save-session=~/.config/aria2/aria2.session
input-file=~/.config/aria2/aria2.session
save-session-interval=60


enable-rpc=true
rpc-allow-origin-all=true
rpc-listen-all=true
#rpc-secret=secret
#event-poll=select
#rpc-listen-port=6800


# for PT user please set to false
enable-dht=true
enable-dht6=true
enable-peer-exchange=true

# for increasing BT speed
listen-port=51413
#follow-torrent=true
#bt-max-peers=55
#dht-listen-port=6881-6999
#bt-enable-lpd=false
#bt-request-peer-speed-limit=50K
peer-id-prefix=-TR2770-
user-agent=Transmission/2.77
seed-ratio=0
#force-save=false
#bt-hash-check-seed=true
bt-seed-unverified=true
bt-save-metadata=true
bt-tracker=udp://62.138.0.158:6969/announce,udp://87.233.192.220:6969/announce,udp://111.6.78.96:6969/announce,udp://90.179.64.91:1337/announce,udp://51.15.4.13:1337/announce,udp://151.80.120.113:2710/announce,udp://191.96.249.23:6969/announce,udp://35.187.36.248:1337/announce,udp://123.249.16.65:2710/announce,udp://210.244.71.25:6969/announce,udp://78.142.19.42:1337/announce,udp://173.254.219.72:6969/announce,udp://51.15.76.199:6969/announce,udp://51.15.40.114:80/announce,udp://91.212.150.191:3418/announce,udp://103.224.212.222:6969/announce,udp://5.79.83.194:6969/announce,udp://92.241.171.245:6969/announce,udp://5.79.209.57:6969/announce,udp://82.118.242.198:1337/announce
```


这里避免使用root账户，如果使用了root账户或sudo命令，需要将`~/.config/aria2/`目录所有权更改为user，不然后续设置aria2自启动时在此目录下创建文件会fail。

```bash
chown -R user:user ~/.config/aria2/
```

测试aria2是否成功启动

```bash
aria2c --conf-path=~/.config/aria2/aria2.config
ps aux | grep aria2
```

若看到如下内容，则启动成功。

```
root      7710  0.0  0.0   2704   640 pts/0    S+   19:19   0:00 grep --color=auto aria2
ubuntu   21855  0.0  1.2  48600 11836 ?        Ss    2018   1:02 /usr/bin/aria2c --conf-path=/home/ubuntu/.config/aria2/aria2.config
```

### 3.2 开机自启动

创建systemctl service文件
```bash
sudo vim /lib/systemd/system/aria2.service
```
User,conf-path下换成自己的username
```ini
[Unit]
Description=Aria2 Service
After=network.target

[Service]
User=ubuntu
ExecStart=/usr/bin/aria2c --conf-path=/home/ubuntu/.config/aria2/aria2.config

[Install]
WantedBy=default.target
```

重载服务并设置开机启动

```bash
sudo systemctl daemon-reload
sudo systemctl enable aria2
sudo systemctl start aria2
sudo systemctl status aria2
```

看到如下文字证明启动成功(记住TCP port，AiraNg配置以及公网端口映射需要)
```
● aria2.service - Aria2 Service
   Loaded: loaded (/lib/systemd/system/aria2.service; enabled; vendor preset: enabled)
   Active: active (running) since Sat 2018-12-29 08:24:20 CST; 4 days ago
 Main PID: 21855 (aria2c)
   CGroup: /system.slice/aria2.service
           └─21855 /usr/bin/aria2c --conf-path=/home/ubuntu/.config/aria2/aria2.config

Dec 29 08:24:20 ubuntu systemd[1]: Started Aria2 Service.
Dec 29 08:24:20 ubuntu aria2c[21855]: 12/29 08:24:20 [NOTICE] IPv4 RPC: listening on TCP port 6800
Dec 29 08:24:20 ubuntu aria2c[21855]: 12/29 08:24:20 [NOTICE] IPv6 RPC: listening on TCP port 6800
Dec 29 08:25:21 ubuntu aria2c[21855]: 12/29 08:25:21 [NOTICE] Serialized session to '/home/ubuntu/.config/aria2/aria
```

## 4. AriaNg使用

aira2比较好的web管理界面当属AriaNg。
有网页版的[AriaNg](https://github.com/mayswind/AriaNg/releases)以及Windows桌面版的[AriaNg-Native](https://github.com/mayswind/AriaNg-Native/releases)，推荐使用网页版的，可以部署在Raspberry Pi上。

### 4.1 将AriaNg部署在Raspberry Pi上

#### 配置Tomcat服务器

首先需要一个web服务器，这里使用了Tomcat 7。（这里纯属个人喜好，用Apache或Nginx看个人需要）
```bash
wget https://www-eu.apache.org/dist/tomcat/tomcat-7/v7.0.92/bin/apache-tomcat-7.0.92.tar.gz
tar -xzvf apache-tomcat-7.0.92.tar.gz
mv apache-tomcat-7.0.92/ tomcat7
vim ./tomcat7/bin/catalina.sh
```

修改此启动配置文件，JAVA路径添加默认java路径，此配置中的java路径为`sudo apt-get install default-jre`安装的默认路径。

```bash
#   JAVA_HOME       Must point at your Java Development Kit installation.
#                   Required to run the with the "debug" argument.
JAVA_HOME=/usr/lib/jvm/java-11-openjdk-arm64

#   JAVA_OPTS       (Optional) Java runtime options used when any command
#                   is executed.
#                   Include here and not in CATALINA_OPTS all options, that
#                   should be used by Tomcat and also by the stop process,
#                   the version command etc.
#                   Most options should go into CATALINA_OPTS.
JAVA_OPTS="-server -Xms512m -Xmx1024m -XX:PermSize=600M -XX:MaxPermSize=600m -Dcom.sun.management.jmxremote"
```

修改服务器文件配置文件：
```bash
vim ./tomcat7/conf/server.xml
```

将
```xml
    <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
```

修改为
```xml
    <Connector port="80" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="443" />
```
即可使用默认http 80 端口

```bash
vim ./tomcat7/bin/start.sh
```
在浏览器中访问`http://your-ip/`即可打开Apache Tomcat的默认webpage了。（如下图）

![](/assets/images/2019-01-02-aira2-on-raspberry/tomcat.PNG)


#### 配置Tomcat自启动

```bash
vim /etc/rc.local
```
在文件中添加
```bash
sudo /home/ubuntu/tomcat7/bin/startup.sh
```

#### 配置AriaNg

```bash
wget https://github.com/mayswind/AriaNg/releases/download/1.0.0/AriaNg-1.0.0.zip
unzip AriaNg-1.0.0.zip -d aira-ng
sudo mv aira-ng ~/tomcat7/webapps/ROOT/
```

在浏览器中访问`http://your-ip/aira-ng`即可打开AriaNg了。

### 4.2 本地使用AriaNg

#### 网页版

直接下载[AriaNg](https://github.com/mayswind/AriaNg/releases)，打开`index.html`  
在AiraNg Settings中，选择RPC选项卡，将Aira2 RPC Address中IP地址和端口号修改为Raspberry Pi的IP地址以及前面status中的端口号（默认6800）。

#### 桌面版

直接下载[AriaNg-Native](https://github.com/mayswind/AriaNg-Native/releases)，打开，设置同上。

## 5. 公网端口映射

配置好Aria2以及AriaNg后，就可以将Raspberry Pi映射到公网上来进行远程下载了。配置公网的方式有以下几种：

### 5.1 DMZ主机

DMZ主机是目前绝大多数路由器都会带的功能，其功能就是实现将内网设备直接暴露在公网之上。

移动光猫配置页面（需要超级帐号）：

![](/assets/images/2019-01-02-aira2-on-raspberry/router_cmcc.PNG)

TP-Link配置页面：

![](/assets/images/2019-01-02-aira2-on-raspberry/router_tp.PNG)

### 5.2 端口映射

如果不想要把所有的端口都暴露在公网上（这样做风险较大），可以只将其中的几个端口映射到公网上。  
这里使用移动光猫的设置举例：

![](/assets/images/2019-01-02-aira2-on-raspberry/router_cmcc2.PNG)

这样配置好访问路由器的公网IP地址以及绑定的端口号就可以直接访问到Raspberry Pi的相应端口。

### 5.3 获取公网IP地址

访问Raspberry Pi还需要获取公网IP地址，而路由的公网IP地址有可能会变化。

可以访问
[https://www.ipchicken.com/](https://www.ipchicken.com/)
[https://www.iplocation.net/find-ip-address](https://www.iplocation.net/find-ip-address)
等网站获取本机的IP地址
这里提供一个（粗鄙）的Python获取IP地址发送邮件的脚本(Python 3.6， 2.7上没尝试过)

```python
import requests, re
import smtplib
from email.header import Header
from email.mime.text import MIMEText
from email.utils import parseaddr, formataddr

def _format_addr(s):
  name, addr = parseaddr(s)
  return formataddr((name, addr))

if __name__ == '__main__':
  # get webpage
  url = "https://www.ipchicken.com/"
  r = requests.get(url)

  # get IP address from webpage
  pattern = re.compile(r'((25[0-5]|2[0-4]\d|[0-1]\d{2}|[1-9]?\d)\.){3}(25[0-5]|2[0-4]\d|[0-1]\d{2}|[1-9]?\d)')
  matchObj = pattern.search(r.text)
  IpAddress = matchObj.group(0)

  # send email

  # email (modify by yourself)
  from_addr   = Your_email_address
  password    = Your_password
  to_addr     = ['Your_another_email_address']
  smtp_server = 'your.smtp.server'
  smtp_port   = 25

  # message
  msg = MIMEText(''+IpAddress, 'plain', 'utf-8')
  msg['Subject'] = Header('Raspberry Pi Ip Address', 'utf-8').encode()
  msg['From']    = _format_addr('Raspberry Pi <%s>' % from_addr)
  msg['To']      = _format_addr('Raspberry Pi Manager <%s>' % to_addr[0])

  # send
  try:
    server = smtplib.SMTP(smtp_server, smtp_port)
    server.login(from_addr, password)
    server.sendmail(from_addr, to_addr, msg.as_string())
    print("Send Email Success")
  except smtplib.SMTPException:
    print("Error: Unable to send Email")

  server.quit()
```

### 5.4 花生壳内网穿透

如果家中用的路由是二级路由，那么恭喜，无论是DMZ主机还是端口映射都没有用了。  
二级路由就是说你是在内网的内网中，如果路由器的公网地址和从[https://www.ipchicken.com/](https://www.ipchicken.com/)上获取的IP地址不一样就说明是二级路由。

这种也可以有个（不需要VPS的）方法解决，就是利用花生壳的内网穿透功能。目前这个功能需要6元钱。

[花生壳](https://hsk.oray.com/)官网上有详细的说明，这里就不赘述了。测试功能支持映射两个端口，这里需要将80和Aria2 RPC的端口（默认6800）映射出去即可。

花生壳会提供一个服务域名，这个域名就相当于公网IP地址。

### 5.5 利用Frp内网穿透

如果有条件（比如有一个VPS），可以通过[Frp](https://github.com/fatedier/frp/blob/master/README_zh.md)作反向代理实现内网穿透。  
安装参见[Frp 的 GitHub page](https://github.com/fatedier/frp/blob/master/README_zh.md)，有很详细的说明，这里不赘述了。  
服务器端使用frps 和配置文件frps.ini如下：  

```ini
[common]
bind_addr = 0.0.0.0
bind_port = 5443
log_file = ./frps.log
log_level = info
log_max_days = 3
# auth token
token = [your token]

max_pool_count = 50
tcp_mux = true
```

Raspberry Pi端使用frpc 和配置文件frpc.ini如下（ip填写服务器的ip，token和服务器端保持一致）：  
除了aria2的6800端口，还可以将其他端口如ssh，http映射到服务器ip上。  

```ini
[common]
server_addr = [your ip]
server_port = 5443
token = [your token]
tcp_mux = true

[aria2]
type = tcp
local_ip = 127.0.0.1
local_port = 6800
remote_port = 6800

[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 4022

[http]
type = tcp
local_ip = 127.0.0.1
local_port = 80
remote_port = 8080
```

## 6. 硬盘挂载（可选）

Raspberry Pi虽然可以用SD卡扩展存储，但空间毕竟小，若有旧台式机、笔记本硬盘可以套个硬盘盒接在Raspberry Pi上。  
3B+ USB口直接带2.5的硬盘或SSD是不需要外接电源的，3.5的没有测试过，但一般是需要的。  

手动挂载的方法如下：

```bash
sudo fdisk -l
```

```
Disk /dev/sda: 465.8 GiB, 500107862016 bytes, 976773168 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x000d2ae0

Device     Boot Start       End   Sectors   Size Id Type
/dev/sda1        2048 976769023 976766976 465.8G  7 HPFS/NTFS/exFAT
```


或
```bash
sudo blkid
```

```
/dev/mmcblk0p1: SEC_TYPE="msdos" UUID="9FF3-2ADB" TYPE="vfat" PARTUUID="c2909e2d-01"
/dev/mmcblk0p2: UUID="a407e17a-e170-45d8-8fb7-dd34fe646ab9" TYPE="ext4" PARTUUID="c2909e2d-02"
/dev/mmcblk0: PTUUID="c2909e2d" PTTYPE="dos"
/dev/sda1: LABEL="New Volume" UUID="848CC9268CC91398" TYPE="ntfs" PARTUUID="000d2ae0-01"
```

从中可以获取到硬盘的盘符`/dev/sda1`

```bash
# 根据自己的需求设置挂载目录，这里这么设置是为了后续的SMB共享
sudo mkdir ~/share/disk1
sudo mount /dev/sda1 ~/share/disk1
```

如果需要重启机器，则需要采用自动挂载的方式：  

```bash
sudo vim /etc/fstab
```

加入如下一条

```ini
UUID=848CC9268CC91398 /home/ubuntu/share/disk1 ntfs  defaults                0       3
```

执行如下命令验证

```bash
sudo mount -a
```


## 7. SMB服务（可选）

单纯的下载机是不够的，不能总是去插拔硬盘，而通过SMB服务可以用终端设备直接访问到Raspberry Pi。  

需要创建一个samba帐号

```bash
sudo touch /etc/samba/smbpasswd
smbpasswd -a ubuntu
```

再输入两次密码后即可创建账户

```bash
sudo vim /etc/samba/smb.conf
```

在文件末尾加入如下信息

```ini
[share]
  path = /home/ubuntu/share
  available = yes
  browseable = yes
  #public = yes
  writable = yes
```

修改读写权限

```bash
sudo chown -R ubuntu:ubuntu /home/ubuntu/share
sudo chmod -R 777 /home/ubuntu/share
```

重启samba服务

```bash
service smbd restart
```

即可进行连接测试（在运行窗口中输入`\\ip`即可访问）

![](/assets/images/2019-01-02-aira2-on-raspberry/smb_service.PNG)

## Reference
[树莓派3B+ 远程下载服务器（Aria2）](https://blog.csdn.net/kxwinxp/article/details/80288006) 作者：宁静致远kioye  
[抛弃迅雷，Aria2 新手入门](https://zhuanlan.zhihu.com/p/37021947) 作者：清顺  
[ubuntu开机自动挂载新硬盘](https://blog.csdn.net/iam333/article/details/17224115) 作者：_iAm333  