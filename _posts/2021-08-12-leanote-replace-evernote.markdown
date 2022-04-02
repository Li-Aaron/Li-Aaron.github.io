---
layout: post
title: 告别印象笔记，使用Leanote自建云笔记
date: 2021-08-12 10:00:00.000000000 +08:00
excerpt: 印象笔记广告越来越多，即便VVVIP会员也免不了遭受广告的荼毒，免费用户限制又颇多。本文介绍使用Leanote自建云笔记服务取代印象笔记的方法，一些遇到的问题和解决办法，以及Leanote相比印象笔记的优势与一些不足之处。
---

## 0. 前言

印象笔记广告越来越多，即便VVVIP会员也免不了遭受广告的荼毒。而且这些年来markdown编辑器一直很烂，代码复制粘贴经常乱套。遂准备将笔记切换到自建Leanote（蚂蚁笔记）。

本文主要介绍使用Leanote自建云笔记服务取代印象笔记的方法，包括Docker部署，后台设置，笔记导入，反向代理与内网穿透，以及数据备份的方法。并针对Windows客户端无法连接代理问题，Ubuntu启动器添加图标问题给出解决方案。

Leanote网页端：

![leanote1](/assets/images/2021-08-12-leanote-replace-evernote/leanote1.png)

Leanote（自建服务器）相比于印象笔记**优缺点**如下：

**优点**：

 - 不限制登录设备数量
 - 不限制存储空间
 - 没有广告
 - 代码排版优秀
 - Markdown好用（当然毕竟不如vscode）
 - 服务器运行内存小（对比为知笔记自建）
 - 部署相对简单（对比WebDev+github+vscode等各种魔法组合）
 - 开源，geek可自己定制

**缺点**：

 - 需要服务器成本、域名成本
 - 国内建站需要备案访问
 - 内网环境外部访问麻烦
 - 开源社区维护热情欠佳，github上issue很多都没人管
 - 桌面客户端同步稳定性差
 - 桌面客户端不支持代理访问（文中有解决方案）

横向对比了一下发现并没有十全十美的笔记，就决定用它了。

> **注**：Leanote官方的笔记托管过了试用期就**收费**，还有过服务器**丢图片**的情况，不建议使用。

## 1. 部署Leanote服务器
虽然网上有很多教程，但是使用Docker搭建的方式是最简单的。鉴于官网的教程只有三行，这里简单写一点。

