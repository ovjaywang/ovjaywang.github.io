---
title: MFC中添加控制台调试
categories:
  - 代码狗
  - 学习log
tags:
  - mfc
  - 控制台
id: 573
date: 2015-05-07 02:56:39
---

<span style="color: #000000;">1.在需要添加控制台调试的cpp页面加上**#include "conio.h";**</span>

<span style="color: #000000;">2.在启动mfc窗体的页面加入**AllocConsole();**</span>

<span style="color: #000000;">3.在需要输出的地方输入 **cprintf()** ,其中是char类型。若是vs2005之后，应改成_cprintf();</span>

<span style="color: #000000;">4.若需要关闭控制台窗口，使用 FreeConsole();</span>