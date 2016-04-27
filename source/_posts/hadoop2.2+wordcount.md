---
title: hadoop2.2.0+32bit centos6.5 全分布安装配置wordcount randomwritter
categories:
  - 代码狗
  - 学习log
tags:
  - hadoop
id: 507
date: 2014-11-12 15:15:31
---

hadoop简直神烦，在centos64位中0.2.0和1.2都在安装的最后一步出了问题。node和name都启动了，log没有错误，就是report里各项均为零。折腾了好几天换到2.2.0需要编译才能使用同样是因为各种dependencies版本不兼容编译不通过。最终确定下来的配置是

*   centos6.5的32位系统3台，其中两台安装在同一台物理机的虚拟机上 &amp;另一台物理机则安装第三个centos虚拟机
*   两台机子分别是：
*   master 192.168.199.111
*   slave1 192.168.199.212
*   slave1 192.168.199.184
*   机子间的网络靠路由器连接的内部IP，由于内部分配的IP不是静态的，需要改成静态的（需要保证ip不会冲突）

一 **在VMware下安装centos6.5 <span style="color: #ff0000;">32bit</span>**

 这个自己解决，在[http://wiki.centos.org/Download](http://wiki.centos.org/Download) 中选择镜像下载centos6.5，centos7由于刚发布，有些兼容不好。在安装的时候，自定义硬件-网络适配器 选择桥接模式，复制物理网络状态不要勾选。

![](http://ww3.sinaimg.cn/large/a5d0ed33gw1em886n10obj2079043q35.jpg)

安装默认是的是NAT模式，与主机共用同一个IP地址，这样虚拟机下无法分离出更多的IP。安装后启动centos6.5，键入

<pre class="brush: bash; gutter: true">su root</pre>

输入密码，进入root最高权限用户。在键入

<pre class="brush: bash; gutter: true">vi /etc/hosts</pre>

可将原来的清空，在hosts文件中添加本机的主机名和IP以便不同节点之间识别，我的如下

<pre class="brush: bash; gutter: true">192.168.199.111  master
192.168.199.212  slave1
192.168.199.184  slave2</pre>

更改主机名

<pre class="brush: bash; gutter: true">vi /etc/sysconfig/network</pre>

我的主机更改如下

<pre class="brush: bash; gutter: true">NETWORKING=yes
HOSTNAME=master</pre>

更改内部自动分配的IP地址

<pre class="brush: bash; gutter: true">vi /etc/sysconfig/network-scripts/ifcfg-eth0</pre>

如下图，其中IPADDR 192.168.199需要与物理机的前缀相同，111自己设一个没有冲突的ip地址，netmask和gateway和物理机完全一致。这些可以在物理机的命令行窗口执行命令

<pre class="brush: bash; gutter: true">ipconfig</pre>

进行查看。再键入

<pre class="brush: bash; gutter: true">service iptables status</pre>

查看虚拟机防火墙状态。若显示firewall is off则表示防火墙已关闭，若显示有内容，则输入

<pre class="brush: bash; gutter: true">chkconfig iptables off</pre>

关闭防火墙自启动，再输入

<pre class="brush: bash; gutter: true">service iptables stop</pre>

关闭防火墙。同时关闭物理机的防火墙和安全软件，这时候可以用ping 测试一下几台配好的机子是否相通。

二 **配置ssh无密码访问**
ssh是为了在不同节点之间传输文件和消息不用频繁的输入密码。在centos6.5中默认集成了ssh，因此不用安装ssh服务。首先在主节点master上键入

<pre class="brush: bash; gutter: true">ssh-keygen -t rsa</pre>

生成主节点信息的rsa密钥。进入/root/.ssh隐藏目录可发现生成了一个id_rsa和id_rsa.pub,将主节点信息注入授权key

<pre class="brush: bash; gutter: true">cd /root/.ssh
ls
cp id_rsa.pub authorized_keys
cat id_rsa.pub&gt;&gt;authorized_keys</pre>

对authorized_keys进行附权操作（可省略，针对新建立的非root权限用户，例如很多例子里的hadoop用户）

<pre class="brush: bash; gutter: true">chmod 600 authorized_keys</pre>

配置sshd_config文件，并将下列项的注释去掉(其余物理机也要进行这样的操作)

<pre class="brush: bash; gutter: true">vim /etc/ssh/sshd_config
 RSAAuthentication yes
PubkeyAuthentication yes
AuthorizedKeysFile     .ssh/authorized_keys</pre>

将包含master密钥的授权key发送给slave1节点。

<pre class="brush: bash; gutter: true">scp  root@slave1 :/root/.ssh/authorized_keys.pub /root/.ssh/</pre>

这里会需要输入slave1的root密码，进行确认。在slave1登陆，同样的步骤生成id_rsa和id_rsa.pub。

<pre class="brush: bash; gutter: true">ssh-keygen -t rsa
cd /root/.ssh
ls
mv id_rsa.pub slave1.pub
cat slave1.pub&gt;&gt;authorized_keys</pre>

将包含master和slave1密钥信息的文件传给slave2（此后每加一个节点都需要这么做，把新生成的密钥加入到先前所有人信息的authorizd_keys中）

<pre class="brush: bash; gutter: true">scp  root@slave2 :/root/.ssh/authorized_keys.pub /root/.ssh/</pre>

重复上面的操作，最后将slave2生成的authorized_keys.pub传回给master和slave1.

<pre class="brush: bash; gutter: true">scp  root@master :/root/.ssh/authorized_keys.pub /root/.ssh/
scp  root@slave1 :/root/.ssh/authorized_keys.pub /root/.ssh/</pre>

最后重启ssh服务

<pre class="brush: bash; gutter: true">service sshd restart</pre>

这时候可以在master上直接键入

<pre class="brush: bash; gutter: true">ssh slave1</pre>

登陆slave1主机，第一次登陆的时候需要输入root密码，之后就不需要了。这样就可以直接对slave1的文件进行读写操作，记得退出的时候要输入exit

三 **JDK和Hadoop路径的安装 **
这两个文件的安装都是下载和路径配置，跟目录分别为

<pre class="brush: bash; gutter: true">/usr/java
/usr/hadoop</pre>

这两个路径简单的目的就是为了以后码得少一点
**Java**
Java最新版本的路径可在[http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)[
](http://www.java.com/zh_CN/download/manual.jsp "java8")下载java8或java7，这里选择的是jdk7，即jdk-7u15-linux-i586.tar.gz。可在物理机下载后直接复制到虚拟机桌面，解压、复制、改文件名

<pre class="brush: bash; gutter: true">tar -zxf jdk-7u15-linux-i586.tar.gz
ls
cp -rf jdk1.7.0_15 /usr
mv jdk1.7.0_15 java</pre>

配置一下全局的环境

<pre class="brush: bash; gutter: true">vi /etc/profile</pre>

在文件末添加如下代码，配置JAVA安装目录，JRE目录，CLASSPASTH和PATH

<pre class="brush: bash; gutter: true">JAVA_HOME=/usr/java/
JRE_HOME=/usr/java/jre
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
export JAVA_HOME JRE_HOME PATH CLASSPATH</pre>

里面的冒号与windows下系统环境的分号相通。更新一下配置的java环境，查看java版本，出现如下则配置成功。

<pre class="brush: bash; gutter: true">source /etc/profile
java -version</pre>

![](http://ww2.sinaimg.cn/large/68eb7c93jw1em9y51jqffj20d001zaa8.jpg)

**hadoop**
hadoop2.2.0在64位下需要源码编译，比较繁琐且容易出错，在32位下则可直接使用。因此选择在[http://www.apache.org/dyn/closer.cgi/hadoop/common/](http://www.apache.org/dyn/closer.cgi/hadoop/common/)官方网站下下载2.2.0版本的tar.gz包，即hadoop-2.2.0.tar.gz，放在虚拟机桌面，解压并复制到usr目录下，改名

<pre class="brush: bash; gutter: true">tar -zxf hadoop-2.2.0.tar.gz
cp -rf hadoop-2.2.0 /usr/
mv hadoop-2.2.0 hadoop</pre>

在/etc/profile中添加hadoop的信息

<pre class="brush: bash; gutter: true">vi /etc/profile
export HADOOP_HOME=/usr/hadoop/
export PAHT=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
export HADOOP_LOG_DIR=/usr/hadoop/logs
export YARN_LOG_DIR=$HADOOP_LOG_DIR</pre>

以上便完成了前期安装工作。

四 **Hadoop全分布的配置**
在hadoop中加入java的路径，修改的是hadoop目录下etc/hadoop的hadoop-env.sh

<pre class="brush: bash; gutter: true">vi /usr/hadoop/etc/hadoop/hadoop-env.sh</pre>

将java路径添加在#export java home那里。
修改core-site.xml,添加在configuration里，下同。

<pre class="brush: bash; gutter: true">vi /etc/hadoop/etc/hadoop/core-site.xml</pre>

<pre class="brush: xml; gutter: true">  
&lt;property&gt;  
    &lt;name&gt;fs.default.name&lt;/name&gt;  
        &lt;value&gt;hdfs://master:9000/&lt;/value&gt;  
&lt;/property&gt;  
&lt;property&gt;
        &lt;name&gt;hadoop.tmp.dir&lt;/name&gt;
        &lt;value&gt;/usr/hadoop/tmp&lt;/value&gt;
&lt;/property&gt;  
</pre>

其中，master是主节点的host名fs.default.name与之前的版本名字有所不同。
修改hdfs-site.xml，定义数据节点和名称节点的路径，dfs.replication是datanode的数量，默认为3，小于3的必须写上确切的数目否则会出错。

<pre class="brush: xml; gutter: true"> 
&lt;property&gt;  
        &lt;name&gt;dfs.datanode.data.dir&lt;/name&gt;  
        &lt;value&gt;/usr/hadoop/hdf/data&lt;/value&gt;  
        &lt;final&gt;true&lt;/final&gt;  
&lt;/property&gt;  
     &lt;property&gt;  
        &lt;name&gt;dfs.namenode.name.dir&lt;/name&gt;  
        &lt;value&gt;/usr/hadoop/hdf/name&lt;/value&gt;  
        &lt;final&gt;true&lt;/final&gt;  
   &lt;/property&gt;  
   &lt;property&gt;  
        &lt;name&gt;dfs.replication&lt;/name&gt;  
        &lt;value&gt;2&lt;/value&gt;  
&lt;/property&gt;
&lt;property&gt;  
         &lt;name&gt;dfs.permissions&lt;/name&gt;  
         &lt;value&gt;false&lt;/value&gt;  
&lt;/property&gt;
</pre>

修改mapred-site.xml 定义Mapreduce应用的jobtracker。由于系统并未自带，但提供了模板，因此复制一下

<pre class="brush: xml; gutter: true"> mv mapred-site.xml.template mapred-site.xml</pre>

<pre class="brush: xml; gutter: true"> 
&lt;property&gt;
          &lt;name&gt;mapred.job.tracker&lt;/name&gt;
          &lt;value&gt;master:9001&lt;/value&gt;
&lt;/property&gt;
</pre>

修改yarn-site.xml,配置resourcemanager和nodemanager，这个是2.0以上版本才有的通过yarn管理资源和节点的功能，

<pre class="brush: xml; gutter: true"> 
&lt;property&gt;  
        &lt;name&gt;yarn.nodemanager.aux-services&lt;/name&gt;  
        &lt;value&gt;mapreduce_shuffle&lt;/value&gt;  
&lt;/property&gt;  
&lt;property&gt;  
        &lt;name&gt;yarn.nodemanager.aux-services.mapreduce.shuffle.class&lt;/name&gt;  
        &lt;value&gt;org.apache.hadoop.mapred.ShuffleHandler&lt;/value&gt;  
&lt;/property&gt;  
&lt;property&gt;  
        &lt;name&gt;yarn.resourcemanager.resource-tracker.address&lt;/name&gt;  
        &lt;value&gt;master:8025&lt;/value&gt;  
&lt;/property&gt;  
&lt;property&gt;  
        &lt;name&gt;yarn.resourcemanager.scheduler.address&lt;/name&gt;  
        &lt;value&gt;master:8030&lt;/value&gt;  
&lt;/property&gt;  
&lt;property&gt;  
        &lt;name&gt;yarn.resourcemanager.address&lt;/name&gt;  
        &lt;value&gt;master:8040&lt;/value&gt;  
&lt;/property&gt; 
</pre>

创建并未存在的目录，在hadoop目录下的tmp hdf/data hdf/name logs等
配置完后直接启动，则进行的是单节点操作。若要进行多节点，编辑同目录下的slaves文件，默认自带的是localhost，将其删除，将slave1和slave2一行一个写在文件中。
配置完以上信息就拍一个快照吧！之后就可以进行格式化和启动了，但是在运行自带程序中发现它并没有将任务分配到不同的节点，而是只在master本机的localjobrunner中做，
![](http://ww3.sinaimg.cn/large/68eb7c93jw1ema12m75evj20k00fxdjj.jpg)
因此若需要在不同的节点分配资源，需要再进行一下配置。
修改yarn-env.sh，同样将java的目录配置在export java_home中。
再次修改mapred-site.xml，保留之前的配置，在后面加入如下属性

<pre class="brush: xml; gutter: true">&lt;property&gt;
   &lt;name&gt;mapreduce.framework.name&lt;/name&gt;
   &lt;value&gt;yarn&lt;/value&gt;
&lt;/property&gt;
&lt;property&gt;
   &lt;name&gt;mapreduce.jobhistory.address&lt;/name&gt;
   &lt;value&gt;master:10020&lt;/value&gt;
&lt;/property&gt;
&lt;property&gt;
   &lt;name&gt;mapreduce.jobhistory.webapp.address&lt;/name&gt;
   &lt;value&gt;master:19888&lt;/value&gt;
&lt;/property&gt;
</pre>

实际上就是利用yarn来管理和配置资源，如果配置了yarn，则之前的jobtracker就不起作用。最后修改yarn-site.xml

<pre class="brush: xml; gutter: true">&lt;property&gt;
    &lt;name&gt;yarn.resourcemanager.address&lt;/name&gt;
    &lt;value&gt;master:8032&lt;/value&gt;
&lt;/property&gt;
&lt;property&gt;
    &lt;name&gt;yarn.resourcemanager.scheduler.address&lt;/name&gt;
    &lt;value&gt;master:8030&lt;/value&gt;
&lt;/property&gt;
&lt;property&gt;
    &lt;name&gt;yarn.resourcemanager.webapp.address&lt;/name&gt;
    &lt;value&gt;master:8088&lt;/value&gt;
&lt;/property&gt;
&lt;property&gt;
    &lt;name&gt;yarn.resourcemanager.resource-tracker.address&lt;/name&gt;
    &lt;value&gt;master:8031&lt;/value&gt;
&lt;/property&gt;
&lt;property&gt;
    &lt;name&gt;yarn.resourcemanager.admin.address&lt;/name&gt;
    &lt;value&gt;master:8033&lt;/value&gt;
&lt;/property&gt;
&lt;property&gt;
    &lt;nameyarn.nodemanager.address&lt;/name&gt;
    &lt;value&gt;master:10000&lt;/value&gt;
&lt;/property&gt;
&lt;property&gt;
    &lt;name&gt;yarn.nodemanager.aux-services&lt;/name&gt;
    &lt;value&gt;mapreduce_shuffle&lt;/value&gt;
&lt;/property&gt;
&lt;property&gt;
    &lt;name&gt;yarn.nodemanager.aux-services.mapreduce.shuffle.class&lt;/name&gt;
    &lt;value&gt;org.apache.hadoop.mapred.ShuffleHandler&lt;/value&gt;
&lt;/property&gt;  
</pre>

好了基本的配置完毕了。。。再照一次快照吧！进行格式化（切记不要进行多次格式化！若出现X node are exclued 。。的错误，把tmp 和hdf目录全删了再重建，再进行格式化）

<pre class="brush: xml; gutter: true">cd /usr/hadoop/bin
./hdfs namenode -format
</pre>

随后启动namenode和yarn

<pre class="brush: xml; gutter: true">cd /usr/hadoop/
./start-hdf.sh
./start-yarn.sh
</pre>

若是没有报错就基本成功了！键入jps查看服务。如果没有出现nodemanager就需要手动启动

<pre class="brush: xml; gutter: true">cd /usr/hadoop/sbin/
./yarn-daemon.sh start nodemanager
</pre>

查看到的jps如下

<pre class="brush: xml; gutter: true">Jps
NameNode
ResourceManager
SecondaryNameNode  
NodeManager
</pre>

这就表示启动成功了，可以查看节点状况

<pre class="brush: xml; gutter: true">cd /usr/hadoop/bin
./hdfs dfsadmin -report  
</pre>

也可以通过[http://master:50070](http://master:50070) 查看节点状况。
![](http://ww1.sinaimg.cn/large/68eb7c93jw1ema1yjwdfjj20va05wdh9.jpg)
这样hadoop就配置好全分布了。

五 **运行自带测试程序**

*   wordcount

这是一个检测目标文本中所含词数的自带程序。可在[http://www.gutenberg.org/files/20417/20417.txt](http://www.gutenberg.org/files/20417/20417.txt)下载一篇英文文章，或者自己选择其他的文章。命名为aaa.txt 并放在虚拟机/etc/hadoop目录下。键入

<pre class="brush: xml; gutter: true">
cd /usr/hadoop
bin/hdfs dfs -mkdir /tmp
bin/hdfs dfs -copyFromLocal /usr/hadoop/aaa.txt /tmp
bin/hdfs dfs -ls /tmp
bin/hadoop jar ./share/hadoop/mapreduce/hadoop-mapreduce-examples-2.2.0.jar wordcount /tmp/ /tmp-output 
</pre>

tmp是建立hdfs的目录空间，在tmp-output中输出结果。运行过程中可以看到进程是由mapreduce.job完成。如图为任务开始时
![](http://ww3.sinaimg.cn/large/68eb7c93jw1ema1sqro0nj20gi0bgtbl.jpg)同时可以登陆[http://master:8088](http://master:8088)观看正在运行的application的信息。如图是结束时候的网页信息。![](http://ww1.sinaimg.cn/large/68eb7c93jw1ema1wia60tj20qv07jjsp.jpg)如图是结束时候的命令行信息。![](http://ww2.sinaimg.cn/large/68eb7c93jw1ema1zogahbj20bc0azmye.jpg)执行完毕后可以查看刚才job的执行结果。

<pre class="brush: xml; gutter: true">
cd /usr/hadoop
bin/hdfs dfs -ls /tmp-output
hdfs dfs -cat /tmp-output-00000
</pre>

第二行的目录是输出目录，列出所有的输出信息后，找到名字头相同，后面添加了-0000等数字的就是输出结果。
![](http://ww3.sinaimg.cn/large/68eb7c93jw1ema29bkuf1j20hk0cqdgm.jpg)

*   randomwriter

一葫芦画瓢，操作基本相同，randomwriter的任务是随机生成数到dfs中，每个map输入单个文件名，再随机写到键值对中，基本上是一个性能测试。

<pre class="brush: xml; gutter: true">
cd  /usr/hadoop
bin/hdfs dfs -mkdir /rdw
bin/hadoop jar ./share/hadoop/mapreduce/hadoop-mapreduce-examples-2.2.0.jar randomwriter rdw
</pre>

时间跨度比较久，机子较旧可能会非常慢。参考各类博客，版权不究。