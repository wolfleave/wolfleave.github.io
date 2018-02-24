---
layout: post
title: "fiddler监控localhost(或127.0.0.1)的请求"
description: 
image: 
category: 'blog'
tags:
- Network
- tools
---

前几天在调试本地服务的时候，发现Fiddler抓不到（localhost和127.0.0.1）包。

查找相关资料，发现官方提供了以下方法：
<a href="http://docs.telerik.com/fiddler/Configure-Fiddler/Tasks/MonitorLocalTraffic">MonitorLocalTraffic</a>

由于不想用机器名，也不想在域名里面使用fiddler字样。所以尝试其他方式。

发现采用修改hosts的方式，可以达到效果

- 找到hosts文件 C:\Windows\System32\drivers\etc\hosts
- 在里面加上一条记录 127.0.0.1 xxx.xxx.com
- 在程序里面使用xxx.xxx.com访问服务
- 打开Fiddler，发现可以抓到对应的包了


-----
