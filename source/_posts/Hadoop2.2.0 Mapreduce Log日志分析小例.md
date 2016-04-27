---
title: Hadoop2.2.0 Mapreduce Log日志分析小例
categories:
  - 代码狗
  - 学习log
tags:
  - hadoop
id: 531
date: 2014-12-12 02:17:26
---

这个任务主要是在搭建好Hadoop2.2.0环境的基础上，模仿sample中的wordcount程序，手动编写一个Mapreduce的对日志文件分析的函数。该日志文件是模拟搜索引擎记录所有用户的查询，当有一个用户使用搜索引擎进行搜索时，日志文件会记录这样一条记录：（搜索时间、搜索关键字、用户IP）。其中，日志文件的格式是：

*   时间+搜索词+IP地址
*   2011-10-26 06:11:35 云计算 210.77.23.12
*   2011-10-26 06:11:45 Hadoop 210.77.23.12

利用Mapreduce模型对日志文件进行分析，需要完成的任务有：

*   所有日志中出现次数最多的50个关键词；
*   特定日期区间日志中出现次数最多的50个关键词；
*   特定IP地址区间（假定前3段相同，区间在4段中设定）日志中出现次数最多的50个关键词

任务主要流程包括：

*   按照一定的规则随机生成并将日志文件写入DFS中。
*   将日志文件进行字符串筛检，提取出所需字段并计数
*   将累计的词数进行排序

