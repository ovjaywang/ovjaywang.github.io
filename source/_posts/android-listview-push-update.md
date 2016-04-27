---
title: Android中listview使用baseAdapter的下拉刷新和加载更多功能
categories:
  - 代码狗
  - 学习log
tags:
  - Android
  - java
id: 225
date: 2013-06-02 04:58:24
---

该功能基于多个成熟的博客和demo的集成，最适合开发的方式推荐。

首先是利用baseAdapter开发listview，baseAdapter拓展性自由性非常高，利用activity加载适配器Adapter再加载资源进入视图view中，图片imageview textview 等都可以自由的组织布局。在一个额外的xml布局中写好单个list项目的布局就可以自由的适配载入整个list 参考[自定义BaseAdapter](http://www.oschina.net/code/snippet_203635_7475) [demo](http://pan.baidu.com/share/link?shareid=464702&uk=4195521600)针对arrayAdapter simpleAdpter等四个适配器的比较。

其次是下拉刷新的功能，虽然网上有很多例子，但是不是继承FrameLayout 就是集成activity。很少有继承listactivity并且还使用baseAdapter的例子。这里主要参考的是[Android中实现下拉刷新](http://blog.csdn.net/berber78/article/details/7387271)以及[Android:ListView使用RelativeLayout与TableLayout比较](http://greatwqs.iteye.com/blog/1058366) 这里的关键并不是头尾的算法和头尾的布局，关键的是如何将listview嵌入另一个布局中并保持布局的稳定和及时性。由于使用的是baseAdapter数据不是直接在activity里添加而是通过继承自baseAdapter的适配器调用getView函数获得的，因而数据的呈现极其容易出现冲突。这里展示两种方法：
其一是针对上边[Android中实现下拉刷新](http://blog.csdn.net/berber78/article/details/7387271)的例子中的函数改写的。原代码是：
<pre class="brush: xml; gutter: true; first-line: 1; highlight: []; html-script: false">
&lt;?xml version=&quot;1.0&quot; encoding=&quot;utf-8&quot;?&gt;
&lt;RelativeLayout
  xmlns:android=&quot;http://schemas.android.com/apk/res/android&quot;
  android:layout_width=&quot;fill_parent&quot;
  android:layout_height=&quot;fill_parent&quot;
  &gt;
  &lt;LinearLayout android:orientation=&quot;vertical&quot;
                android:layout_width=&quot;fill_parent&quot;
                android:layout_height=&quot;fill_parent&quot;
                android:background=&quot;#000000&quot;&gt;       
	    //将ListView改为包名.自定义的列表类名，MsgListView.java代码见后，此代码不需改通用，下拉刷新相关即此处，其他无关
	    &lt;cn.xd.microblogging.tools.MsgListView
	    android:id=&quot;@android:id/list&quot;
	    android:layout_height=&quot;wrap_content&quot;
	    android:layout_width=&quot;fill_parent&quot;
	    android:layout_weight=&quot;1.0&quot;
	    android:drawSelectorOnTop=&quot;false&quot;
	    android:scrollbars=&quot;vertical&quot;
	    android:fadingEdgeLength=&quot;0dip&quot; 
	    android:divider=&quot;@drawable/line&quot;
	    android:dividerHeight=&quot;3.0dip&quot;
	    android:cacheColorHint=&quot;#00000000&quot; 
	    android:paddingBottom=&quot;40dp&quot;/&gt;
  &lt;/LinearLayout&gt;
  &lt;LinearLayout
      android:id=&quot;@+id/msg_list_load&quot;
      android:layout_width=&quot;fill_parent&quot;
      android:layout_height=&quot;fill_parent&quot; &gt;      
    &lt;LinearLayout android:id=&quot;@android:id/empty&quot;
                  android:layout_width=&quot;fill_parent&quot;
                  android:layout_height=&quot;fill_parent&quot; &gt;       
      &lt;include
        android:id=&quot;@android:id/empty&quot;
        layout=&quot;@layout/empty_loading&quot;
        android:layout_width=&quot;fill_parent&quot;
        android:layout_height=&quot;fill_parent&quot; /&gt;      
     &lt;/LinearLayout&gt;
  &lt;/LinearLayout&gt;
&lt;/RelativeLayout&gt;
</pre>
我的原baseAdapter的适配xml是：
<pre class="brush: xml; gutter: true; first-line: 1; highlight: []; html-script: false">
&lt;?xml version=&quot;1.0&quot; encoding=&quot;utf-8&quot;?&gt;
&lt;LinearLayout xmlns:android=&quot;http://schemas.android.com/apk/res/android&quot;
    android:layout_width=&quot;fill_parent&quot;
    android:layout_height=&quot;fill_parent&quot;
    android:orientation=&quot;horizontal&quot; &gt;
    &lt;LinearLayout
        android:layout_width=&quot;wrap_content&quot;
        android:layout_height=&quot;wrap_content&quot;
        android:orientation=&quot;vertical&quot; &gt;
        &lt;ImageView
            android:id=&quot;@+id/img&quot;
            android:layout_width=&quot;100dp&quot;
            android:layout_height=&quot;wrap_content&quot;
            android:layout_margin=&quot;5dp&quot; /&gt;
        &lt;TextView
            android:id=&quot;@+id/msgname&quot;
            android:layout_width=&quot;wrap_content&quot;
            android:layout_height=&quot;wrap_content&quot;
            android:layout_gravity=&quot;center&quot;
            android:layout_marginBottom=&quot;4dp&quot;
            android:textColor=&quot;#FFFFFFFF&quot;
            android:textSize=&quot;12sp&quot; /&gt;          
    &lt;/LinearLayout&gt;
    &lt;LinearLayout
        android:layout_width=&quot;wrap_content&quot;
        android:layout_height=&quot;wrap_content&quot;
        android:orientation=&quot;vertical&quot; &gt;
        &lt;TextView
            android:id=&quot;@+id/msgtext&quot;
            android:layout_width=&quot;wrap_content&quot;
            android:layout_height=&quot;100dp&quot;
            android:textColor=&quot;#FFFFFFFF&quot;
            android:textSize=&quot;22sp&quot; /&gt;
        &lt;LinearLayout
            android:layout_width=&quot;wrap_content&quot;
            android:layout_height=&quot;wrap_content&quot;
            android:orientation=&quot;horizontal&quot;
            android:layout_gravity=&quot;left&quot; &gt;
            &lt;TextView
                android:id=&quot;@+id/topic&quot;
                android:layout_width=&quot;wrap_content&quot;
                android:layout_height=&quot;wrap_content&quot;
                android:layout_gravity=&quot;left&quot;
                android:layout_marginRight=&quot;10dp&quot;
                android:textColor=&quot;#FFFFFFFF&quot;
                android:textSize=&quot;13sp&quot; /&gt;
            &lt;TextView
                android:id=&quot;@+id/msgtime&quot;
                android:layout_width=&quot;wrap_content&quot;
                android:layout_height=&quot;wrap_content&quot;
                android:layout_marginRight=&quot;10dp&quot;
                android:layout_gravity=&quot;left&quot;
                android:textColor=&quot;#FFFFFFFF&quot;
                android:textSize=&quot;13sp&quot; /&gt;
            &lt;TextView
                android:id=&quot;@+id/msgaddr&quot;
                android:layout_width=&quot;wrap_content&quot;
                android:layout_height=&quot;wrap_content&quot;
                android:layout_gravity=&quot;left&quot;
                 android:layout_marginRight=&quot;10dp&quot;
                android:textColor=&quot;#FFFFFFFF&quot;
                android:textSize=&quot;13sp&quot; /&gt;
            &lt;TextView
                android:id=&quot;@+id/comment&quot;
                android:layout_width=&quot;wrap_content&quot;
                android:layout_height=&quot;wrap_content&quot;
                android:layout_gravity=&quot;left&quot;
                android:textColor=&quot;#FFFFFFFF&quot;
                android:textSize=&quot;13sp&quot; /&gt;
        &lt;/LinearLayout&gt;
    &lt;/LinearLayout&gt;
&lt;/LinearLayout&gt;
</pre>
改写后的是：<pre class="brush: xml; gutter: true; first-line: 1; highlight: []; html-script: false">
&lt;?xml version=&quot;1.0&quot; encoding=&quot;utf-8&quot;?&gt;
&lt;RelativeLayout
  xmlns:android=&quot;http://schemas.android.com/apk/res/android&quot;
  android:layout_width=&quot;fill_parent&quot;
  android:layout_height=&quot;fill_parent&quot;
  &gt;
  &lt;LinearLayout android:orientation=&quot;vertical&quot;
                android:layout_width=&quot;fill_parent&quot;
                android:layout_height=&quot;fill_parent&quot;
                android:background=&quot;#000000&quot;&gt;       
	    &lt;com.wjy.message.MsgListView
	    android:id=&quot;@android:id/list&quot;
	    android:layout_height=&quot;wrap_content&quot;
	    android:layout_width=&quot;fill_parent&quot;
	    android:layout_weight=&quot;1.0&quot;
	    android:drawSelectorOnTop=&quot;false&quot;
	    android:scrollbars=&quot;vertical&quot;
	    android:fadingEdgeLength=&quot;0dip&quot; 
	    android:divider=&quot;@drawable/line&quot;
	    android:dividerHeight=&quot;3.0dip&quot;
	    android:cacheColorHint=&quot;#00000000&quot; 
	    android:paddingBottom=&quot;40dp&quot;/&gt;
  &lt;/LinearLayout&gt;
  &lt;LinearLayout 
    android:layout_width=&quot;fill_parent&quot;
    android:layout_height=&quot;fill_parent&quot;
    android:orientation=&quot;horizontal&quot; &gt;
    &lt;LinearLayout
        android:layout_width=&quot;wrap_content&quot;
        android:layout_height=&quot;wrap_content&quot;
        android:orientation=&quot;vertical&quot; 
        android:id=&quot;@+id/hehe&quot;&gt;
        &lt;ImageView
            android:id=&quot;@+id/img&quot;
            android:layout_width=&quot;100dp&quot;
            android:layout_height=&quot;wrap_content&quot;
            android:layout_margin=&quot;5dp&quot; /&gt;
        &lt;TextView
            android:id=&quot;@+id/msgname&quot;
            android:layout_width=&quot;wrap_content&quot;
            android:layout_height=&quot;wrap_content&quot;
            android:layout_gravity=&quot;center&quot;
            android:layout_marginBottom=&quot;4dp&quot;
            android:textColor=&quot;#FFFFFFFF&quot;
            android:textSize=&quot;12sp&quot; /&gt;
    &lt;/LinearLayout&gt;
    &lt;LinearLayout
        android:layout_width=&quot;wrap_content&quot;
        android:layout_height=&quot;wrap_content&quot;
        android:orientation=&quot;vertical&quot; &gt;
        &lt;TextView
            android:id=&quot;@+id/msgtext&quot;
            android:layout_width=&quot;wrap_content&quot;
            android:layout_height=&quot;100dp&quot;
            android:textColor=&quot;#FFFFFFFF&quot;
            android:textSize=&quot;22sp&quot; /&gt;

        &lt;LinearLayout
            android:layout_width=&quot;wrap_content&quot;
            android:layout_height=&quot;wrap_content&quot;
            android:orientation=&quot;horizontal&quot;
            android:layout_gravity=&quot;left&quot; &gt;

            &lt;TextView
                android:id=&quot;@+id/topic&quot;
                android:layout_width=&quot;wrap_content&quot;
                android:layout_height=&quot;wrap_content&quot;
                android:layout_gravity=&quot;left&quot;
                android:layout_marginRight=&quot;10dp&quot;
                android:textColor=&quot;#FFFFFFFF&quot;
                android:textSize=&quot;13sp&quot; /&gt;
            &lt;TextView
                android:id=&quot;@+id/msgtime&quot;
                android:layout_width=&quot;wrap_content&quot;
                android:layout_height=&quot;wrap_content&quot;
                android:layout_marginRight=&quot;10dp&quot;
                android:layout_gravity=&quot;left&quot;
                android:textColor=&quot;#FFFFFFFF&quot;
                android:textSize=&quot;13sp&quot; /&gt;

            &lt;TextView
                android:id=&quot;@+id/msgaddr&quot;
                android:layout_width=&quot;wrap_content&quot;
                android:layout_height=&quot;wrap_content&quot;
                android:layout_gravity=&quot;left&quot;
                 android:layout_marginRight=&quot;10dp&quot;
                android:textColor=&quot;#FFFFFFFF&quot;
                android:textSize=&quot;13sp&quot; /&gt;
            &lt;TextView
                android:id=&quot;@+id/comment&quot;
                android:layout_width=&quot;wrap_content&quot;
                android:layout_height=&quot;wrap_content&quot;
                android:layout_gravity=&quot;left&quot;
                android:textColor=&quot;#FFFFFFFF&quot;
                android:textSize=&quot;13sp&quot; /&gt;
        &lt;/LinearLayout&gt;
    &lt;/LinearLayout&gt;
&lt;/LinearLayout&gt;
&lt;/RelativeLayout&gt;
</pre>
这其中加入了一个ImageView，一些textview的横横竖竖的布局。效果如图，![](http://i1061.photobucket.com/albums/t476/ov_beeshoot/635583B7-2.jpg)
另一种是针对这个demo的[PullToRefreshView](http://pull-to-refresh-view.googlecode.com/svn/trunk/PullToRefreshView)这个托管在googlecode的资源非常完满的解决了很多问题，其中关于样式的xml有两个分别是：
grid_item.xml
<pre class="brush: xml; gutter: true; first-line: 1; highlight: []; html-script: false">
&lt;?xml version=&quot;1.0&quot; encoding=&quot;utf-8&quot;?&gt;
&lt;LinearLayout xmlns:android=&quot;http://schemas.android.com/apk/res/android&quot;
    android:layout_width=&quot;wrap_content&quot;
    android:layout_height=&quot;wrap_content&quot;
    android:orientation=&quot;vertical&quot; &gt;
	&lt;TextView 
	    android:id=&quot;@+id/text&quot;
	    android:drawableTop=&quot;@drawable/ic_launcher&quot;
	    android:layout_width=&quot;wrap_content&quot;
	    android:layout_height=&quot;wrap_content&quot;
	    android:gravity=&quot;center_horizontal&quot;
	    android:text=&quot;test&quot;/&gt;
&lt;/LinearLayout&gt;
</pre>
其中就是放入了一个固定的图片和固定的文字
可以随意替换。另一个是
test_listview.xml
<pre class="brush: xml; gutter: true; first-line: 1; highlight: []; html-script: false">
&lt;?xml version=&quot;1.0&quot; encoding=&quot;utf-8&quot;?&gt;
&lt;com.miloisbadboy.view.PullToRefreshView xmlns:android=&quot;http://schemas.android.com/apk/res/android&quot;
    android:id=&quot;@+id/main_pull_refresh_view&quot;
    android:background=&quot;@android:color/white&quot;
    android:layout_width=&quot;fill_parent&quot;
    android:layout_height=&quot;fill_parent&quot; 
    android:orientation=&quot;vertical&quot;&gt;
	&lt;!-- 这里放置listview,gridview或者scrollview的布局 ,PullToRefreshView
	要设置android:orientation=&quot;vertical&quot;属性
	否则,显示不正确--&gt;
    &lt;ListView
        android:id=&quot;@android:id/list&quot;
        android:cacheColorHint=&quot;#00000000&quot;
        android:layout_width=&quot;fill_parent&quot;
        android:layout_height=&quot;fill_parent&quot; /&gt;

&lt;/com.miloisbadboy.view.PullToRefreshView&gt;
</pre>
这里和上边的一样都调用的私有的一个下拉刷新的头样式的view文件。这里都可以直接用。这个demo几乎不用改动xml，就是在Adapter适配器getView中
convertView = mInflater.inflate(R.layout.你的list样式名字, null);不同罢了。

最后一个问题是下拉刷新中list数据改变的情况。这里不能在额外的thread进行数据的更改，这样会异常。另外在非主线程调用Adapter的notifyDataSetChanged方法也会抱错。这里可以参考[notifyDataSetChanged() 动态更新ListView](http://www.cnblogs.com/wangjianhui/archive/2011/06/15/2081705.html)的例子，两种方法：1利用Handler处理2利用AsyncTask异步处理。这里图鉴第二种，这里加载图片，加载大数据方面AsyncTask非常管用。 可以参考这里[Android中AsyncTask的用法实例](http://www.pin5i.com/showtopic-android-asynctask-sample.html)以及这个例子：[异步下载图片](http://pan.baidu.com/share/link?shareid=464708&uk=4195521600)。
这是最后的效果：
![](http://i1061.photobucket.com/albums/t476/ov_beeshoot/635583B7-3.jpg)
BaseAdapter使用好可以搭配出很多用户体验很好的列表，嵌入别的linerlayout也是很不错的。