---
title: 'Harris检测及匹配的python实现及PyInstaller编译成可执行文件exe'
categories:
  - 代码狗
  - 学习log
tags:
  - python
id: 575
date: 2015-05-12 01:48:13
---

<span style="color: #000000;">最近学习计算机视觉，初步学习了特征点的几个算子和描述子，最初级的是Harris和进阶的SIFT.囿于最近又想学python所以弄了一下它的数值计算的库scipy numpy和图形界面的库PIL和matplotlib..具体环境是：</span>

1.  <span style="color: #000000;">1python环境的搭建有两种方式，可以使用官方的[Python2.79](https://www.python.org/downloads/) 搭配上Enthought提供的一个轻量Free版本[canopy](https://store.enthought.com/downloads/#default)供学习和开发者使用（包括**NumPy, SciPy, Pandas, Matplotlib, IPython**）；或者直接使用[Scipy官网](http://scipy.org/install.html)推荐使用的Python其他搭载了Numpu和Scipy的发行版，我体验了Python(x,y)效果还是不错的</span>
2.  <span style="color: #000000;">PIL图像库，可在其[官网](http://www.pythonware.com/products/pil/#)下载最新版本1.1.7的PIL（虽然已经是2009年。。目前只支持2.X可在WIn LInux OX平台使用）</span>
3.  <span style="color: #000000;">Matplotlib在安装Numpy和Scipy时应该已经提供，若没有下载完成则可在其[网站](http://matplotlib.sourceforge.net/)免费获取</span>

## **<span style="color: #000000;">Harris特征检测</span>**

<span style="color: #000000;">安装完成后就可以进行Python图形可视化的编写。首先对Harris角点检测算法有个介绍。主要思想是，<span style="color: #ff0000;">如果像素周围存在多于一个方向的边，认为该点就是兴趣点，称为角点。</span></span>

<span style="color: #000000;"><span style="color: #ff0000;">角点的定义是：局部窗口沿各方向运动，均产生明显变化的点；图像局部曲率突变的点。<span style="color: #000000;">窗口函数可以选择创体内1窗体外0，或者高斯函数。窗口移动导致的图像变化可利用实对称矩阵的特征值分析，计算各角点的响应。其中一种方式是，利用最大的特征值进行判断，对特征值均非常大的，表述为各方向上变化均非常大；对特征值均非常小的，表述为各方向上无明显变化；若一个特征值大一个特征值小，则判定为边缘。另一种方法是，利用响应函数det (H)−αtrace(H)^2 或者λ0λ1/(λ0+λ1)。最后，利用阈值筛选出局部极大值。该算法具有光照不变性，旋转不变性，对比度部分不变。尺度变换不具有不变性。</span></span></span>

<pre class="brush: shell; gutter: true">from scipy.ndimage import filters
from PIL import Image
from numpy import *
from pylab import *

def compute_harris_response(im,sigma=3):
#计算harris响应函数
imx = zeros(im.shape)
filters.gaussian_filter(im,(sigma,sigma),(0,1),imx)
imy = zeros(im.shape)
filters.gaussian_filter(im,(sigma,sigma),(1,0),imy)

Wxx = filters.gaussian_filter(imx*imx,sigma)
Wxy = filters.gaussian_filter(imx*imy,sigma)
Wyy = filters.gaussian_filter(imy*imy,sigma)

Wdet = Wxx*Wyy - Wxy**2
Wtr = Wxx +Wyy

return Wdet / Wtr

def get_harris_points(harrisim,min_dist,threshold):

corner_threshold = harrisim.max() * threshold
harrisim_t = (harrisim &gt; corner_threshold) * 1

coords =array(harrisim_t.nonzero()).T

candidate_values = [harrisim[c[0],c[1]] for c in coords]

index = argsort(candidate_values)

allowed_locations = zeros(harrisim.shape)
allowed_locations[min_dist:-min_dist,min_dist:-min_dist]=1

filtered_coords = []
for i in index:
if allowed_locations[coords[i,0],coords[i,1]] == 1:
filtered_coords.append(coords[i])
allowed_locations[(coords[i,0]-min_dist):(coords[i,0]+min_dist),(coords[i,1]-min_dist):(coords[i,1]+min_dist)] = 0
return filtered_coords

def plot_harris_points(im1,im2,filtered_coords1,filtered_coords2):
figure()
gray()
im=concatenate((im1,im2),axis=1)
imshow(im)
shp1 = im1.shape[1]
for j in filtered_coords2:
jj = [j[0] , j[1] +shp1]
filtered_coords1.append(jj)
plot([p[1] for p in filtered_coords1],[p[0] for p in filtered_coords1],&#039;*&#039;)
axis(&#039;off&#039;)
show()

im01= array(Image.open(&#039;1.JPG&#039;))
im02= array(Image.open(&#039;2.JPG&#039;))
im1 = array(Image.open(&#039;1.JPG&#039;).convert(&#039;L&#039;))
im2 = array(Image.open(&#039;2.JPG&#039;).convert(&#039;L&#039;))
harrisim1 = compute_harris_response(im1)
harrisim2 = compute_harris_response(im2)
min_dist=7
threshold=0.05
filtered_coords1 = get_harris_points(harrisim1,min_dist,threshold)
filtered_coords2 = get_harris_points(harrisim2,min_dist,threshold)
print len(filtered_coords1)
print len(filtered_coords2)
plot_harris_points(im01,im02,filtered_coords1,filtered_coords2)

其中1.jpg和2.jpg可换成自己的图像。函数说明在注释中已说明。</pre>

![](http://7xlk6q.com1.z0.glb.clouddn.com/harris.PNG)

## 

## **harris特征匹配**

<span style="color: #000000;">角点检测器只能检测图像中的兴趣点，但是没有给出匹配角点的方法，这里需要对每个角点加上描述子信息，即角点周围的点的信息。描述子描述越好，寻找匹配对应点越好。Harris角点的描述子又周围像素块的灰度值以及用于比较归一化互相关矩阵构成。通常两个相同大小像素块的相关矩阵定义为函数f对像素块卷积的和。归一化互相关矩阵的一种变形可以定义为：</span>

![](http://7xlk6q.com1.z0.glb.clouddn.com/ncc自相关矩阵.PNG)

<span style="color: #ff0000;">其中，n表示像素块的数目，μ1和μ2表示每个像素块平均像素强度，σ1和σ2表示每个像素块的标准差。通过减去均值和标准差，该方法对图像亮度变化具有稳健性。</span>

<pre class="brush: shell; gutter: true">#_*_encoding:utf-8_*_
from scipy.ndimage import filters
from PIL import Image
from numpy import *
from pylab import *&lt;/code&gt;

def compute_harris_response(im,sigma=3):
#计算灰度图像中harris角点响应函数

#计算导数
imx = zeros(im.shape)
filters.gaussian_filter(im,(sigma,sigma),(0,1),imx)
imy = zeros(im.shape)
filters.gaussian_filter(im,(sigma,sigma),(1,0),imy)
#计算harris矩阵分量
Wxx = filters.gaussian_filter(imx*imx,sigma)
Wxy = filters.gaussian_filter(imx*imy,sigma)
Wyy = filters.gaussian_filter(imy*imy,sigma)
#计算特征值和迹
Wdet = Wxx*Wyy - Wxy**2
Wtr = Wxx +Wyy

return Wdet / Wtr

def get_harris_points(harrisim,min_dist,threshold):
#得到harris特征点集 harrisim是计算harris响应函数的灰度图像 min_dist为分割角点和图像边界最少像素数

#寻找高于阈值的候选点
corner_threshold = harrisim.max() * threshold
harrisim_t = (harrisim &amp;gt; corner_threshold) * 1
#得到候选点的坐标
coords =array(harrisim_t.nonzero()).T
#计算候选点响应值
candidate_values = [harrisim[c[0],c[1]] for c in coords]
#对候选点按照响应值进行排序
index = argsort(candidate_values)
#将可行点的位置保存到数组
allowed_locations = zeros(harrisim.shape)
allowed_locations[min_dist:-min_dist,min_dist:-min_dist]=1
#按照min_distance原则选择最佳Harris点
filtered_coords = []
for i in index:
if allowed_locations[coords[i,0],coords[i,1]] == 1:
filtered_coords.append(coords[i])
allowed_locations[(coords[i,0]-min_dist):(coords[i,0]+min_dist),(coords[i,1]-min_dist):(coords[i,1]+min_dist)] = 0
return filtered_coords

def get_descriptors(image,filtered_coords,wid):
#对返回的坐标点，获取2*wid+1个像素的值，即获取harris描述
desc=[]
for coords in filtered_coords:
patch = image[coords[0]-wid:coords[0]+wid+1,coords[1]-wid:coords[1]+wid+1].flatten()
desc.append(patch)
return desc

def match(desc1,desc2,threshold=0.5):
#特征点匹配，使第一幅图中的角点描述子利用归一化相关选取它在第二幅图像的匹配角点
n = len(desc1[0])
#每对点的距离
d = -ones((len(desc1),len(desc2)))
for i in range(len(desc1)):
for j in range(len(desc2)):
d1 = (desc1[i]-mean(desc1[i])) / std(desc1[i])
d2 = (desc2[j]-mean(desc2[j])) / std(desc2[j])
ncc_value = sum(d1 * d2) / (n-1)
if ncc_value &amp;gt;threshold:
d[i,j] = ncc_value
ndx = argsort(-d)
matchscores = ndx[:,0]

return matchscores

def match_twosided(desc1,desc2,threshold=0.5):
#两边对称版本的match
matches_12 = match(desc1,desc2,threshold)
matches_21 = match(desc2,desc1,threshold)

ndx_12 = where(matches_12 &amp;gt; 0)[0]
#去除非对称的匹配
for n in ndx_12:
if matches_21[matches_12[n]]!=n:
matches_12[n] = -1
return matches_12

def appendimages(im1,im2):
#拼接两幅图像
rows1 = im1.shape[0]
rows2 = im2.shape[0]
if rows1 &amp;lt; rows2:
im1 = concatenate((im1,zeros((rows2-rows1,im1.shape[1]))),axis=0)
elif rows1 &amp;gt; rows2:
im2 = concatenate((im2,zeros((rows1-rows2,im2.shape[1]))),axis=0)

return concatenate((im1,im2),axis=1)

def plot_matches(im01,im02,locs1,locs2,matchscores):
#显示一副有连接线的匹配图像
im3 = appendimages(im01,im02)
im4 = vstack((im3,im3))
imshow(im4)

cols1 = im1.shape[1]
rows1 = im1.shape[0]
for i,m in enumerate(matchscores):
if m&amp;gt;0:
plot([locs1[i][1],locs2[m][1]+cols1],[locs1[i][0],locs2[m][0]],&#039;c&#039;)
new_coords =[]
for j in filtered_coords1:
new_coords.append ([j[0] + rows1, j[1]])
for i in filtered_coords2:
new_coords.append ([i[0] + rows1, i[1] + cols1])
plot([p[1] for p in new_coords],[p[0] for p in new_coords],&#039;*&#039;)
axis(&#039;off&#039;)

min_dist = 7
wid = 6
threshold=0.1
im01=array(Image.open(&#039;1.JPG&#039;))
im02=array(Image.open(&#039;2.JPG&#039;))
im1 = array(Image.open(&#039;1.JPG&#039;).convert(&#039;L&#039;))
im2 = array(Image.open(&#039;2.JPG&#039;).convert(&#039;L&#039;))
harrisim = compute_harris_response(im1,5)
filtered_coords1 = get_harris_points(harrisim,min_dist,threshold)
print &#039;Harris Points in 1.JPG:&#039;+bytes(len(filtered_coords1))
d1 = get_descriptors(im1,filtered_coords1,wid)

harrisim = compute_harris_response(im2,5)
filtered_coords2 = get_harris_points(harrisim,min_dist,threshold)
print &#039;Harris Points in 2.JPG:&#039;+bytes(len(filtered_coords2))
d2 = get_descriptors(im2,filtered_coords2,wid)

print &#039;start matching&#039;
matches = match_twosided(d1,d2)
print &#039;Matched Points in 1.JPG and 2.JPG:&#039;+bytes(len(matches))
figure()
gray()
plot_matches(im01,im02,filtered_coords1,filtered_coords2,matches)
show()
</pre>

<span style="color: #000000;">其中输入图像在python程序的同目录，为两幅近似的图像。其结果一举不同的min_dist和threshold而改变。</span>
![](http://7xlk6q.com1.z0.glb.clouddn.com/sift.jpg)

&nbsp;

## ***.py文件打包成.exe可执行文件**

<span style="color: #000000;">这里使用的是PyInstaller将py文件打包。实际效果非常方便，会自动搜寻需要的库并合成。可到其[官网](https://pypi.python.org/pypi/PyInstaller/2.1)下载最新版2.1</span>

基本语法是

<pre class="brush: python; gutter: true">python pyinstaller.py [opts] yourprogram.py
</pre>

其中opts是可选项，可加入的参数为：

<pre class="brush: css; gutter: true">-F, –onefile 打包成一个exe文件。
-D, –onedir 创建一个目录，包含exe文件，但会依赖很多文件（默认选项）。
-c, –console, –nowindowed 使用控制台，无界面(默认)
-w, –windowed, –noconsole 使用窗口，无控制台</pre>

<span style="color: #ff0000;">**BUG**</span>

这里遇到了恼人的无可药救的无与伦比的bug。其中之一是

<pre class="brush: shell; gutter: true">no module named _ufuncs
</pre>

阅遍无数网页终于在[这里](https://github.com/pyinstaller/pyinstaller/issues/826)发现了解决方案，需要在PyInstall文件夹的hooks文件夹中添加一个寻找 _ufuncs的py文件。在[这里](https://github.com/pyinstaller/pyinstaller/blob/develop/PyInstaller/hooks/hook-scipy.special._ufuncs.py) ，命名为hook-scipy.special._ufuncs.py

<pre class="brush: python; gutter: true">#-----------------------------------------------------------------------------
# Copyright (c) 2013, PyInstaller Development Team.
#
# Distributed under the terms of the GNU General Public License with exception
# for distributing bootloader.
#
# The full license is in the file COPYING.txt, distributed with this software.
#-----------------------------------------------------------------------------

# Module scipy.io._ufunc on some other C/C++ extensions.
# The hidden import is necessary for SciPy 0.13+.
# Thanks to dyadkin, see issue #826.
hiddenimports = [&#039;scipy.special._ufuncs_cxx&#039;]</pre>

然后又发现另一个bug，这回是

<pre class="brush: python; gutter: true">no module named integrate</pre>

阅遍无数网页，依然没有结果。。偶然的在需要编译的py文件中加入了一句
`from scipy.integrate import *`
咦居然成功了。。。好吧。结果出来就和在python shell中运行的一样了。可以打包成一整个exe文件，也可以是一个文件夹，包含其他类库的。