首先clone [leanote-docker](https://github.com/leanote/leanote-docker.git)这个git repo。
```bash
git clone https://github.com/leanote/leanote-docker.git
git submodule update --init
```

修改`app.conf`与下面相关的部分
```conf
http.addr=0.0.0.0 # listen on all ip addresses #这里不要修改
http.port=9000 #修改监听端口

site.url=https://your.own.host # 改为反向代理后的地址

# You Must Change It !! About Security!!
app.secret=mustchangeit! # 这里必须改掉，但是docker的教程没有说，官网的教程倒是有
```

修改`docker-compose.yaml`与下面相关的部分，避免database被容器外直接访问（其实不改也可以）。
```yml
services:
  mongo:
    # 增加这一项
    networks:
      - database_only

  leanote:
    # 增加这一项
    networks:
      - public_access
      - database_only

# 增加这一项
networks:
  public_access: # Provide the access for leanote
  database_only: # Provide the communication between leanote and database only
    internal: true
```

注：如果是**群辉**上面使用docker，则需将下列目录映射的本地目录手动新建：
```bash
/data/db
/leanote/src/github.com/leanote/leanote/files
/leanote/src/github.com/leanote/leanote/public/upload
```
并增加Owner用户组并允许读取/写入，否则mongodb会不断重启。
![synology-setting](/assets/images/2021-08-12-leanote-replace-evernote/synology-setting.png)

如果使用默认配置，则直接将`.`（即`leanote-docker`）目录设置增加Owner用户组即可。

下面可以启动docker了。
```bash
docker-compose up -d
```

稍等一会，即可以打开`http://localhost:9001`（或者docker机器的内网IP地址）。

![leanote2](/assets/images/2021-08-12-leanote-replace-evernote/leanote2.png)


## 2. 设置admin账户
默认的admin账户如下
```
用户名: admin
密码:   abc123
```
登录之后修改下密码：
![leanote_setting2](/assets/images/2021-08-12-leanote-replace-evernote/leanote_setting2.png)
![leanote_setting1](/assets/images/2021-08-12-leanote-replace-evernote/leanote_setting1.png)

建议关闭注册功能：
![leanote_setting5](/assets/images/2021-08-12-leanote-replace-evernote/leanote_setting5.png)

设置一下上传文件大小限制：
![leanote_setting4](/assets/images/2021-08-12-leanote-replace-evernote/leanote_setting4.png)


## 3. 反向代理&内网穿透
通常我们部署docker，都需要**反向代理**实现通过域名的外网HTTPS访问。

如果是部署在内网，则需要使用**内网穿透**。

>思考：为什么有外网服务器还要部署在内网？

这里简单讲一个实现内网穿透的思路，之前的文章也有提到过。详细的配置可能要单独整理一篇文章。

### 3.1 FRP内网穿透
内网机器上安装frpc（可以通过docker的方式安装）。

修改`frpc.ini`
```ini
[common]
server_addr = ur.own.ip.adr
server_port = 12345
token = setyourowntoken
tcp_mux = true

[Leanote]
type = tcp
local_ip = 127.0.0.1
local_port = 9001
remote_port = 9001
```

外网服务器上安装frps。

修改`frps.ini`
```ini
[common]
bind_port = 12345
log_file = ./frps.log
# debug, info, warn, error
log_level = info
log_max_days = 3
# auth token
token = setyourowntoken
max_pool_count = 50
tcp_mux = true
subdomain_host = your.own.domain
```

### 3.2 Caddy反向代理（推荐）
[Caddy](https://caddyserver.com/)是自动申请证书并将HTTP转化成HTTPS的服务器，配置超简单。

修改`Caddyfile`文件
```bash
your.own.domain {
    reverse_proxy http://127.0.0.1:9001/
}
```

### 3.3 Nginx反向代理（较麻烦）
[Nginx](https://www.nginx.com/)是生产环境使用的老牌代理服务，配置相对复杂，需要自己申请HTTPS证书。

在 `/etc/nginx/conf.d` 目录中创建 `leanote.conf`
```bash
server {
    listen     443;
    server_name  your.own.domain;
    ssl on;
    ssl_certificate /etc/nginx/conf.d/your.own.domain_chain.crt;
    ssl_certificate_key /etc/nginx/conf.d/your.own.domain_key.key;
    ssl_session_timeout 5m;
    ssl_protocols TLSv1.2;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    client_max_body_size 1000M;

    location / {
        proxy_pass http://127.0.0.1:9001;
    }
}
```

### 3.4 SSL证书申请
证书申请可以使用[这个网站](https://freessl.cn/)，每次可以申请一年的免费证书。
https://freessl.cn/
![freessl](/assets/images/2021-08-12-leanote-replace-evernote/freessl.png)


## 4. 客户端下载
官网下载地址： [http://app.leanote.com/](http://app.leanote.com/)

Github下载地址： [https://github.com/leanote/desktop-app/releases/](https://github.com/leanote/desktop-app/releases/)

iOS移动端： [https://itunes.apple.com/app/leanote/id1022302858]([https://itunes.apple.com/app/leanote/id1022302858)

Android移动端： [https://ali-cdn.leanote.top/apk/Leanote-v1.0-beta.7.apk](https://ali-cdn.leanote.top/apk/Leanote-v1.0-beta.7.apk)

### 4.1 Windows
![client2](/assets/images/2021-08-12-leanote-replace-evernote/client2.png)

登录自建服务的时候要填写`https://`开头。

![client1](/assets/images/2021-08-12-leanote-replace-evernote/client1.png)

桌面客户端不能用**代理**，看github一堆issue也没人管。如果是公司内网（需要代理），就需要[Proxifier](https://www.proxifier.com/)软件辅助桌面客户端走代理。

官网下载地址：[https://www.proxifier.com/](https://www.proxifier.com/)

试用版可以试用31天，之后需要花钱购买license。

一图流配置说明：
![proxy](/assets/images/2021-08-12-leanote-replace-evernote/proxy.png)

之后重启Leanote客户端即可正常联网。

### 4.2 Linux
这里使用了Ubuntu18.04desktop，在Ubuntu下界面和Windows差不多。
![ubuntu1](/assets/images/2021-08-12-leanote-replace-evernote/ubuntu1.png)

在启动器里面增加图标
```bash
cd ~/.local/share/applications
vi leanote.desktop
```

```ini
[Desktop Entry]
Encoding=UTF-8
Version=1.5
Comment=cloud based note-taking application
Type=Application
Terminal=false
Name=Leanote
Exec=/home/xxx/leanote/Leanote
Icon=/home/xxx/leanote/leanote.png
StartupNotify=true
Categories=TextEditor;
SingleMainWindow=true
X-GNOME-UsesNotifications=true
X-GNOME-SingleWindow=true
StartupWMClass=leanote-desktop
```

效果如下(2022/4/1更新)：
![ubuntu2](/assets/images/2022-04-01-Leanote-Linux-app-startup/2.png)

> **未解决问题**：侧边栏图标仍无法显示  
> 2022/4/1 更新：问题解决，参考上方ini文件  

### 4.3 iOS
iPhone界面：
![ios1](/assets/images/2021-08-12-leanote-replace-evernote/ios1.png)

iPad界面：
![ios3](/assets/images/2021-08-12-leanote-replace-evernote/ios3.png)
无论是iPhone还是iPad，界面都不是很好用，只能说凑合看吧。

## 5. 印象笔记导入
Windows客户端支持导入印象笔记格式（enex)以及html格式：

 - html导入的缺点是不能同步时间，部分图片会丢失。
 - enex导入的缺点是不能全部导入（会丢文章），部分图片会导入两次。

没有能完美解决的方法，只能手动修改一些重要的文章（累死我了）。

图为html格式导入流程：
![porting](/assets/images/2021-08-12-leanote-replace-evernote/porting.png)



## 6. 数据备份
`mongodump`只能保存数据库的快照，并不能保存上传的文件与图片（在`file`目录下）。

数据库升降级还是要用到，不过一般一个服务开起来基本就不会考虑数据库升降级了。

这里推荐使用计划任务`crontab -e`备份整个目录（示例）：

```bash
# 每日0点备份
0 0 * * * tar -czf /home/share/leanote_bk/leanote_$(date +%Y%m%d).tar.gz /home/share/leanote
# 每日1点检查并删除7日前备份文件
0 1 * * * find /home/share/leanote_bk/ -type f -mtime +7 -exec rm -f {} \;
```


如果是群晖，则使用`控制面板`--`计划任务`
![crontab2](/assets/images/2021-08-12-leanote-replace-evernote/crontab2.png)
![crontab3](/assets/images/2021-08-12-leanote-replace-evernote/crontab3.png)

脚本示例
```bash
tar -czf /volume1/backup/leanote_bk/leanote_$(date +%Y%m%d).tar.gz /volume1/backup/leanote
find /volume1/backup/leanote_bk/ -type f -mtime +7 -exec rm -f {} \;
```


## 7. 总结

虽然Leanote在细节上有很多不足，社区也基本不再维护，但是对于目前的使用场景足够用。

这篇文章虽然不是很完善，但对于想要自建Leanote的朋友来说还是有一些帮助，不足之处也欢迎指正。

> 一些尚未解决的问题：
> - 群晖docker使用wkhtmltopdf将PDF导出的问题。
> - Linux客户端侧边栏图标的问题。