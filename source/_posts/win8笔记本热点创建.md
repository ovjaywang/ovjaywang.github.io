---
title: win8笔记本热点创建
categories:
  - 软件测评
  - 软文
tags:
  - 热点
id: 187
date: 2013-05-17 09:18:46
---

-netsh wlan show drivers
查看是否支持承载网络
-netsh wlan set hostednetwork mode=allow ssid=wlanname key=password
设置密码和名字
-netsh wlan start hostednetwork
启动热点
-netsh wlan stop hostednetwork
关闭热点
我不会告诉你这个会更加方便
http://jingyan.baidu.com/article/48b558e371b8e87f38c09a8e.html