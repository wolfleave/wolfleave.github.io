---
layout: post
title: "Cookie 及HttpWebResponse中的Cookie"
description: 
image: 
category: 'blog'
tags:
- Network
- Http
---

Http协议中用于Cookie机制的头部有（Cookie  Set-Cookie）。
Cookie在请求头中，携带一些客户端的信息，供服务端认识或者了解它。例如：
Cookie: x-auth-token=xxx; x-auth-app=xxx; client_version=xxx

可以将多个Cookie一起发送，像上面那样。每一个Cookie是一个键值对，用“=”连接。
多个Cookie用"; "间隔

Set-Cookie是响应头中服务端发送给客户端，让客户端保存的Cookie信息。例如：
Set-Cookie: pt_token=001677bf97104e12a89f62b5176b234234; Domain=.seewo.com; Path=/; HttpOnly

Set-Cookie包含多个属性。
- NAME=VALUE
- expires=DATE
- path=PATH
- domain=域名   指定使用此Cookie的域名  **只有发送请求的域名和指定的域名相同或者是指定域名的子域名时，才能在HttpWebResponse中的Cookies字段中拿到服务端返回的Cookie**
- Secure
- HttpOnly    此属性表示，Cookie不能被JavaScript访问

使用HttpWebRequest发送请求，HttpWebResponse获取请求。HttpWebResponse会对响应报文中的Cookie信息进行解析，保存在Cookies字段中

-----
