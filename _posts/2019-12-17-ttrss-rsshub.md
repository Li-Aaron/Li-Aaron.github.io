---
layout: post
title: TTRSS + RSSHUB 个人RSS服务
date: 2019-12-17 18:00:00.000000000 +08:00
description: 如何在众多越来越难用的新闻客户端中保持自己对新闻纯净的要求，本文介绍如何利用TTRSS和RSSHUB打造自己专属的新闻客户端。
author: aaron-li
categories: [工具指南]
tags: [ttrss, rsshub, rss]
---

最近的聚合类新闻客户端越来越难用了。
又不喜欢一堆手机客户端，搞一个聚合的RSS势在必行。  
不过RSS已经过气多年，支持的软件/网站少之又少，所以决定自己搞一个。  
看了一些安利的文章，决定使用docker来部署[Tiny Tiny RSS](https://tt-rss.org/)作为RSS服务器，[RSSHub](https://docs.rsshub.app/)作为RSSfeed的生成器。  

## 1. 安装docker/docker-compose

[docker](https://docs.docker.com/)就是造福人类的神器啊，不需要从头开始一点点配置，直接一步到位。  
ubuntu使用如下指令安装docker:  

```bash
sudo apt install docker.io
```

如果是用群晖，docker只需要在应用中心中启用就可以了，但是群晖没办法(或者是我没找到)docker-compose，这里还是以ubuntu为主。  

[docker-compose](https://docs.docker.com/compose/gettingstarted/)最好不要使用apt来安装，在ubuntu16.04上安装的是1.8版，而TTRSS和RSSHub的docker-compose.yml都需要更高的版本支持。这里直接从官网的[安装教程](https://docs.docker.com/compose/install/)中截取一段

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.25.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

github比较慢的同学可以用代理下载
```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.25.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose -x http://your.proxy:port/
```

安装好测试一下
```bash
docker-compose --version
```

注：若官网更新请以官网为准

## 2. 部署TTRSS

TTRSS全名Tiny Tiny RSS，是一款基于 PHP 语言的开源 RSS 阅读器，提供了网页阅读和通过插件对很多RSS客户端的支持。

[Henry Wang](https://henry.wang/) 提供了一站式部署的[Awesome TTRSS](https://ttrss.henry.wang/zh/#%E5%85%B3%E4%BA%8E)容器，经过简单修改既可以直接使用。

[通过 docker-compose 部署](https://ttrss.henry.wang/zh/#%E9%80%9A%E8%BF%87-docker-compose-%E9%83%A8%E7%BD%B2) <=官方教程

直接下载 [docker-compose.yml](https://raw.githubusercontent.com/HenryQW/Awesome-TTRSS/master/docker-compose.yml) 或
```bash
wget https://raw.githubusercontent.com/HenryQW/Awesome-TTRSS/master/docker-compose.yml
```

按照其注释修改其中的密码和`SELF_URL_PATH=http://your.domain/`  
若需要代理则增加环境变量`HTTP_PROXY=http://your.proxy:port/`  

注意RaspberryPi不支持OpenCC

```bash
docker-compose up -d
```
等待部署完成即可，若需修改变量则需
```bash
docker-compose down
docker-compose up -d
```
注意
```bash
docker-compose restart
```
并不会对修改做出响应，详见[https://docs.docker.com/compose/reference/restart/](https://docs.docker.com/compose/reference/restart/)

通过`http://your.domain/`即可访问TTRSS，注意更改用户名密码。

其他插件的启用详见[https://ttrss.henry.wang/zh/#插件](https://ttrss.henry.wang/zh/#%E6%8F%92%E4%BB%B6)

## 3. 部署RSSHUB

RSSHub 是一个开源、简单易用、易于扩展的 RSS 生成器，发起人为[DIYgod](https://github.com/DIYgod)，目前在Github有300+Contributors。(万物皆可 RSS)

[RSSHub 部署](https://docs.rsshub.app/install/#docker-compose-bu-shu) <=官方教程

和TTRSS类似直接下载 [docker-compose.yml](https://raw.githubusercontent.com/DIYgod/RSSHub/master/docker-compose.yml) 或
``` bash
wget https://raw.githubusercontent.com/DIYgod/RSSHub/master/docker-compose.yml
docker volume create redis-data
docker-compose up -d
```
这里不需要修改什么，如果需要代理则参见[代理配置](https://docs.rsshub.app/install/#dai-li-pei-zhi)

## 4. Caddy反向代理

具体的Caddy反向代理和Cloudflare CDN加速可以参考之前写的[蜗牛星际+Frp+Caddy+Cloudflare折腾记]({{site.url}}/2019/07/nas-frp-caddy-cloudfare/)

这里给出具体的Caddyfile
```ini
example.com {
    tls {
        dns cloudflare
    }
    gzip
    timeouts none
    root /var/www/html
    basicauth / accountname password
    proxy /RSSHub 127.0.0.1:1200 {
        without /RSSHub
    }
    proxy /TTRSS 127.0.0.1:181 {
        transparent
    }
}
```

或想和主站分开，做独立的subdomain
```ini
example.com {
    tls {
        dns cloudflare
    }
    gzip
    timeouts none
    root /var/www/html
}

rsshub.example.com {
    tls {
        dns cloudflare
    }
    gzip
    basicauth / accountname password
    proxy / 127.0.0.1:1200
}

ttrss.example.com {
    tls {
        dns cloudflare
    }
    gzip

    proxy / 127.0.0.1:181 {
        transparent
    }
}
```

## 5. 遇到的一些问题

在配置过程中遇到了奇奇怪怪的（弱智）问题

### 5.1 docker-compose 启动失败

在配置好RSSHub的时候发现TTRSS怎么都up不了，要么就是Killed，要么就是docker service stop了（error exit）

```
root@xxx:~/ttrss# docker-compose up -d
Creating network "ttrss_default" with the default driver
Creating mercury  ...
Creating postgres ...
Creating opencc   ...
Creating ttrss    ...
Killed
```

因为一直是SSH登陆没发现什么原因，打开console一看原来是内存不够了，这时候我们需要增加swap（毕竟内存要花钱买）。参考：[Vultr VPS内存不足怎么办？增加SWAP交换分区可以解决](http://vultr.idcspy.com/464.html)。

注意这里是root权限
```bash
cd /var
dd if=/dev/zero of=swapfile bs=1024 count=524288 #512M, 按需调整
mkswap /var/swapfile
chmod 0600 /var/swapfile
/sbin/swapon swapfile
```
在 `/etc/fstab`中增加如下一行
```
/var/swapfile swap swap defaults 0 0
```
重启即可

执行`free -m`
```
root@xxx:~/ttrss# free -m
              total        used        free      shared  buff/cache   available
Mem:            480         225          18          11         236         190
Swap:           511         244         267
```
swap已经建立好了


### 5.2 TTRSS SELF_URL_PATH incorrect

配置好TTRSS的时候，登陆发现提示错误
```
Please set SELF_URL_PATH to the correct value detected for your server: http://127.0.0.1:181/
```
我明明设置了正确的域名，却让我将SELF_URL_PATH设置成本地地址  
结果修改后登陆页面是有了，一登陆就会跳转到 http://127.0.0.1:181/

结果发现Caddyfile中忘记设置`transparent`
```ini
ttrss.example.com {
    tls {
        dns cloudflare
    }
    gzip

    proxy / 127.0.0.1:181 {
        transparent
    }
}
```


### 5.3 RSSHub wrong path

配置好RSSHub后，什么地址都会报错
```
Error: wrong path
    at module.exports (/app/lib/middleware/parameter.js:14:15)
    at process._tickCallback (internal/process/next_tick.js:68:7)
```
在github的issue里面找到的都是一些不按照教程使用错误，仔细检查发现并没有

结果也是Caddyfile的坑，忘记了加`without RSSHub`这一条，结果后端解析的地址都是错的  
比如  
`example.com/RSSHub/36kr/newsflashes`  
解析成了  
`127.0.0.1:1200/RSSHub/36kr/newsflashes`  

```ini
example.com {
    tls {
        dns cloudflare
    }
    gzip
    timeouts none
    root /var/www/html
    basicauth / accountname password
    proxy /RSSHub 127.0.0.1:1200 {
        without /RSSHub
    }
}
```

## Reference
[Awesome TTRSS](https://ttrss.henry.wang/zh/#%E5%85%B3%E4%BA%8E)  
[RSSHub](https://docs.rsshub.app/)  
[如何搭建属于自己的 RSS 服务，高效精准获取信息](https://sspai.com/post/41302)  
[（另一篇）Tiny Tiny RSS 教程](https://sspai.com/post/42787)  
[蜗牛星际+Frp+Caddy+Cloudflare折腾记]({{site.url}}/2019/07/nas-frp-caddy-cloudfare/)  
[Vultr VPS内存不足怎么办？增加SWAP交换分区可以解决](http://vultr.idcspy.com/464.html)
