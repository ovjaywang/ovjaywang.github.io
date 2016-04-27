---
title: 'Android为毛耗电[2]'
categories:
  - App强推
  - 软文
tags:
  - Android
id: 899
date: 2016-01-01 23:32:08
---

**<font size="4" face="华文中宋">本文只讨论安卓手机软节电，不负责推荐底包、rom、调频调压方案，刷机root<font color="#ff0000">后果自负</font>。</font>**</p>

<font size="4" face="华文中宋">本小节只讨论干货。在root及xposed框架下。同时兼顾不能肉身翻墙又想用google服务的小狗们。本人的使用情况是正常功能照开，google需要gmail和google photo及部分gcm，黑屏开启手势和解锁，正常收微信、闹钟、短信电话外熄屏一切不开，后台常开应用有snap锁屏、grenntify绿色守护、调音的audio mix、天气插件、BBS、输入法、Nova Launcher。测试的环境为Android L 5.1.1</font>

<font size="4" face="华文中宋">&nbsp;

* * *

<font color="#ff0000">App干货推荐</font></font>

&nbsp;[http://coolapk.com/apk/com.oasisfeng.greenify](http://coolapk.com/apk/com.oasisfeng.greenify) 绿色守护（评论好多捐赠包）国内优秀开发者--唤醒对其 深度休眠hibernateservice 

[http://coolapk.com/apk/com.asksven.betterbatterystats](http://coolapk.com/apk/com.asksven.betterbatterystats)BetterBatteryStats（捐赠包只多了小插件和知识库） 国外应用（使用稳定后可删，留着也不费电）--查看耗电最多的程序 资源 频率占比 [http://coolapk.com/apk/com.uzumapps.wakelockdetector](http://coolapk.com/apk/com.uzumapps.wakelockdetector) wakelock detector（使用后可删，被注册为系统程序，留着不太费电）--查看被CPU 屏幕 时钟唤醒的程序以便将其控制 可以便利的<font color="#ff0000">结合绿色守护使用</font>

[http://coolapk.com/apk/ccc71.at](http://coolapk.com/apk/ccc71.at) 安卓调谐器:3C Toolbox（Android Tuner）--系统调节<font color="#ff0000">神器</font> 控制自启 控制软件权限 自大组件控制 cpu调节 自动内存调节

一张图解释大部分唤醒、杀进程软件的思路

![](http://ww2.sinaimg.cn/large/68eb7c93gw1ezs65djw3nj20tx0pn0xp.jpg)

————————喂下面几个是一类 看清楚需求———————————

[http://coolapk.com/apk/com.linangran.nowakelock](http://coolapk.com/apk/com.linangran.nowakelock) 禁止唤醒1.2.1 （国内开发者）--只能禁5个 暂未发现捐献包 主程序调用捐助包需要与play验证。但也不贵才十几块。 但是禁止效果良好 百度 阿里 系的随便禁一两个就发现效果超级棒 就是挺贵 

[http://coolapk.com/apk/me.piebridge.forcestopgb](http://coolapk.com/apk/me.piebridge.forcestopgb) 阻止运行 国内开发者，免费!--阻止后台、前台、服务及可见的各种组件 不限数目 限制谷歌服务效果良好 

[http://coolapk.com/apk/com.ryansteckler.nlpunbounce](http://coolapk.com/apk/com.ryansteckler.nlpunbounce) 唤醒控制amplify3.3.4--可以查看唤醒次数&nbsp;&nbsp;&nbsp; 

&nbsp;

* * *

<font color="#ff0000">电量消耗结构总结</font>

对于电量，总有一些是必然会消耗的，一些是可以减少的，而一些是可以避免的。那么，对于不同状态下那些成分占的比例会占比略高呢？

1.正常使用状态下（Screen on）：无容置疑的是屏幕的发亮，因而合适的调低亮度能更大的省电，尤其是主动发光的amoled使用深色主题会更省电。当然，大部分使用状态，需要更多的网络连接，4G&gt;3G&gt;2G&gt;wifi是显而易见的。同时使用不同类型的App也会让CPU频率相应的上升，尤其是大型3D网游和功放影音体验。

正常开机使用，耗电排行一般为：

*   屏幕发亮  <li>高CPU占用的App ，包括某些情况下刷帖、浏览网页等长时间交互行为  <li>手机通信（联网、移动通话、视频通话）  <li>Android系统  <li>手机待机 ，即Android OS 之类

2.正常待机状态下（Screen off）包括唤醒和睡眠、深睡眠。当然，大部分人都希望手机进入Idle正常无唤醒的状态，达到hibernate。但国产App可不会这么干。对于待机来说两大耗电来源就是alarm计时器唤醒各类监听进行后台操作、偷跑流量、不必要的消息推送；wakelock把持唤醒锁让手机无法进入深睡眠从而维持后台服务。

正常待机状况下，不飞行，不关闭数据网络，关闭GPS、wifi、蓝牙等，耗电排行是：

*   手机通信 Cell Standy之类。这一块比较大，依赖数据网络的制式  <li>Android OS（即BBS的Kernel wakelock或partial wakelock标记为Android的）  <li>手机待机 （PowerManagerService 及Device Idle之类的）处理休眠状态下设备监听及唤醒锁的管理

&nbsp;

* * *

开启wld和bbs的实时监控，把该装的常用软件装上<font color="#ff0000">先用着那么一段时间。</font>

1.

①利用这段时间 把权限都给干了 装软件给你显示了一list的权限然并卵 不同意没法装 装了也没几个需要的&nbsp; 在【应用程序权限】的设置框中把权限悉心的扫个遍 曾经启用过的权限会有提示何时使用过 例如我把美团外卖的权限全禁了 然而还是可以正常使用

目标是：给app只留必要的权限 像读取短信直接get验证码那些权限就不必了，重点关注权限里的【<font color="#ff0000">保持唤醒</font>】。这个权限意味着能在熄屏时候对手机进行唤醒操作，在亮屏时能够保证数据一致性。没有必要进行推送和同步的应用就应该关闭。

②利用小汽车autostat或者3c toolbox自带的自启动控制器 控制不需要自行启动的项目 除了launcher greentify 3c tool box类似物 等系统框架 其余全都不需要 不需要不需要

目标是：启动速度棒 未启动的应用不会推送任何消息 （微信 微博 GMail等可以排除）

③给app一点微弱的信任，试着在app的设置中，关掉消息推送、wifi下自动更新、wifi下同步等设置，稍微有点技术含量的程序猿都会在设置取消时cut掉后台进程。

④对于各种传感器，依据需求关闭能很好的节约电量（这一块和rom的优化很大关系）例如黑屏手势、人脸解锁、黑屏解锁、口袋检测等。

⑤利用系统的电量统计和后台开启程序大致了解本机运行的大消耗后台。例如本机待机时系统消耗的比例想当高。同时保持着很高的唤醒状态。。

**<font color="#ff0000">系统的电量统计</font>**

![](http://ww2.sinaimg.cn/large/68eb7c93gw1ezrv30mvwcj20u01hc435.jpg)![](http://ww1.sinaimg.cn/large/68eb7c93gw1ezrv0fux6ej20u01hc0x7.jpg)

点开正在运行的后台，可以看本机Android系统和Android操作系统电量消耗高的原因是高通CPU的定位服务。经过调校后的曲线，可以看到中度使用后，待机超过了一个天。同时待机时休眠比例也超过了8成。

![](http://ooo.0o0.ooo/2016/01/10/5693136395d1a.png)![](http://ooo.0o0.ooo/2016/01/10/569313c0b6950.png)

2.使用过后 开启betterbatterystats和wakelock detector

D1 2016/1/6

①<font color="#ff0000">**betterbatterystats BBS**</font>查看耗电大户 一般来说googleservice由于不能随时翻 会造成gmail keep backup contacts 都会发送心跳包不停的连接 此外 除了显而易见的三大流氓滥用唤醒锁 相互唤醒 外 还有奇形怪状的耗电大户 如location定位 微信 

官方帮助指南：[http://better.asksven.org/bbs-how-to/](http://better.asksven.org/bbs-how-to/)

应用beta版及正式版更新日志 [http://forum.xda-developers.com/showthread.php?t=1179809](http://forum.xda-developers.com/showthread.php?t=1179809 "http://forum.xda-developers.com/showthread.php?t=1179809")&nbsp; 包含apk下载。捐助包多了知识库和小控件。

它承诺不在后台记录电量，只在开机、断开充电器、插上充电器、亮屏、熄屏。custom是自定义的操作记录，current是当前时间截止的电量消耗的样本。它提供了后台电量实时监控，这会消耗一定的电量；同时提供了进程监控，并对唤醒次数超过阈值的进行提示。<font color="#ff0000">右侧的圈圈一律代表该应用、该服务、该定时器在所选时间段的使用时间比例</font>

Entries：

![](http://ww4.sinaimg.cn/large/68eb7c93gw1ezp07m935cj20u012saci.jpg)

&nbsp;

<font color="#ff0000">other</font>——睡眠、唤醒、亮屏等其他重要时间节点的电量消耗数据【深睡眠、熄屏唤醒、亮屏、通话时间、wifi开启时长、wifi连上时长、蓝牙开启时长】

*   一般选择unplugged-current 即断电后到现在 一般熄屏情况下 要么deep sleep要么awake，所以两者相加应该是100%。  <li>（Awake）Screen off显示了熄屏时被唤醒所占比例 可以很直观的感受手机离开操作后后台运作的程度 最优的情况当然是0%，手机处于最节能的状态（之前的版本将awake&amp;screen off分开计时 需要两个数据一起对比）  <li>此外，还有wifi 启动、wifi运行与唤醒的关系，很明显，在晚上断开wifi会给手机更好的进入休眠

<font color="#ff0000">kernel wakelocks</font>——使手机唤醒的内核操作【大量的短暂唤醒会严重影响唤醒时间导致手机消耗更多的时间返回睡眠】

*   重点关注项目-PowerManagerService wakelock和multipdp、svnet-dormancy wakelock。前者是激活大部分app的唤醒锁服务，如果它占第一位，则取partial wakelocks关注唤醒时间较多的项目；后者是网络使用量，若存在后台偷跑流量的软件，可以很明显的发现异常，到network数据栏中看一下。  <li>kernel wakelock中的alarm是所有定时器的总和 基本不耗电 但如有异常 可到alarms或amplify中查看详细的计时器项目。  <li>*sync* *backup* *job*很明显是系统的操作 同步 备份 消息推送 如果不需要就禁了吧

<font color="#ff0000">partial wakelocks</font>——是大部分app把持的唤醒锁。（PowerManagerService高的可以关注这项）

*   根据使用频率来限制大耗电partial wakelock 如ig tw fb wb wx等。可以在后文自行设置后台计时器时间。  <li>卸载/冻结/不可用 这些做法有些极端 但实在忍无何忍下还是放弃这个app吧。钛备份下的冻结就比较好用 对google系的后文提到。  <li>主动手动开关GPS。使用时才打开。  <li>根据需要使用后退键退出还是home键退出  <li>wifi是个奇葩 晚上记得关（If you don't need Wifi turn it off: in some cases Wifi is known to cause wakeups and an overhead in e.g. location services）  <li>一次不要改太多，一次一次改 能对手机每个app和每个设置有直观的体验和感受

<font color="#ff0000">alarms</font>——由于应用及服务的定时器唤醒手机【apps设定的触发器在某个时间节点唤醒手机，某些alarms只在唤醒时执行、另一些则直接唤醒手机。通常一个alarm绑定一个intent】

*   长时间和过高次数的alarm都是不正常的。一定要及时纠出。例如微信这狗。  <li>更多的alarm应该在手机亮屏时启动 除了闹钟、微信等必要的消息之外 如知乎提醒、百度外卖提醒等应用，按照使用习惯，更应该是熄屏时完全关闭计时器。

<font color="#ff0000">network</font>——app的网络使用率【区分3G和wifi的网络连接数据】

*   相当多的工具都能查看网络传输的状态。系统自带的也不错。  <li>信号较弱的wifi或窝蜂数据会频繁的连接并唤醒手机

<font color="#ff0000">CPU states</font>——不同CPU频率所占用的时间【包括深度睡眠时的CPU频率占用时间】

<font color="#ff0000">Processes</font>——用户及系统进程消耗的CPU时间【进程CPU消耗，分为用户及系统，以不同颜色标识】

进阶功能：

【<font color="#ff0000">Watchdog</font>】——熄屏分析功能，可自定义亮屏、解锁等。对唤醒超过阈值的app和service进行提示，可在熄屏后一定时间对watchdog关闭。

【<font color="#ff0000">Active Monitoring</font>】——后台运行的一定时间间隔的数据采集，可能会造成额外耗电。

观察一下本机的耗电比例，首先看到other中，Awake(Screen Off)的比例相当高，在其他人的截图是有deep sleep的！！！我安的软件已经炸裂了。尽管开启了wifi会消耗更多的电量，但大量的唤醒让手机不能正常进入休眠。

![](http://ww1.sinaimg.cn/large/68eb7c93gw1ezrv6kd1wgj20u01hcaep.jpg)

再观察partial wakelock ，除了gmail可能不顺利的同步导致消耗略多外，其余app都还算正常的耗电（注意关注连接时间，最多的也才3m35s，这在8h14m11s里来说是相当小的比例，同时消耗的电量也只有0.7%，还可以接受。当然是提前锁过了一部分app的后台）

![](http://ww3.sinaimg.cn/large/68eb7c93gw1ezrv97d35sj20u01hc7ci.jpg)

注意力关注到kernel wakelock，可以看到，核心的耗电量还是比较大的（由于之前的耗电太过大已经关闭了锁屏手势和黑屏解锁）。可以明显的看到，wlan联网在熄屏时间一直连接，qpnp_fg_memaccess qpnp_fg_update_temp qpnp_fg_update_sram是内核的操作。和rom有关。![](http://ww3.sinaimg.cn/large/68eb7c93gw1ezrvef5tykj20u01hctff.jpg)

最后发现。。其实是rom的锅。我的other居然没有出现deep sleep。这是相当可怕的事情。经过调优和整理唤醒后，得到如下相当爽快的数据。接近70%的深睡眠还可以让人接受。毕竟还开启了微信。微博。邮件的消息推送。

![](http://ww4.sinaimg.cn/large/006fVPCvjw1ezvcpamb0bj30u01hcgrd.jpg)

![](http://ww3.sinaimg.cn/large/006fVPCvjw1ezvconkjs5j30u01hctfb.jpg)![](http://ww2.sinaimg.cn/large/006fVPCvjw1ezvcqv63zoj30u01hcah1.jpg)

②<font color="#ff0000">**wakelock detector WLD**</font>查看流氓唤醒大户 这里要提到微信。千万不要手欠在应用内使用它的升级。最好在play或者cool市场升级。官方提供的流氓唤醒一个晚上唤醒好几千次。从未进入过休眠状态。一进入唤醒状态又有很多应用跟着启动。 微信的消息是利用同步，而CPU唤醒锁则是被滥用了的。

Test1 2016.01.06

![](http://ww4.sinaimg.cn/large/68eb7c93gw1ezp096st0oj20u01hcwlt.jpg)![](http://ww4.sinaimg.cn/large/68eb7c93gw1ezp0akfqdzj20u01hcgrl.jpg)![](http://ww4.sinaimg.cn/large/68eb7c93gw1ezp09t72gcj20u01hcwja.jpg)![](http://ww3.sinaimg.cn/large/68eb7c93gw1ezp0a61g8yj20u01hcgt8.jpg)

上图分别是WLD四个视图-CPU唤醒锁时间、Kernel唤醒锁时间、屏幕唤醒锁触发次数、Alarm触发器次数.

CPU锁实际上就是用户app的持有wakelock的时间。可以看到由于需要同步相册里的图片，本次测试中google photo占用了极大部分的唤醒锁时间，其余的应用则比较正常；内核持有wakelock一般与rom的品质有关，由于po主使用的是第三方精简rom，出现了一些奇特的bug，例如第一项qcom_rx_wakelock高通的cpu在5.1的bug，在熄屏状态下仍在进行不停的wifi连接，持有唤醒锁对应用进行超负荷的运算，这也是比较不容易控制也难以改善的方面；屏幕唤醒则和用户使用的点亮次数有关，可以看到，每次启动SU超级root、微信、QQ、BBS都进行了短暂的唤醒，而视频软件如果在播放中熄屏，亮屏后也会唤醒屏幕锁（持久保有）；最后一项则是需要重点关注的，定时器启动唤醒锁，2个多小时内，google服务、微信、微博、日历、输入法都进行了大量的唤醒，导致80%的时间都难以进入deep sleep。

Test2 2016.01.08

CPU唤醒锁：本次测试google相册已经不进行同步，更多的是Gmail的同步操作。由于输入了7个邮件，不可避免的会进行更多频次的操作。除了gmail能利用谷歌框架同步外，其他邮件都应该设置比较合理的同步时间。而微信（后文提及）设置了封锁CPU锁后同步的频次也有所减少。

kernel wakelock：qcom_rx_wakelock已经减少了很多，这是关闭黑屏手势的减少唤醒次数的效果。两次测试中，PowerManagerService.WakeLocks、qcom_rx_wakelock、PowerManagerService.Broadcasts、NETLINK四个核心服务占据的时间远远超过其他的服务。这也是需要优化的部分。

屏幕唤醒锁触发：

Alarm计时器触发：可以看到QQ、微信在夜晚的唤醒次数有些多，其实在qq、微信中都可以设置免打扰时间段，主动的消除在夜间待机时的计时器唤醒推送。

![](http://ww4.sinaimg.cn/large/68eb7c93gw1ezrvmr0wr2j20u01hcqaa.jpg)![](http://ww3.sinaimg.cn/large/68eb7c93gw1ezrvnilpewj20u01hc7ai.jpg)

![](http://ww1.sinaimg.cn/large/68eb7c93gw1ezrvjeovxej20u01hcahy.jpg)![](http://ww3.sinaimg.cn/large/68eb7c93gw1ezrvlj75wdj20u01hcn5a.jpg)

&nbsp;

4.绿色守护、3C ToolBox、Amplify的使用。

①.<font color="#ff0000">**绿色守护**</font>

<font color="#ff0000">思路</font>：熄屏清掉所有被勾选的app的服务和缓存进程，同时阻止所有可唤醒包含服务的监听receiver&amp;释放所有包含服务的相互唤醒wakelock，达到相对休眠状态。主要运作在熄屏后不希望后台运作的程序（<font color="#ff0000">无法在前台操作切断应用间唤醒</font>）。捐助版的亮点是可以在休眠状态下使用GCM服务。

绿色守护在xda的讨论贴 开发者oisisfeng也在。

[http://forum.xda-developers.com/showthread.php?t=2155737](http://forum.xda-developers.com/showthread.php?t=2155737 "http://forum.xda-developers.com/showthread.php?t=2155737")

绿守作者在知乎的问答

[https://www.zhihu.com/question/38311793/answer/75897889](https://www.zhihu.com/question/38311793/answer/75897889 "https://www.zhihu.com/question/38311793/answer/75897889")

由于测试机在5.1.1,还未升级到Android M，因此没有享受到doze mode和app standby带来的福利。

一张图看到就算不是 百度阿里疼腾讯系 从某个市场出来的app也会相互唤醒。这锅到底谁来背。以后反编译源码看一看。即刻-豆瓣-虎皮体育-同花顺-航旅纵横-什么值得买是什么关系。更不用说百度及包含百度字样的app-爱奇艺-去哪儿-uber-乐视-91-多米-音悦台-穷游途牛-汽车之家&nbsp; 腾讯及包含qq字样-京东-大众-美团-饿了么-58-猎豹-搜狗-同城 阿里及UC-微博-优酷土豆-陌陌-滴滴快的-小米魅族

![](http://ww2.sinaimg.cn/large/68eb7c93gw1ezs6frg1q4j20u01hc461.jpg)

使用方法：主界面显示正在运行的服务及已经绿色化的应用。点击上方【+】号对所有潜在威胁进行评估，勾选的应用会在熄屏数分钟后清空缓存、杀死服务、抑制app唤醒及alarm的唤醒。root及xposed模式下效果更佳。支持gcm的应用会有小图标，能在绿色化的情况下还可收发消息。

②.<font color="#ff0000">**3C tool box**</font>

<font color="#ff0000">思路</font>：封杀选定的activity receiver service。结晶化应用（无论前台后台，需要更多的测试，容易出现应用fc 可以实现前台界面退出后直接清缓存 <font color="#ff0000">可直接在应用使用中禁止广告、介绍、推荐app的activity和service</font>）（本篇不涉及3c的调频调压 内存管理设置）

例如：杀错了谷歌的邮件提醒，即便能实时接收邮件，但系统不做出提醒通知。

![](http://ww4.sinaimg.cn/large/68eb7c93gw1ezrulsjoalj20u01hcgt5.jpg)

又例如：杀错了QQi的消息消息提示核心，虽然在后台能看到QQ的进程，但新消息通知会出现不同步（极有可能是阻止了开机自启杀掉的，但这个receiver需要的权限相当多，网络改变、时间改变等乱七八糟的）

![](http://ww1.sinaimg.cn/large/68eb7c93gw1ezrusb4h51j20u01hctf7.jpg)

再比如杀掉了随手记的通知权限，居然能把自己的同步服务给搞挂了。。这代码写的。此外几个应用，饿了么，大众点评，都有不同程度的易fc但后台又唤醒个不停的问题。

使用方法：对四大组件中那三个进行轮番排查，限制自启动，限制唤醒路径，冻结后台。（主要操作在应用管理里）

③**<font color="#ff0000">唤醒控制Amplify</font>**的使用

思路：对Alarm计时器的时间进行限制重设、对Service进行屏蔽、限制同步，可使用进阶的正则表达式过滤。

使用方法：对在wld找到的唤醒次数多的详细分析，对alarm中唤醒频次高的限制更长的时间，对显见的服务停用，先做完这两者再对wakelock进行限制。

![](http://ooo.0o0.ooo/2016/01/10/569322dfc0422.png)

④**<font color="#ff0000">禁止唤醒</font>**的使用(非捐助版只能对5个app进行限制，入正也就十几块)

思路：在前台就主动把各类唤醒锁进行封闭、定时器对齐、对同步进行限制

作者曾提到了<font color="#ff0000">唤醒锁的危害</font><font color="#000000">及禁止唤醒对这些危害的防范</font>

**CPU唤醒锁**: 这是在息屏后阻止你的设备进入休眠状态的唤醒锁, 禁用它通常不会有任何问题.

**所有其它唤醒锁**: 除了CPU唤醒锁之外, 还有一些唤醒锁可以阻止设备休眠, 甚至阻止设备息屏. 开启这个选项以禁用这些唤醒锁.

**同步**: 同步也可以唤醒设备, 如果你不需要应用的同步功能, 使用这个选项来禁用掉它.

**对齐定时器:** (&gt;= Android 4.4) AlarmManager可以使用定时器来周期性的唤醒设备, 阻止CPU进入长期休眠状态. 启用这个选项来强制对齐定时器, 让它们尽量在同一时间触发以节省电量. 请注意: 对于设计不良的应用, 启用此选项有可能会引发推送消息延迟.

[https://www.linangran.com/?p=611](https://www.linangran.com/?p=611) 作者使用指南

![](http://ooo.0o0.ooo/2016/01/10/569323251d9ce.png)

⑤阻止运行的使用

[https://github.com/liudongmiao/ForceStopGB](https://github.com/liudongmiao/ForceStopGB "https://github.com/liudongmiao/ForceStopGB") 开源项目地址 免费

相当推荐的一款软件，无论前后台，通过劫持系统api，直接禁止非需求service的启动。精简版的3c tool box，保证应用只在需要时启动。添加到阻止列表的应用只在以下情况开启：

*   启动器直接或第三方provider，如手动点button、分享、支付  <li>桌面小部件定时更新、但只维持30秒（这个好顶赞）  <li>同步开启时的定时同步，也只能维持30秒  <li>除谷歌服务外的系统服务、支付宝的支付服务  <li>其他可能的用户行为引起的启动

同时，谷歌服务在阻止列表时，可以支持gcm和谷歌家族应用的使用。但当任何一个谷歌家族应用没有退出时，都不会退出谷歌服务。

![](http://ooo.0o0.ooo/2016/01/10/569322a0e590e.png)

5.针对微信的方案

微信最近从良而加入gcm的推送方案，能在有谷歌框架的手机在hibernate休眠状况下直接推送消息！既不用随时拿着唤醒锁又不唤醒其他乱七八糟的应用！但原先保留的wakelock模式依旧存在，握着各种随机数的锁禁都不知道禁哪些。但大神出现了。 

[https://www.zhihu.com/question/31136645](https://www.zhihu.com/question/31136645) 知乎高票的正则表达式杀wakerlock方案

[http://bbs.gfan.com/android-7963258-1-1.html](http://bbs.gfan.com/android-7963258-1-1.html) gfan精华帖

上面是一种方案，但微信更改了wakelock的命名方式就会得不偿失错杀唤醒锁了。最近发现单独使用【 禁止唤醒】中的CPU唤醒锁，也能阻止微信胡乱的唤醒，同时不影响推送；但相同的情况下使用绿色守护对微信绿色化却不能及时收到消息。

6.针对gms的方案 考虑到gms 的复杂性 提一下它涉及到google账户的同步*sync* 备份 keep photo gmail(分为对被墙的不被墙的) 三个文档&nbsp; play软件升级等等 google service包含了N个心跳包的alarm 各种硬件CPU唤醒锁

①完全禁掉是一个办法（当然这需要框架 service play一起禁 不然各种报错）这种方式最省电 但是又想使用google收国内账户 或者时不时FQ看看FB TW IG使用其中一些

②不停更换host文件。这个方式能保证随时上墙又不会因为vpn挂着长期费电。测试几天发现耗电稳定，gmail是不是抽风但效果拔群。（推荐神站load.cn）

③长时间挂载稳定vpn。该方式同样能随时保证上墙接收同步和推送。同时配合now能享受到肉身翻墙的欢喜效果。但vpn的开发良莠不齐，耗电状况不一。

④不需要google家族的直接装无gms版的精简rom或国内大部分融都会删掉——————这个方法纯天然摒除了谷歌的抽风状况，但，滚粗。

⑤久不久上墙。gms耗电的问题就是连接不上google而大量地进行重连接。所以希望使用google服务又不想随时耗电使用vpn的一个便捷方案就是每天保证登录一次，并在amplify对google框架进行限制时间，每隔一天敲醒alarm一次，但不能阻止cpu唤醒锁对gms想要获取推送的影响。

参考：

【2013.04.14】绿守作者 微信收费事件背后被广泛忽略的技术细节

[http://blog.oasisfeng.com/2013/04/14/dirty-secret-behind-weixin-charge-gate/](http://blog.oasisfeng.com/2013/04/14/dirty-secret-behind-weixin-charge-gate/ "http://blog.oasisfeng.com/2013/04/14/dirty-secret-behind-weixin-charge-gate/")

【2013.06.19】PC Online 的省电优化方案介绍

[http://pcedu.pconline.com.cn/334/3345895_all.html](http://pcedu.pconline.com.cn/334/3345895_all.html "http://pcedu.pconline.com.cn/334/3345895_all.html")

【2014.02.16】唤醒锁: 检测 Android* 应用中的 No-Sleep（无法进入睡眠）问题

[https://software.intel.com/zh-cn/android/articles/wakelocks-detect-no-sleep-issues-in-android-applications?language=es](https://software.intel.com/zh-cn/android/articles/wakelocks-detect-no-sleep-issues-in-android-applications?language=es "https://software.intel.com/zh-cn/android/articles/wakelocks-detect-no-sleep-issues-in-android-applications?language=es")

【2014.06.05】wakelock alarm wifi 详细数据测评 唤醒锁: 检测 Android* 应用中的 No-Sleep（无法进入睡眠）问题[http://www.oneplusbbs.com/forum.php?mod=viewthread&amp;tid=366390](http://www.oneplusbbs.com/forum.php?mod=viewthread&amp;tid=366390)

【2014.07.22】Android Standby 对于绿色守护唤醒对其 进入深睡眠的一些思考

[http://lockekk.github.io/jekyll-bootstrap/2014/07/22/Android-Standby/](http://lockekk.github.io/jekyll-bootstrap/2014/07/22/Android-Standby/ "http://lockekk.github.io/jekyll-bootstrap/2014/07/22/Android-Standby/")

&nbsp;

android论坛关于ba保持屏幕唤醒和cpu唤醒

[http://developer.android.com/training/scheduling/wakelock.html](http://developer.android.com/training/scheduling/wakelock.html "http://developer.android.com/training/scheduling/wakelock.html")

此外 几个app的酷市场下评论都很精彩。

例如Amplify的评论 [http://coolapk.com/feed/1201169](http://coolapk.com/feed/1201169 "http://coolapk.com/feed/1201169")

例如Google Play Service 的评论 [http://coolapk.com/feed/1196656](http://coolapk.com/feed/1196656 "http://coolapk.com/feed/1196656")