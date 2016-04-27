---
title: 记录一次心酸而无终的悲催却又惊喜的python添加sift类库过程
categories:
  - 代码狗
  - 学习log
tags:
  - python
id: 580
date: 2015-05-12 17:17:45
---

基于上回对Harris的角点检测和匹配的过程，试想这学习一下另一种角点检测和匹配算法sift.
在《Python计算机视觉》书中，由于Python实现所有步骤的效率并不是很高，提到了VLFeat工具进行SIFT兴趣点的检测。

好的，那就去[官网](http://www.vlfeat.org/)找。它明确写着Windows Mac Linux都可用。太好了。但是主要提供的是c 和 MATLAB 的api，在python中只能利用shell进行接口的调用。将目录下的\bin\arch\sift.exe 添加到path环境路径，就能在cmd控制台使用shell。可是测试了几次，依据[例子](http://www.vlfeat.org/man/sift.html#EXAMPLES)或者普林斯顿的[抄袭版本](http://vision.cs.princeton.edu/pvt/SiftFu/SiftFu/SIFTransac/vlfeat/doc/man/sift.html)，都只能打开可视化的界面，不能输入需要的.sift或者frame图像。

好吧换个方案，听说Python有一个包装器。找到了供github下的[pyhton wrappers主页](https://github.com/mmmikael/vlfeat/tree/python-wrappers/python)（居然是寄生在MATLAB的插件下面）可是。。可是。在它的readme发现只能在linux和Mac OX使用。。书中亦提到包装器的安装需要非常高的技巧。好吧放弃。

再次寻找。发现一个python官网提供的一个[python插件](https://pypi.python.org/pypi/pyvlfeat)。直接可以利用setup.py insatll进行安装。好了进行安装

<pre class="brush: shell; gutter: true">python setup.py install
</pre>

居然出现了bug

<pre class="brush: shell; gutter: true">Unable to find vcvarsall.bat
</pre>

找到一些[讨论](http://stackoverflow.com/questions/2817869/error-unable-to-find-vcvarsall-bat)，大家都说是要安MinGW，找到它的官网，[这个](http://sourceforge.net/projects/mingw/files/Installer/)看起来不错。默认选择了安装路径，又安装了一堆.bin 的类库。输入

<pre class="brush: shell; gutter: true">setup.py install build --compiler=mingw32
</pre>

又出现了bug

<pre class="brush: shell; gutter: true">error: numpy/arrayobject.h: No such file or directory
</pre>

日了狗了。在[这里](http://stackoverflow.com/questions/14657375/cython-fatal-error-numpy-arrayobject-h-no-such-file-or-directory)说需要加点什么路径，然后又在[这里](http://stackoverflow.com/questions/14657375/cython-fatal-error-numpy-arrayobject-h-no-such-file-or-directory)发现了说是它的mingw已经很旧了不太能匹配新的版本的软件了。可以找到一个[非官方版本](https://github.com/develersrl/gccwinbinaries)。

好吧。开心的安上。再一次安了gcc等一系列乱七八糟的bin库。然后执行。。。shit。又是bug arraypobjects.h又找不到。。

好吧超出了我的技能点了。顺带在半路还捡到一个非官方的python类库，叫做[unofficial windows binaries for python extention package](http://www.lfd.uci.edu/~gohlke/pythonlibs/)
..

正在绝望之时。。居然在csdn发现了这本书的[源代码](http://download.csdn.net/detail/shidecai1230/8255391#comment)oh wtf。居然它的自带vlfeat能用。好吧到此结束了。直接用命令行好了。py中利用

<pre class="brush: python; gutter: true">os.system(&#039;cmd&#039;)</pre>

使用命令行进行编写。