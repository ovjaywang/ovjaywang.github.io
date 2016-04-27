---
title: 无人机影像相关--拍摄、校正、平差、拼接
categories:
  - 什么都学一下
  - 学习log
tags:
  - 航空摄影测量
id: 816
date: 2015-11-04 21:12:46
---

无人机在近地拍摄全色或者多光谱数据，进行近景摄影测量，成图为后期的制图反演批处理提供数据来源。一般对像片各项操作，都需要知道像片的<font color="#FF0000">空中姿态及其位置</font>。然而，无人机自身系统受外界干扰较大，飞行姿态难以准确控制，同时缺乏监测飞机姿态的设备，处理无人机数据不能完全采用传统航摄流程进行作业。

关于摄影平台——无人机或带着单数码相机或组合相机，或带着遥感平台。常用的大面阵数码航摄相机，有DMC和UC（UCD UCX UCXp UCE）。DMC由4个全色4个多光谱镜头组成，4个中心投影拼接合成一副具有虚拟投影中心固定虚拟焦距的投影合成影像。UC4个全色波段对应3*3矩阵排列的9个CCD面阵，分别为<font color="#FF0000">4221</font>（分别对应二行左二（东南东北西南西北），二行左一（正南正北），二行左三（正东正西），二行左四（中心））.如下图所示，UC革新的将四个角的镜头设为主镜头（master cone）其他三个为从属镜头（slave cone）为的是控制相机区域，尽量覆盖完整视场角，其他3个镜头的影响依据它进行精确配准保证输出的影响具有强健刚性的严格中心投影。

![](http://ww3.sinaimg.cn/large/68eb7c93gw1exq55pwc5xj20l30cyjt1.jpg)

几个全色影像CCD之间存在重叠（航向258旁向262），通过重叠部分<font color="#FF0000">影像精确配准</font>，消除曝光时间带来的误差影响，生成一个完整的中心投影影像。图为四个镜头在不同时刻曝光，最后生成虚拟影像。值得一提的是，UC系统四个镜头虽然在不同时刻曝光拍摄，但是<font color="#FF0000">**选取了合适了速度并沿着第二行相机方向飞行**</font>时，每个曝光的相机都几乎在同一时空位置，因此得到的是一个完整的中心投影大幅面全色影像。

![](http://ww1.sinaimg.cn/large/68eb7c93gw1exq5bny66wj21180koag8.jpg)

关于数码航摄仪——若采用多相机进行拍摄，扩大VOF（view of field），或采取张大仰角或采取移轴摄影，都需要对多相机进行合成，<font color="#FF0000">组成一张虚拟影像</font>，来达到对后面平差拼接做前期准备，因此精度相当重要。

关于图像拼接——无人机搭载的数码相机虽然提高了影像的分辨率，但焦距有限视野较小，获得整个飞行区域的完整影像设计<font color="#FF0000">像片的拼接问题</font>。影像拼接涉及匀光匀色等色彩处理、对误差较大的图像进行粗提取、采用某些算计对影像提取特征点或者利用空间位置进行拼接等步骤。

这里可以参考的文献有，

[基于光束法自由网平差的无人机影像严格拼接](http://www.cnki.net/KCMS/detail/detail.aspx?QueryID=2&amp;CurRec=1&amp;recid=&amp;filename=TJDZ201205018&amp;dbname=CJFD2012&amp;dbcode=CJFQ&amp;pr=&amp;urlid=&amp;yx=&amp;uid=WEEvREcwSlJHSldRa1FhalpDTFRFYlpBY0d5OVdscE1hTndUdWJEd0ZyRGtESXlzS2lQZ1pVUkVRNVNOL1VwNmlRPT0=$9A4hF_YAuvQ5obgVAqNKPCYcEjKensW4IQMovwHtwkF4VYPoHbKxJw!!&amp;v=MTk3OTZVTDdKTVNmUGRMRzRIOVBNcW85RWJJUjhlWDFMdXhZUzdEaDFUM3FUcldNMUZyQ1VSTCtlWnVackZDam4=)&nbsp;主要总结了航摄基本方法（空三方式利用野外控制点加密测出大量控制点、通过灭点建立影像直线段与方位元素的关系、自检校光束区域平差利用定向参数解求外方位元素、量测直线段长度确定外方位元素、区域平差及SIFT+一定数量控制点针对不同尺度三维影像进行拼接、光束法区域平差+秩亏可以解决缺少控制信息的像片）

<div class="PoweredByWebStory" style="margin-top:15px;margin-bottom:10px">[](http://sns.juziyue.com/webinvite.php?u=94887)&nbsp;今天你[菊子曰](http://sns.juziyue.com/webinvite.php?u=94887)了么？</div>