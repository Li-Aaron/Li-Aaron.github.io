---
layout: post
title: 蜗牛星际黑裙升级，硬解H265
date: 2022-08-20 16:00:00.000000000 +08:00
description: 本文介绍黑裙升级CPU主板，系统信息更换正确CPU名称，以及开启video station中truehd/dts/ac3音频解码的方法。
author: aaron-li
categories: [tools]
tags: [synology, upgrade, video station]  
---


最近下载的好多美剧都是h265编码的，[J1900](https://ark.intel.com/content/www/us/en/ark/products/78867/intel-celeron-processor-j1900-2m-cache-up-to-2-42-ghz.html)的黑裙不支持硬解，卡的要死。

![j1900-codec](/assets/img/posts/2022-08-20-nas-upgrade/j1900-codec.png)

为了更舒服的在家里看视频，我决定升级一下这台黑裙。


# 1 前期调研
蜗牛星际A机箱只能塞下17x17的ITX板子，而我们还有4块3.5的SATA HDD+1块2.5的SATA SSD，这样可选择的余地就不多了。

综合需求就是必须满足：
1. ITX
2. 至少5 Sata
3. 支持H265的硬解

根据调研发现满足需求的方式有：
1. 映泰[J4125](https://ark.intel.com/content/www/us/en/ark/products/197305/intel-celeron-processor-j4125-4m-cache-up-to-2-70-ghz.html)集成主板(2 Sata)+PCI-E转4 Sata卡 (669+89 RMB)
2. [豆希工控H310主板](http://www.exey.cn/productinfo/17634.html)+[G5400散片](https://ark.intel.com/content/www/us/en/ark/products/129951/intel-pentium-gold-g5400-processor-4m-cache-3-70-ghz.html)+超薄散热器 (368+290+39 RMB)

我们简单做一下[对比](https://www.cpubenchmark.net/compare/Intel-Pentium-Gold-G5400-vs-Intel-Celeron-J4125-vs-Intel-Celeron-J1900/3248vs3667vs2131)：

|指标|G5400|J4125|J1900|
| - | - | - | - |
|Launch|2018 Q2|2019 Q4|2013 Q1|
|Generation|8|9|6|
|Architecture|Coffee Lake S|Gemini Lake|Bay Trail|
|Lithography|14 nm|14 nm|22 nm|
|Physical Cores|2|4|4|
|Threads|4|4|4|
|Clockspeed|3.7 GHz|2.0 GHz|2.0 GHz|
|Turbo Speed|Not Supported|Up to 2.7 GHz|Up to 2.4 GHz|
|Max TDP|54W|10W|10W|
|Memory|DDR4|DDR4/LPDDR4|DDR3L|
|GPU|Intel HD Graphics 610|Intel UHD Graphics 600|Intel HD Graphics for Atom|
|H265 8bit Support|Decode/Encode|Decode/Encode|No|
|H265 10bit Support|Decode/Encode|Decode/Encode|No|
|H264 Support|Decode/Encode|Decode/Encode|Decode/Encode|
|VP8 Support|Decode/Encode|Decode/Encode|No|
|VP9 Support|Decode/Encode|Decode/Encode|No|

J4125除了功耗以外，均弱于G5400，（毕竟TDP在这摆着）。

综合价格考虑，我选择第二套改造方案。

（使用下来，待机功耗大概增加了12W左右，但机器热量上去了，导致给设备柜增加了一个风扇）

# 2 升级

## 2.1 更换CPU 主板
由于买的都是二手板U，成色还算可以。

![mb+cpu.JPG](/assets/img/posts/2022-08-20-nas-upgrade/mb+cpu.JPG)

店家提供了跳线配置的说明（其实板子上印的蛮详细）：

![douxi1.JPG](/assets/img/posts/2022-08-20-nas-upgrade/douxi1.JPG)

![douxi2.JPG](/assets/img/posts/2022-08-20-nas-upgrade/douxi2.JPG)

![douxi3.JPG](/assets/img/posts/2022-08-20-nas-upgrade/douxi3.JPG)

加上超薄风扇，内存和SSD，上机测试（这个M2接口不支持m2 sata，只支持m2 nvme)。

![testrun.JPG](/assets/img/posts/2022-08-20-nas-upgrade/testrun.JPG)

查看BIOS。

![bios.JPG](/assets/img/posts/2022-08-20-nas-upgrade/bios.JPG)

将板子替换上去（这个风扇高度已经比较极限了），插好所有sata盘，直接正常开机无需额外操作（DSM 6.2.1版本适用，其他版本不一定）。

## 2.2 4K 视频测试

播放如下视频：

![finch.png](/assets/img/posts/2022-08-20-nas-upgrade/finch.png)

播放效果以及CPU负载：

![4k.png](/assets/img/posts/2022-08-20-nas-upgrade/4k.png)

4k h265 解码毫无压力。

## 2.3 正确显示CPU信息

黑裙的控制面板中，CPU永远显示J3455，这让人有点强迫症。

![cpuinfoincorrect.png](/assets/img/posts/2022-08-20-nas-upgrade/cpuinfoincorrect.png)

我们可以使用[ch_cpuinfo](https://github.com/FOXBI/ch_cpuinfo)，非常简单，链接：[https://github.com/FOXBI/ch_cpuinfo](https://github.com/FOXBI/ch_cpuinfo)。

![ch_cpuinfo.png](/assets/img/posts/2022-08-20-nas-upgrade/ch_cpuinfo.png)

注销再登陆，CPU被改成了正确的型号。

![cpuinfocorrect.png](/assets/img/posts/2022-08-20-nas-upgrade/cpuinfocorrect.png)

# 3 Video Station 开启 TrueHD / DTS / AC3 解码

虽然升级了系统支持了H265解码，但是由于video station的版权原因，其内置的ffmpeg不支持TrueHD/DTS/AC3解码，而我的美剧大部分是AC3，这就很让人蛋疼。

在网上找到了基于ffmpeg-wrapper的解决方法，归纳整理一下。

首先在群晖的community中下载ffmpeg：[https://synocommunity.com/package/ffmpeg](https://synocommunity.com/package/ffmpeg)

在套件中心中手动安装：

![install-ffmpeg.png](/assets/img/posts/2022-08-20-nas-upgrade/install-ffmpeg.png)

替换video station中的ffmpeg，并解除TrueHD/DTS/AC3的限制：

```bash
# 解除eac3, dts, truehd的限制并备份文件
sed -i'.orig' -e 's/eac3/3cae/' -e 's/dts/std/' -e 's/truehd/dheurt/' /var/packages/VideoStation/target/lib/libsynovte.so

# 备份video station的ffmpeg，同时是ffmpeg-wrapper的备选 (wrapper会在调用套件中ffmpeg失败后fall back到这一个，名字一定要用.orig)
cp /var/packages/VideoStation/target/bin/ffmpeg /var/packages/VideoStation/target/bin/ffmpeg.orig
chown root:VideoStation /var/packages/VideoStation/target/bin/ffmpeg.orig
chmod 750 /var/packages/VideoStation/target/bin/ffmpeg.orig
chmod u+s /var/packages/VideoStation/target/bin/ffmpeg.orig

# 为了方便普通用户执行一些特权命令，SUID/SGID程序允许普通用户以root身份暂时执行该程序，并在执行结束后再恢复身份。
# chmod +s 就是给某个程序或者脚本以SUID权限
# 必须执行下面项目不然wrapper会出现问题

sudo chmod +s /var/packages/ffmpeg/target/bin/ffmpeg
sudo chmod +s /var/packages/ffmpeg/target/bin/ffprobe
sudo chmod +s /var/packages/ffmpeg/target/bin/vainfo

# 注入ffmpeg代码
wget -O - https://gist.githubusercontent.com/BenjaminPoncet/bbef9edc1d0800528813e75c1669e57e/raw/ffmpeg-wrapper > /var/packages/VideoStation/target/bin/ffmpeg

# 更改权限
chown root:VideoStation /var/packages/VideoStation/target/bin/ffmpeg
chmod 750 /var/packages/VideoStation/target/bin/ffmpeg
chmod u+s /var/packages/VideoStation/target/bin/ffmpeg
```

# 后记

躺在床上看美剧真舒服。

![bed.png](/assets/img/posts/2022-08-20-nas-upgrade/bed.png)


