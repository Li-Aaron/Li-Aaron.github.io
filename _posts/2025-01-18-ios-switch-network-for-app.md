---
layout: post
title: IOS 为APP单独设置走不同的流量卡
date: 2025-1-18 13:00:00.000000000 +08:00
description: 本文介绍在IOS中，如何在启动APP的时候自动切换数据流量走哪个号码（针对不同的APP使用不同的流量卡）。
author: aaron-li
categories: [玩机攻略, IOS]
tags: [ios, 流量, 自动化]
---

由于中午需要用流量看催困视频，但是平时流量主卡并不是B站的免流卡。  
而手动切换后经常忘了换回来导致免流卡流量爆炸，通过一番折腾，找到了个针对B站自动切换流量的方法。

本方法针对IOS，任意APP均可。

打开快捷方式：

![0](/assets/img/posts/2025-01-18-ios-switch-network-for-app/0.png)

切到自动化界面，新建一个自动化。

![0.25](/assets/img/posts/2025-01-18-ios-switch-network-for-app/0.25.png)

选择APP：

![0.5](/assets/img/posts/2025-01-18-ios-switch-network-for-app/0.5.png)

APP选择哔哩哔哩。  
勾选已打开，立即运行。

![0.75](/assets/img/posts/2025-01-18-ios-switch-network-for-app/0.75.png)

选择新建空白自动化。

![0.8](/assets/img/posts/2025-01-18-ios-switch-network-for-app/0.8.png)

搜索默认号码

![0.9](/assets/img/posts/2025-01-18-ios-switch-network-for-app/0.9.png)

设置为数据，以及所需的流量卡。

![2](/assets/img/posts/2025-01-18-ios-switch-network-for-app/2.png)

完成。

![1](/assets/img/posts/2025-01-18-ios-switch-network-for-app/1.png)

建立相似的自动化，关闭时切回原来的流量卡。

![3](/assets/img/posts/2025-01-18-ios-switch-network-for-app/3.png)

看看效果。

![effect](/assets/img/posts/2025-01-18-ios-switch-network-for-app/effect.gif)

效果拔群。

不过刚切过去的时候，会卡一会连不上网。