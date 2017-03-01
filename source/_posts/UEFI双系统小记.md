---
title: UEFI双系统小记
date: 2017-02-20 23:44:55
tags:
---

喷血了。一个小误会居然装了一个晚上。作为一个6年前就装过双系统的薛薇有些池乳。



正题

------

# 一些不会犯的错

- 安装的时候不要选双系统也不要选抹掉原系统啊。。不然哪来的双系统
- 分区确定好后记得把引导装在boot里面。



# 一些可能犯的错误

- 这次居然遇到了。。分区按别的教程给boot5120M。。居然不够？？最终提示无足够空间
- 一般的分区4个都不要少就好了-/ /boot swap /home(/usr也行)



# UEFI下的双系统



按照这两个来就相当好办了。

[[UEFI引导的系统下装双系统解决方案](http://blog.csdn.net/lyhdream/article/details/52266523)](http://blog.csdn.net/lyhdream/article/details/52266523)



[UEFI模式下安装Windows 10、Ubuntu 16.04 LTS双系统教程](http://weibo.com/p/1001603968287473010231?from=page_100606_profile&wvr=6&mod=wenzhangmod&sudaref=www.google.com&retcode=6102)



之前的系统全是在mbr上安的，按照以往是吧iso里的几个文件解压出来放在c盘里，使用easybcd添加引导安装，再自动搜索添加Shaun该系统启动目录，双系统的引导直接放在win下。现在uefi的引导专门搞了一个盘来放置引导和乱七八糟的恢复文件。使用开源的uefi修改器相当顺滑。



TensorFlow1.0搞起。

然后发现了一个相当顺滑的hexo同步[方案](https://www.zhihu.com/question/21193762/answer/147374293)。不用更新完一次用一次git命令和hexo命令了。vps必须要摆在日程里了。自动化服务器方案是以后记录笔记、更新博客、抄段子的必备手段了。





