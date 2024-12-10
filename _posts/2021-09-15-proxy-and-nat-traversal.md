---
layout: post
title: 代理与内网穿透简介
date: 2021-09-15 19:00:00.000000000 +08:00
description: 本文介绍正向代理、反向代理、内网穿透的概念以及一些基本的配置方法。
author: aaron-li
categories: [tools]
tags: fanxiangdaili, neiwangchuantou] 
---

## 1. 什么是代理
一般来说，终端（client）通过A去连接服务器（Server），A就是代理（proxy）。  
A可以是网关、路由器、或者客户端/服务器自身部署的网络代理服务。  

文中IP都是瞎编的。  

## 2. 什么是正向代理
客户端使用**代理协议**通过**已知**的代理服务去访问互联网/特定网络的过程。  
正向代理是**客户端**的代理，通常和客户端同属一个局域网/内网。  
![proxy](/assets/img/posts/2021-09-15-proxy-and-nat-traversal/proxy.png)

### 2.1 正向代理：一般场景示例
下图演示一个局域网内使用代理来管理外网通信，一般应用在公司网络：  
![forward_proxy](/assets/img/posts/2021-09-15-proxy-and-nat-traversal/forward_proxy.gif)
防火墙阻止除了代理服务器外的所有内网机器直接访问外网。  
内网机器通过代理服务器访问外网。  
代理服务器和防火墙可以都部署在路由器上，也可以分别部署在不同机器上。  

客户端需要知道代理服务器的地址及其所使用的协议，如http,socks等。  

### 2.2 DHCP 自动配置正向代理（PAC）
以openwrt为例：  

在 `/etc/config/dhcp` 下添加 `dhcp_option 252` 这一行。  
```
config dhcp 'lan'
        option interface 'lan'
        ...
        list dhcp_option '252,http://192.168.1.1/proxy.pac'
        ...
```