![](http://ww1.sinaimg.cn/large/68eb7c93jw1en69s477ktj20fc0atdge.jpg)

编程之前，研究了一下eclipse-hadoop的插件，其作用是可以快速的查看DFS中的新增的文件，即写入日志，生成日志都能快速的查看。下载jar包，将之放到eclipse目录下的dropins里。启动eclipse后再preference里可以配置hadoop的路径。

![](http://ww4.sinaimg.cn/large/68eb7c93jw1en69y207dij20hm055dga.jpg)

在导航中打开map-reduce的窗口，右键新建一个HadoopLocation，如图：

![](http://ww3.sinaimg.cn/large/68eb7c93jw1en69yxctfuj20s20enq5z.jpg)        这里Mapreduce和DFS的port分别是mapred-site.Xml和core-site.xml中配置的端口。这样就可以在Program Explorer 中看到dfs的文件。

![](http://ww2.sinaimg.cn/large/68eb7c93jw1en69zmtefxj205v04e3yj.jpg)此外，加入了插件后还可以直接建立Mapreduce的项目，这里就不详述了。

![](http://ww1.sinaimg.cn/large/68eb7c93jw1en6a08tm2hj20gz09174x.jpg)
变成之前对Hadoop2.2.0的Mapreduce做一个简介。之前版本的Mapreduce是由Jobtracker和TaskTracker来管理的，之后出现了Yarn模型的版本，其要义是优化了每次Job都要经过JobTracker分配的结构，通过NodeManager管理节点任务，通过ResourceManager管理资源的分配。

![](http://ww1.sinaimg.cn/large/68eb7c93jw1en6a10wwoxj20cv09ogme.jpg)

该系统程序的编写建立在系统自带的Wordcount demon上，通过重写Mapper类下的Map函数和Reducer类下的Reduce函数来读入分布式文件的数据，并进行分布式处理。其中，排序并找出前K个任务由两个Mapreduce完成。第一个任务Map负责读取分布式文件，截取字符串，并按任务需求过滤log信息；第一个任务Reduce负责对截取出的搜索词计数。第二个任务的Map负责封装新的词频和搜索词；第二个任务的Reduce负责将K-V键值对加入TreeMap并进行排序，最终输出TopK文件。
操作者需要在一台配置好Hadoop的Linux全分布式节点的环境下，在控制台通过hdfs命令输入7项参数（其中5项必选）：1.输入log日志文本路径 2.第一个Mapreduce输出的词频统计文件路径 3.第二个Mapreduce输出的词频统计顺序排列的文件路径（中间过程）4.第二个Mapreduce输出的最高词频统计结果的文件路径 5.任务需求 6.参数1 7.参数2 进行系统的操作，最终在在hdfs文件系统中找到生成的结果，一般以ABC-part-00000命名。参数输入错误或者系统崩溃会自动停止程序。

![](http://ww3.sinaimg.cn/large/68eb7c93jw1en6a20gi08j20o20bwmxy.jpg)

之前叙述了系统由两部分构成，其中第一部分不作为系统的主体框架，只作为普通JavaApplication处理，另一部分则具体分为MAP1，REDUCE1,MAP2,REDUCE2来处理，以下是系统设计的具体框架。
第一部分较简单，通过调用Hadoop写入文件的API，再加上随机函数分别生成搜索词、IP和日期来实现。第二部分则通过编写两个函数入口run、两个map读取文件和两个reduce实现相应功能来组成。本系统的主要类如下：

![](http://ww1.sinaimg.cn/large/68eb7c93jw1en6a3grqz1j2055053jrl.jpg)

其中，Config主要存放系统全局的参数，例如日志行数，日志起止时间，topK个数等。
DFSWritter负责第一个主要任务写入数据，包含以下函数：

![](http://ww2.sinaimg.cn/large/68eb7c93jw1en6a4d8g9pj205902cq2w.jpg)

写入数据构成如下：

![](http://ww1.sinaimg.cn/large/68eb7c93jw1en6a51ij4xj20sr080q3h.jpg)

其中获取随机日期的getRandomDate函数，其中在config中指定各参数

![](http://ww2.sinaimg.cn/large/68eb7c93jw1en6a5w98dvj20ke07fgn5.jpg)

获取随机搜索词的函数，其中Config可以配置搜索词最短和最长词数。

![](http://ww1.sinaimg.cn/large/68eb7c93jw1en6a6p3he8j20nw07l0u9.jpg)

获取随机IP的函数，其中IP最大最小值可在config中指定。

![](http://ww1.sinaimg.cn/large/68eb7c93jw1en6a7e0i34j20ne04j0tc.jpg)

Writter函数，负责写入log数据，已指定了写入的路径。
![](http://ww1.sinaimg.cn/large/68eb7c93jw1en6a856bgrj20i80eatbf.jpg)

在main调用写入函数即可。第二个任务是进行日志文件的筛查、排序、查找最大值。主要用到的类有：

![](http://ww3.sinaimg.cn/large/68eb7c93jw1en6a9b6vr0j204w03a74d.jpg)

LogAnalysiser21、LogAnalysiser21分别负责第一个、第二个Job的入口，继承了Configured类，实现配置Map和Reduce，若有Combiner或者partitioner也在这里进行配置。它们均重写了run函数进行mapreduce的启动。二者可以合并，这里分开让结构更清晰。
MapClass MapClass2分别是两个job的映射，读取输入的文件并传给Reduce。它们均继承自Mapper并重写map函数。
ReduceClass ReduceClass2 分别是两个Job的处理细节，收到来自Map的信息并作整合处理。它们均继承自Reducer并重写reduce函数。
如下图是该任务的系统流程：

![](http://ww4.sinaimg.cn/large/68eb7c93jw1en6aa7zbrjj20n60c8409.jpg)

其中logAnalysis21负责对参数中mode进行判断，加载第一个job，主要函数run'为：

![](http://ww4.sinaimg.cn/large/68eb7c93jw1en6aapphpij20qz0byq4p.jpg)

![](http://ww4.sinaimg.cn/large/68eb7c93jw1en6ab82ey0j20eg05yt9u.jpg)

MapClass中map函数如下，主要负责依据不同的需求对日志文件进行筛选

![](http://ww2.sinaimg.cn/large/68eb7c93jw1en6ac19sqjj20js0ct418.jpg)

![](http://ww1.sinaimg.cn/large/68eb7c93jw1en6acnbc0lj20k109djt9.jpg)

ReduceClass 中reduce函数如下，主要负责计数并输出。

![](http://ww4.sinaimg.cn/large/68eb7c93jw1en6ad4qxadj20jg04daap.jpg)

LogAnalysis22负责加载第二个job，run函数如下：

![](http://ww4.sinaimg.cn/large/68eb7c93jw1en6ady0bw1j20o20dydjd.jpg)

MapClass2负责读取上一个job的wordcount文件，并对数据键值对进行改写，将词频改写为填充到10位的long，由于排序时同词频会被删除，因此将key改写为（词频+搜索词）value改写为（搜索词） 。Map函数如下：

![](http://ww2.sinaimg.cn/large/68eb7c93jw1en6aexl5nyj20o1039dgl.jpg)

Reduce2负责读取传来的键值对，写入TreeMap，利用TreeMap的排序器自动对传来的改写（搜索词+词频）进行排序。这里用到了TreeMap自动根据key值进行排序的方法，但是key不能重复，因此在Map中改写起到了作用。同时，这里重写了TreeMap的Comparator函数，默认依据是从小到大排序。若指定了排序器Comparator，这里就可以按照从达到小进行排序了。排序完成后截取key前10位即为词频，value即为搜索词。Reduce函数主要如下：

![](http://ww3.sinaimg.cn/large/68eb7c93jw1en6affldqej20lv0dnjto.jpg)

这里context输出只是中间结果sort，需要找到前N个高频搜索词，因此指定新的输出：

![](http://ww4.sinaimg.cn/large/68eb7c93jw1en6afzezfbj20ix0720u3.jpg)

在安装配置好hadoop的分布式系统启动hadoop：

&nbsp;

<pre class="brush: bash; gutter: true">./start-dfs.sh ./start-yarn.sh</pre>

![](http://ww1.sinaimg.cn/large/68eb7c93jw1en6b2rf3ndj20hz05at9r.jpg)

在Config.java确认配置参数

![](http://ww2.sinaimg.cn/large/68eb7c93jw1en6b3i6ungj207g07qmya.jpg)

运行DFSWritter.java文件，将日志写入HDFS分布式文件系统，这里写入50个作为测试。

![](http://ww4.sinaimg.cn/large/68eb7c93jw1en6b41kibtj20ci01vdfx.jpg)

输入hdf格式的命令，先列出log日志的结果

![](http://ww4.sinaimg.cn/large/68eb7c93jw1en6b4i2mh6j20cn0cvgnt.jpg)

![](http://ww2.sinaimg.cn/large/68eb7c93jw1en6b4wispqj20990bg0uy.jpg)

将Java程序导出成JAR到本地路径/usr/workspace/jar/logAnalysis2.0.jar。

![](http://ww4.sinaimg.cn/large/68eb7c93jw1en6b5hlroyj20h709bgmn.jpg)

在命令行hdf命令，并带上必要的参数。首先运行需求一

![](http://ww4.sinaimg.cn/large/68eb7c93jw1en6b6bmiaoj20i101y74m.jpg)

两个Mapreduce一气呵成完成，分别查看wordcount\sort\topK,
查看Wordcount，可以看到按需统计的词频

![](http://ww2.sinaimg.cn/large/68eb7c93jw1en6b6um7e7j20hy0buq49.jpg)

查看sort，其中大的值排在了最后。

![](http://ww3.sinaimg.cn/large/68eb7c93jw1en6b7psqyhj20fx0bv3yt.jpg)

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

![](http://ww3.sinaimg.cn/large/68eb7c93jw1en6b84efm9j203x0ad749.jpg)

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

最后查看topK结果，即截取了sort的最后的8项并倒序输入。

![](http://ww4.sinaimg.cn/large/68eb7c93jw1en6b9u4yjyj20hp06o3z6.jpg)

运行第二个需求，在参数五中输入2，并写上起止IP

![](http://ww3.sinaimg.cn/large/68eb7c93jw1en6bad6w83j20i301474g.jpg)

运行结果的wordcount、sort、topK文件分别如下

![](http://ww2.sinaimg.cn/large/68eb7c93jw1en6bb6k4j8j20gg0a2mxn.jpg)

最后运行第三个需求，参数中带入3 和起止时间。

![](http://ww1.sinaimg.cn/large/68eb7c93jw1en6bbm6hi0j20i101k74j.jpg)

再次分别查看wordcount sort topk文件，比对前文的全log发现完全符合。

![](http://ww2.sinaimg.cn/large/68eb7c93jw1en6bcbhaifj20fi0cxmxr.jpg)

这样，一个建立在Hadoop的Mapreduce小例就完成了。

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;