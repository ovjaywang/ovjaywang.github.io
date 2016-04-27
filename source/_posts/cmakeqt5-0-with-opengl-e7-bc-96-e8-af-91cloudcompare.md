---
title: Cmake+qt5.0 with Opengl编译CloudCompare
categories:
  - 代码狗
  - 学习log
tags:
  - 点云
  - cmake
id: 558
date: 2015-04-21 21:31:35
---

最近有使用到CloudCompare，是一个开源的可视化显示三维点云的程序。
[![](http://ww4.sinaimg.cn/large/68eb7c93gw1erdnmdd9qqj202j02sjrc.jpg)](http://ww4.sinaimg.cn/large/68eb7c93gw1erdnmdd9qqj202j02sjrc.jpg)

*   [项目首页](http://www.danielgm.net/cc/ "Cloudcompare项目首页")
*   [Github](https://github.com/cloudcompare/trunk "git地址")
*   [wiki](http://www.cloudcompare.org/doc/wiki/index.php?title=Main_Page "wiki")

CloudCompare功能较为丰富，可以处理3d点云数据，可以比较两幅3d点云图像（例如激光扫描获取的图像）亦或者比较一副点云图像和一副三角网。它基于独特的八叉树结构使得性能极大的提升，一般可同时处理超过一千万点云数据，最高可在2Gb的内存下处理1.2亿个点云数据。该软件有可执行文件exe,也可以在linux、mac、windows中编译后使用最新的功能。项目首页有exe、7z版本的完整版、7z简化版的CCviewer，本次编译使用Windows8.1 64bit系统、MS Visual Studio2013 64bit，使用项目的github代码在Cmake下编译。

编译环境：

1.  [Qt5.4](http://www.qt.io/download-open-source/#section-2)  Qt中需要下载一个Qt 5.4.1 for Windows 64-bit (VS 2013, OpenGL, 711 MB) (info)来安装Qt的环境，还需要下载一个Visual Studio Add-in 1.2.4 for Qt5 (156 MB) (info)来在VS中进行可视化编辑。
2.  [Cmake ](http://www.cmake.org/download/)Cmake是一个根据不同系统和配置进行编译的工具，在windows、linux和mac都可以使用。windows下64位系统可以正常安装32位版本。
3.  利用svn checkoutgithub中的最新项目。这里地址为G:\CCViewer\trunk

参考[这里](http://www.cloudcompare.org/doc/wiki/index.php?title=Compilation "Compilation")开始编译：

[![](http://ww2.sinaimg.cn/large/68eb7c93gw1erdnius1ayj20uo0o9qaq.jpg)](http://ww2.sinaimg.cn/large/68eb7c93gw1erdnius1ayj20uo0o9qaq.jpg)

1.  在编译好的文件G:/build中打开sln，先重新生成解决方案。打不开。参考[这里](http://www.danielgm.net/cc/forum/viewtopic.php?t=992)，先右键在项目属性中选择debug（编译）将命令那一栏改成debug文件夹下的CloudCompare.exe文件。
2.  最后找不到各种*.dll，在qt的安装文件夹中找到，放到项目的debug文件夹中，若是qcc是完整版，ccviewer是浏览器简版，均是放在相应的debug文件夹下。

这样就完成了编译，若需要在vs中编辑qt的窗口，需要在qt的菜单栏选项中指定qt版本和相应的地址，参考[这里](http://www.bogotobogo.com/Qt/Qt5_Visual_Studio_Add_in.php)，点击Add，写入 版本号5.4.1和地址E:\Qt\Qt5.4.1\msvc2013_64_opengl,这样双击项目中的ui就可以在qt的可视化编辑器中编写qt窗口。。里面的函数封装好复杂，，窝先匿去研究一会儿了。[![](http://ww4.sinaimg.cn/large/68eb7c93gw1erdl1rx0bpj214e0sj13b.jpg)](http://ww4.sinaimg.cn/large/68eb7c93gw1erdl1rx0bpj214e0sj13b.jpg)