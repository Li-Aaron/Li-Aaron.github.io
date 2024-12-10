---
layout: post
title: Leanote APP with proxy
date: 2022-09-06 20:00:00.000000000 +08:00
description: 本文介绍如何让Leanote支持proxy（原软件不支持）。
author: aaron-li
categories: [工具指南]
tags: [leanote, proxy]
---

本文介绍之前没有解决的Leanote APP无法使用代理服务器问题。  
关于Leanote其他部分介绍详见：[告别印象笔记，使用Leanote自建云笔记]({{site.url}}/2021/08/leanote-replace-evernote/)

修改`resource/app/node_modules/needle/lib/needle.js`文件中如下部分
```js
    var config = {
      base_opts       : {},
      // proxy           : options.proxy,
      proxy           : 'http://your.proxy.server:12345',
      output          : options.output,
      encoding        : options.encoding || (options.multipart ? 'binary' : defaults.encoding),
      decode_response : options.decode === false ? false : defaults.decode_response,
      parse_response  : options.parse === false ? false : defaults.parse_response,
      follow          : options.follow === true ? 10 : typeof options.follow == 'number' ? options.follow : defaults.follow,
      timeout         : (typeof options.timeout == 'number') ? options.timeout : defaults.timeout
    }
```
