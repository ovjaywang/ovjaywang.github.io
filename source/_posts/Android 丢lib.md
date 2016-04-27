---
title: 'Android 丢lib'
categories:
  - 代码狗
  - 学习log
tags:
  - mark
  - 吐槽
id: 200
date: 2013-05-20 03:59:55
---

android极其容易发生的NoClassDefFoundError错误 
在csdn和stackflow都有人提问。。。最终的解决方案是：
1.建一个libs包（名字不楞换）libs右键->Build path->Use as source folder 并将所需的jar包丢进去 jar包也要加入buildpath
2.工程右键 java build path ->order and export 中勾选android private library 并置顶