`proxy.pac`文件内容示例：  
```c
function FindProxyForURL(url, host) {
    // if(isPrivateIp(host)) {
    //     return "DIRECT";
    // }
    return "PROXY 192.168.1.1:8080; DIRECT";
}
```
### 2.3 TinyProxy 正向代理配置
以linux的[TinyProxy](http://tinyproxy.github.io/)举例:  
配置文件： `/etc/tinyproxy/tinyproxy.conf`  
```bash
User tinyproxy
Group tinyproxy
Port 11111
Allow 127.0.0.1
Allow 192.168.1.0/24
```
详细配置参考 [http://tinyproxy.github.io/](http://tinyproxy.github.io/)。  

### 2.4 正向代理：透明代理示例
下图演示一个局域网内使用透明代理来管理外网通信：  
![transparent](/assets/img/posts/2021-09-15-proxy-and-nat-traversal/transparent.gif)
透明代理的优势在于，客户端无需知道代理服务器的地址与协议。  


### 2.5 iptables 透明代理配置
以linux下的配置为例：  
```bash
# reference https://liqiang.io/post/dive-in-iptables
# -t nat              -> add to nat table
# -A PREROUTING       -> PREROUTING table, handle the package before arrived to firewall
# -p tcp              -> matches tcp protocol
# -s 192.168.1.0/24   -> matches 192.168.1.0/24 net segment is source 
# ! -d 192.168.1.0/24 -> matches 192.168.1.0/24 net segment is not destination
# -j REDIRECT          -> action is redirect

# Redirect TCP
iptables -t nat -A PREROUTING -p tcp -s 192.168.8.0/24 ! -d 192.168.8.0/24 -j REDIRECT --to-port 11111
```

> Openwrt 可参考[官方教程](https://openwrt.org/docs/guide-user/services/proxy/tinyproxy)实现基于TinyProxy的透明代理：  
> [https://openwrt.org/docs/guide-user/services/proxy/tinyproxy](https://openwrt.org/docs/guide-user/services/proxy/tinyproxy)

## 3. 什么是反向代理
客户端通过网络访问反向代理服务器，通过反向代理服务器获取服务器资源，在这个过程中客户端**不知道**反向代理的存在（客户端将反向代理视为服务器）。  
反向代理是**服务器**的代理，通常和服务器同属一个局域网/内网。  
![reversed_proxy](/assets/img/posts/2021-09-15-proxy-and-nat-traversal/reversed_proxy.png)

### 3.1 反向代理：一般场景示例
下图演示一个局域网内使用反向代理来管理多个web服务器资源，一般应用在公网IP资源有限的情况：  
![reversed_proxy](/assets/img/posts/2021-09-15-proxy-and-nat-traversal/reversed_proxy.gif)
防火墙阻止除了代理服务器外的所有外网机器直接访问内网。  
反向代理服务器通过反向代理协议将不同服务器上的Web服务映射到本机不同的网络地址。  
代理服务器具有公网IP地址/域名。  

> **通常**个人不会有这么多机器，一般web服务是vps上的一些docker，再通过vps上的nginx/caddy做反向代理到公网。  

### 3.2 nginx 反向代理配置
以linux上的[nginx](https://www.nginx.com/)为例，在 `/etc/nginx/sites-available` 或 `/etc/nginx/conf.d` 目录内添加 `some.domain.com.conf` 文件。  
参考文件内容如下：  
```bash
server {
    listen     443;
    server_name  some.domain.com;
    ssl on;
    ssl_certificate /etc/nginx/conf.d/some.domain.com_chain.crt;
    ssl_certificate_key /etc/nginx/conf.d/some.domain.com_key.key;
    ssl_session_timeout 5m;
    ssl_protocols TLSv1.2;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    location / {
        proxy_pass http://192.168.1.100:8080/;
    }
    auth_basic 'Restricted';
    auth_basic_user_file /etc/nginx/conf.d/some.domain.com.htpasswd;
}
```

以上配置将内网的 HTTP 服务 `http://192.168.1.100:8080/` 转成 HTTPS，并映射到域名 `some.domain.com`。  

HTTPS服务需要的证书 `some.domain.com_chain.crt` 与 `some.domain.com_key.key`  
可以在 [let's encrypt](https://letsencrypt.org/getting-started/) 或 [freessl.cn](https://freessl.cn/) 申请。  

简单验证所用的密码文件 `some.domain.com.htpasswd` 可以通过 `htpasswd` 工具生成，指令如下：  

```bash
sudo apt install apache2-utils
htpasswd -bc /etc/nginx/conf.d/some.domain.com.htpasswd username password
```

### 3.3 Caddy 反向代理配置
[Caddy](https://caddyserver.com/)简单得多，证书自动申请，适合个人网站。  
下面以caddy 2配置为例：  
```bash
some.domain.com {
    reverse_proxy http://192.168.1.100:8080/
    basicauth /* {
	    username JDJhJDEwJEVCNmdaNEg2Ti5iejRMYkF3MFZhZ3VtV3E1SzBWZEZ5Q3VWc0tzOEJwZE9TaFlZdEVkZDhX
    }
}
```
简单验证所用的密码HASH通过`caddy hash-password`指令生成
```bash
caddy hash-password
	[--plaintext <password>]
	[--algorithm <name>]
	[--salt <string>]
```
详细参考：[caddy-hash-password](https://caddyserver.com/docs/command-line#caddy-hash-password)

### 3.4 反向代理：CDN示例
下图演示一个多个客户端通过CDN访问网络资源：  
![cdn](/assets/img/posts/2021-09-15-proxy-and-nat-traversal/cdn.gif)
三个**IP不同**的反向代理服务器（CDN）代理同一个网络资源。  
不同的客户端在请求DNS时，DNS分配给客户端**速度最快**的IP地址，从而使客户端访问最快的CDN，一般情况下可以加速客户端的网络访问。  

### 3.5 反向代理：负载均衡示例
下图演示一个服务器集群的负载均衡：  
![balancer](/assets/img/posts/2021-09-15-proxy-and-nat-traversal/balancer.gif)
服务器集群提供的web服务一般是**一样**的，负载均衡可以根据当前服务器的空闲程度合理分配网络连接，从而有效提升数据处理能力。  
在负载均衡服务器挂掉的时候，备份服务器可以顶上。  
负载均衡的实现也是一种反向代理。  

## 4. 什么是内网穿透
内网穿透可以理解为一个**专用信道**，这个信道是由**内网机器**(Client)发起向**外网服务器**(Server)的连接，目的是使处于**外网的客户端**可以通过外网服务器访问到内网的服务。  

与反向代理不同的是，内网穿透所建立的专用信道需要一直保持，且发起连接的方向是相反的。  

![nat_traversal](/assets/img/posts/2021-09-15-proxy-and-nat-traversal/nat_traversal.png)

### 4.1 内网穿透+反向代理示例
内网穿透可以单独使用，也可以配合反向代理使用（通常是用多个子域名代理不同的端口，以及加HTTPS支持）。  
下图演示一个内网穿透与反向代理结合的例子：  
![nat_traversal](/assets/img/posts/2021-09-15-proxy-and-nat-traversal/nat_traversal.gif)
内网机器通过 [FRP](https://gofrp.org/docs/) 与外网机器建立连接，并将其 8080 端口映射到外网机的 8080 端口，外网机的反向代理将 `http://101.120.252.139:8080` 代理到 `https://some.domain.com` 。  
这里可以在外网机防火墙配置拒绝 8080 端口的访问，以阻止通过 HTTP 的访问。  

### 4.2 FRP 内网穿透配置
内网机器上安装frpc，并修改 `frpc.ini`：  

```ini
[common]
server_addr = 101.120.252.139
server_port = 12345
token = setyourowntoken
tcp_mux = true

[web]
type = tcp
local_ip = 192.168.1.10
local_port = 8080
remote_port = 8080
```

外网服务器上安装frps。 并修改 `frps.ini`：  

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
subdomain_host = some.domain.com
```

详细配置参考： [https://gofrp.org/docs/](https://gofrp.org/docs/)

### 4.3 花生壳内网穿透
如果没有自己的外网机器（比如vps），选择内网穿透服务商提供的服务也可以。  

比如花生壳，配置比较无脑，国内的访问速度也不错。  

不过是收费服务，还有带宽和流量限制。  

详细可以参考官网： [https://hsk.oray.com/](https://hsk.oray.com/)  

### 4.4 Cloudflare argo tunnel
如果不太在乎国内的访问速度，或者是国外用户，可以用免费的 [Cloudflare argo tunnel](https://www.cloudflare.com/zh-cn/products/tunnel/)。  

配置方法可以参考： [https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup)  


## 5. 总结
这其实没啥好总结的，这些个动图弄得挺费劲的。  
有哪个地方写的不对欢迎指正。  
