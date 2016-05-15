---
title: jQuery的三个坑
date: 2016-05-10 01:23:30
tags: jQuery
categories:
  - 代码狗
  - 学习log
---

许久没用jquery，今天碰到三个坑。小记一下
<!-- more -->
第一个是live()这个函数，在学习很早的资料，表示用来为被选元素附加一个或多个事件处理程序,形式为

```javascript
$(selector).live(event,data,function)
```
例如为button绑定一个单击事件，event为事件名，data为可选的传参，function为自定义函数。
测了半天没开F12一直以为代码有问题，，后来一看原来jquery1.9+以上不支持live了。先改成了delegate代理，后来又改成on,真特么烦人。  

应该值得注意到的是，改成on后，触发的事件在容器对象上，而live返回值在事件触发的对象上。
[这里](http://jquery.com/upgrade-guide/1.9/#live-removed)是官方解释

****

一直以为``$(selector)``能抓到所有的id啊。。。原来好像dom树的id不能明明一样的。（但在1.4-版本测试是可以的，不知道是改进了对id选择器的定义和方法加载，明确了id唯一这个特征）  
要对类似的组件操作类似的方法，例如对tr表单下的td采用同个样式，a使用类似的连接，可以使用``$(.class)``的方式获取。

****

jquery1.9+以后不支持.attr属性的设置了，改成了.prop .妈蛋啊怎么改这么多好烦啊简直排错都不知道什么bug
炸裂看看1.9+的[更新](http://www.ppblog.cn/jquery1-9live.html)  
1. true false两个选项的属性用prop
2. 添加属性名称该属性就生效用prop
3. 其他可以用attr