---
title: jQuery的两个坑
date: 2016-05-10 01:23:30
tags: jQuery
categories:
  - 代码狗
  - 学习log
---

许久没用jquery，今天碰到两个坑。

第一个是live()这个函数，在学习很早的资料，表示用来为被选元素附加一个或多个事件处理程序,形式为

```javascript
$(selector).live(event,data,function)
```
例如为button绑定一个单击事件，event为事件名，data为可选的传参，function为自定义函数。
测了半天没开F12一直以为代码有问题，，后来一看原来jquery1.9+以上不支持live了。先改成了delegate代理，后来又改成on,真特么烦人。  

应该值得注意到的是，改成on后，触发的事件在容器对象上，而live返回值在事件触发的对象上。
[这里](http://jquery.com/upgrade-guide/1.9/#live-removed)是官方解释

****

一直以为$(selector)能抓到所有的id啊。。。原来好像dom树的id不能明明一样的。要对类似的组件操作类似的方法，例如对tr表单下的td采用同个样式，a使用类似的连接，可以使用$(.class)的方式获取。