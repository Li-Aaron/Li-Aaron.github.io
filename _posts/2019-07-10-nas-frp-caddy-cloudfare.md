---
layout: post
title: 蜗牛星际+Frp+Caddy+Cloudflare折腾记
date: 2019-07-10 18:00:00.000000000 +08:00
description: 黑裙如何实现外网访问？本文详细介绍使用Frp内网穿透，Caddy反向代理，Cloudflare CDN加速的方法。
author: aaron-li
categories: [tools]
tags: [synology, frp, caddy, cloudflare, nas, fanxiangdaili]  
---

一段时间没有折腾什么东西，感觉人都颓废了。

前一阵蜗牛星际炸雷了之后，网上很多小机箱搞的很火爆。鉴于一直想搞一个NAS但又苦于昂贵的价格，赶上这班车直接就剁了一个A款。

![](/assets/img/posts/2019-07-10-nas-frp-caddy-cloudfare/A.JPG)

选择A款的原因，其实是因为便宜，在一个有个外面板不会直接漏硬盘在外面，网上有很多对比评测。

黑裙的系统是918（其实马爸爸家的卖家都会给装好），google上也有很多就不复读了。

下面正文开始。

## 1. 利用Frp内网穿透

迫于没有公网IP，以及黑裙，只能走反向代理的方式从外网访问，这时候[Frp](https://github.com/fatedier/frp/blob/master/README_zh.md)就很重要了。

要使用Frpc，首先需要安装docker，在套件中心的第三方中找到Docker，安装。

![](/assets/img/posts/2019-07-10-nas-frp-caddy-cloudfare/docker.JPG)

在Docker中的注册表搜索frpc，安装[oldiy/frpc](https://hub.docker.com/r/oldiy/frpc)这一映像，详见[使用说明](https://hub.docker.com/r/oldiy/frpc)。

服务器端安装详见[Frp 的 GitHub page](https://github.com/fatedier/frp/blob/master/README_zh.md)。

安装好之后frpc.ini以及vps上的frps.ini参考如下：

### 1 Port to Subdomain
对于有域名的同学，可以直接使用Frp将内网的端口映射到外网的subdomain，可以直接实现访问，由于黑裙的原因，证书申请有些问题，https访问会有insecure的提示，可以将local_port设置为5000走http访问（5000为默认Http端口，5001为默认Https端口，可以更改）。

NAS端配置文件：

```ini
[common]
server_addr = [your ip]
server_port = [your port]
token = [your token]
tcp_mux = true

[nas]
type = tcp
local_ip = 127.0.0.1
local_port = 5001
subdomain = nas
```

Server配置文件，默认路径/etc/frps/frps.ini，取决于frps解压路径：

```ini
[common]
bind_port = [your port]
log_file = ./frps.log
log_level = info
log_max_days = 3
token =  [your token]
max_pool_count = 50
tcp_mux = true
subdomain_host = your.domain
```

访问方式 https://nas.your.domain

### 2 Port to Port
对于没有域名（或想使用caddy申请证书）的同学，可以采用映射port的方法，配置文件如下。

NAS端配置文件：

```ini
[common]
server_addr = [your ip]
server_port = [your port]
token = [your token]
tcp_mux = true

[nas]
type = tcp
local_ip = 127.0.0.1
local_port = 5001
remote_port = 5001
```

Server配置文件，最后一行可有可无：

```ini
[common]
bind_port = [your port]
log_file = ./frps.log
log_level = info
log_max_days = 3
token =  [your token]
max_pool_count = 50
tcp_mux = true
#subdomain_host = your.domain
```

访问方式 https://[your ip]:5001

## 2. 利用Caddy反向代理
对于有域名又不想麻烦申请证书的同学，可以用[Caddy](https://caddyserver.com/)做反向代理，使用上面Port to Port的方式。

Caddy的使用教程详见[Caddy 的 Github page](https://github.com/caddyserver/caddy#quick-start)。

简单的CaddyFile如下：

```
nas.your.domain {
    tls your@email.com

    proxy / https://127.0.0.1:5001 {
        insecure_skip_verify
    }
}
```
访问方式 https://nas.your.domain

## 3. 利用Cloudfare CDN加速

对于访问速度慢的VPS，或不想暴露真实IP的同学，可以使用CDN加速，这里推荐Cloudfare的免费CDN。

Caddy的安装文件需要在网站上定制（增加cloudflare的支持），在[Download](https://caddyserver.com/download)界面Plugin中添加dns.cloudflare

![](/assets/img/posts/2019-07-10-nas-frp-caddy-cloudfare/Caddy.JPG)

Caddy的配置文件也需要相应修改：

```ini
nas.your.domain {
    tls {
        dns cloudflare
    }

    proxy / https://127.0.0.1:5001 {
        insecure_skip_verify
    }
}
```

CloudFlare设置好DNS解析，并将Cloudflare nameservers加入到VPS的DNS nameserver中，详见[Changing your domain nameservers to Cloudflare](https://support.cloudflare.com/hc/en-us/articles/205195708-Step-3-Change-your-domain-name-servers-to-Cloudflare)


访问方式 https://nas.your.domain

## Reference
[Frp 的 Github page](https://github.com/fatedier/frp/blob/master/README_zh.md)  
[oldiy/frpc](https://hub.docker.com/r/oldiy/frpc)  
[Caddy 的 Github page](https://github.com/caddyserver/caddy#quick-start)  
[Changing your domain nameservers to Cloudflare](https://support.cloudflare.com/hc/en-us/articles/205195708-Step-3-Change-your-domain-name-servers-to-Cloudflare)  
