---
title: 记静态ip的坑
categories:
  - 什么都学一下
  - 学习log
tags:
  - hadoop
  - 静态ip
id: 1117
date: 2016-04-01 16:06:32
---

明明在hadoop配置静态IP的时候被坑过一次，却完全想不起来怎么解决的。尤其是在ssh登录的时候需要记下host和ip的对应，才能直接输出主机名来访问局域网内的其他主机。
现在记下来。

<!-- more -->
1.原生linux+无线网卡/有线端口
ubuntu改interface和fedora改eth0-XXXXX都是极好的 里面对应的netmask ipaddress gateway dns1dns2（dnsnameserver）在resolv.conf配置dns信息
但是**比较easy**的纯粹做法直接在有线/无线的连接信息里面加这几个对应信息

![](https://ooo.0o0.ooo/2016/04/01/56fe2742cea08.png)

最最最easy的就是直接在路由器dhcp绑定mac静态分配，直接原生效果拔群。

2.虚拟机+有线端口/原生无线网卡

这种情况有点坑 虚拟机切记开启桥接模式 直接把主机和虚拟机按照局域网内的两台机器分配ip。这样当然也可以按照配置文件配置 切记写入MN_COTROLLED=yes 然后开启networkmanager管理网络，并使用

<pre class="brush: shell; gutter: true">systemctl enable netwoekmanager</pre>

开启开机启动。诚然，在连接信息直接配置也行。
3.虚拟机+外接无线网卡
呵呵 没找到解决方案。直接桥接驱动没法搞；直接nat不是我想要的结果。反正奏事上不了网ping不了局域网