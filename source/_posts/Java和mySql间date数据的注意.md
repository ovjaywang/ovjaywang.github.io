---
title: Java和mySql间date数据的注意
categories:
  - 代码狗
  - 学习log
tags:
  - java
  - mysql
id: 185
date: 2013-05-16 21:00:26
---

已经吃了很多次亏了...每每其他调试得得意满满的时候就出来捣乱了
首先是在java中 date和string类型之间的转换就容易出错
直接使用toString默认输出的是
/** Date类的格式: Sat Apr 16 13:17:29 CST 2006 */
data-&gt;string
<pre class="brush: java; gutter: true; first-line: 1; highlight: []; html-script: false">public static String DatetoString(Date dt){
		DateFormat df=new SimpleDateFormat(&quot;yyyy/MM/dd HH:mm:ss&quot;);
		String myD=df.format(dt);
		return myD;
	}</pre>
string-&gt;date
<pre class="brush: java; gutter: true; first-line: 1; highlight: []; html-script: false">public static Date Trans(String str)
	{
		DateFormat df=new SimpleDateFormat(&quot;yyyy/MM/dd HH:mm:ss&quot;);
		Date d=null;
		try {
			d=df.parse(str);
		} catch (ParseException e) {
			e.printStackTrace();
		}
		if(d==null)
		{System.out.println(&quot;格式输入失败&quot;);}
		else
		{
			System.out.println(&quot;格式转换成功  &quot;+d);
		}
		return d;
	}</pre>
其中，yyyyMMddHHmmss表示年月日时分秒不可以随便代替 其中可以加入: / - 空格等一切你想塞入的符号
这是月份的表示 其中月份是从0开始的
<pre class="brush: java; gutter: true; first-line: 1; highlight: []; html-script: false">Calendar cal = Calendar.getInstance();  
cal.setTime(new Date());  
System.out.println(&quot;System   Date:   &quot; + cal.get(Calendar.MONTH + 1));</pre>
以下是mysql和java之间传值转化的类
MySql的时间类型有             Java中与之对应的时间类型 
date                        java.sql.Date 
datetime                    java.sql.Timestamp 
timestamp                   java.sql.Timestamp 
time                        java.sql.Time 
year                        java.sql.Date
<pre class="brush: java; gutter: true; first-line: 1; highlight: []; html-script: false">
/获得系统时间  
java.util.Date date = new java.util.Date();  
//将时间格式转换成符合Timestamp要求的格式  
String nowTime = new SimpleDateFormat(&quot;yyyy-MM-dd HH:mm:ss&quot;).format(date);  
java.sql.Timestamp ts_date = java.sql.Timestamp.valueOf(nowTime)  
</pre>
TimeDate->String
<pre class="brush: actionscript3; gutter: true; first-line: 1; highlight: []; html-script: false">
Timestamp ts = new Timestamp(System.currentTimeMillis());  
        String tsStr = &quot;&quot;;  
        DateFormat sdf = new SimpleDateFormat(&quot;yyyy/MM/dd HH:mm:ss&quot;);  
        try {  
            //方法一  
            tsStr = sdf.format(ts);  
            System.out.println(tsStr);  
            //方法二  
            tsStr = ts.toString();  
            System.out.println(tsStr);  
        } catch (Exception e) {  
            e.printStackTrace();  
        }  
</pre>
TimeDate->Date
<pre class="brush: actionscript3; gutter: true; first-line: 1; highlight: []; html-script: false">
Timestamp ts = new Timestamp(System.currentTimeMillis());  
        Date date = new Date();  
        try {  
            date = ts;  
            System.out.println(date);  
        } catch (Exception e) {  
            e.printStackTrace();  
        } 
</pre>