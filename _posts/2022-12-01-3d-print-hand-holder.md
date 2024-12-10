---
layout: post
title: 3D打印个Ergodox键盘手托
date: 2022-12-01 20:30:00.000000000 +08:00
description: 本文介绍如何使用SketchUp画简单的3D图形，并展示3D打印的Erdodox手托。
author: aaron-li
categories: [工具指南]
tags: [sketchup, 3d, ergodox]  
---

多年前定制了一个基于[Ergodox](https://ergodox-ez.com/)的开源键盘，一直作为主力键盘来使用。  

由于是定制的，一直没有一个好的手托，之前几经周折买了一个木制手托。

![ori.jpg](/assets/img/posts/2022-12-01-3d-print-hand-holder/ori.jpg)

虽然舒服，但是用几小时下来，满手都是汗，搞得湿疹都犯了，严重影响工作进度。

![ori+sweat.jpg](/assets/img/posts/2022-12-01-3d-print-hand-holder/ori+sweat.jpg)

实在受不了，想起3D打印，自己画一个带孔洞手托打出来好了。

先看成品，这个透汗的同时不那么硌手，自觉不错。

![finish1.jpg](/assets/img/posts/2022-12-01-3d-print-hand-holder/finish1.jpg)

![finish2.jpg](/assets/img/posts/2022-12-01-3d-print-hand-holder/finish2.jpg)

接下来我们介绍下这其中涉及的流程。

# 1. 画个3D模型

首先，我们画个3D模型。

## SketchUp 的好处

我们先看下目前最好的3D制作软件[Rhinoceros](https://www.rhino3d.com/)。

![Rhinoceros.png](/assets/img/posts/2022-12-01-3d-print-hand-holder/Rhinoceros.png)

直接劝退了，咱就想画个手托，不想学三个月的3D绘图。

再看[SketchUp](https://app.sketchup.com/app)，清晰易懂，所见即所得，适合画我们这种简单的小玩意。

![Sketchup1.png](/assets/img/posts/2022-12-01-3d-print-hand-holder/Sketchup1.png)


## 一些基本操作

我们先画个长方形。

![Sketchup2.gif](/assets/img/posts/2022-12-01-3d-print-hand-holder/Sketchup2.gif)

把它变成长方体。

![Sketchup3.gif](/assets/img/posts/2022-12-01-3d-print-hand-holder/Sketchup3.gif)

再在上面打个洞。

![Sketchup4.gif](/assets/img/posts/2022-12-01-3d-print-hand-holder/Sketchup4.gif)

基本上会这些就可以画了。


## 之前画的一些图

之前做室内装修的时候就用SketchUp来画3D示意图，那时候绘画技术还很粗糙。

注意3D打印一定要闭合的图形，这里有些图其实不是闭合的（有些面是空面，或者干脆就是个面不是体）。

![3ds.jpg](/assets/img/posts/2022-12-01-3d-print-hand-holder/3ds.jpg)

## 手托的模型

画好的手托模型就这个样子。

![handholder.gif](/assets/img/posts/2022-12-01-3d-print-hand-holder/handholder.gif)

这里还要注意3D打印尺寸一定要对，最开始我画了个以米为单位的（应该是毫米），把卖家都搞懵了还以为我要打体育馆。

## 导出STL

SketchUp支持导出STL，一般卖家都要这个格式的3D文件。

![stl.png](/assets/img/posts/2022-12-01-3d-print-hand-holder/stl.png)

3D模型也传了一份，链接在resource里。

# 2. 3D打印手托

## 到手的模型

随便找了个好评多的卖家，用最便宜的材料，打印了一个实心的，到货效果如下。

![printed1.jpg](/assets/img/posts/2022-12-01-3d-print-hand-holder/printed1.jpg)

![printed2.jpg](/assets/img/posts/2022-12-01-3d-print-hand-holder/printed2.jpg)

直接拿来使用还是有一些硌手的，我们还需要再给它喷个油漆。

## 上色

某宝买了个快干的油漆（全干要72小时，然后才能承重），厚厚的喷几层。

![spray.jpg](/assets/img/posts/2022-12-01-3d-print-hand-holder/spray.jpg)

喷好之后，六边形孔的位置就圆润了。

# 3. 完成效果

## 对比3D模型

基本一模一样。

![cmp+model.jpg](/assets/img/posts/2022-12-01-3d-print-hand-holder/cmp+model.jpg)

![cmp+model2.jpg](/assets/img/posts/2022-12-01-3d-print-hand-holder/cmp+model2.jpg)


## 对比原来的手托

这个用起来就没那么捂汗了，可以继续开心地工作了。

![compare1.jpg](/assets/img/posts/2022-12-01-3d-print-hand-holder/compare1.jpg)

![compare2.jpg](/assets/img/posts/2022-12-01-3d-print-hand-holder/compare2.jpg)

# resources

[Sketchup Web APP](https://app.sketchup.com/app)  
[Ergodox](https://ergodox-ez.com/)  
[Config Your Own Ergodox](https://configure.zsa.io/)  
[镂空手托2L](/assets/img/posts/2022-12-01-3d-print-hand-holder/镂空手托2L.stl) 
[镂空手托2R](/assets/img/posts/2022-12-01-3d-print-hand-holder/镂空手托2R.stl)
