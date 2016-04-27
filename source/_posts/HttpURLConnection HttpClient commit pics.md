---
title: 'Android使用HttpURLConnection HttpClient提交图片'
categories:
  - 代码狗
  - 学习log
tags:
  - Android
id: 201
date: 2013-05-21 14:43:00
---

Android进行图片提交大多数用的是HttpURLConnection 但是我的大半天的试验只成功了一次。参考了绝大多数主流的代码，不论是Base64还是Json还是URLEncode能换过的包都换过了，但都是上传成功，上传的效果要么图裂要么只有顶部的几条扫描进去了。
由于是服务器在国外甚至还考虑了翻墙的做法。但收效甚微，最终都是没达到上传到效果。
首先了解一下Android图片提交的普遍方案：
![图片提交的流程](http://blog.sptechnolab.com/wp-content/uploads/2011/03/phpad1.png)
将图像转换成二进制的Bitmap数据然后进行Base64加密通过字符串进行传输。但是这里有个问题就是安卓的图片上传原则上是不能直接在机子的内存直接操作的，一般情况是先将图片一边写入缓存放入sd卡里，一边利用HttpURLConnection 进行上传。（但这个方法从始至终只成功过一次。。。然后我就崩溃决然了）<!--more-->

<span style="color: #cc99ff;">普通java程序的图片上传</span>

如果不考虑内存的影响，普通java程序下可以利用HttpClient在参数中加入格式化过的图像数据进行传输。或者如找到的一个Demo里一样，利用ImageIO进行流的输入输出。以下是关键代码

<pre class="brush: java; gutter: true"> ByteArrayOutputStream baos = new ByteArrayOutputStream();//字节数组输出流
    ImageIO.write(ImageIO.read(new File(SAMPLE_PIC_FILE)), &quot;jpg&quot;, baos);
        //利用ImageIO将图片一次性转换成二进制数据写入baos里
    URL url = new URL(YOUR URL);
        //服务器地址
    String data = URLEncoder.encode(&quot;image&quot;, &quot;UTF-8&quot;) + &quot;=&quot; 
        + URLEncoder.encode(Base64.encodeBase64String(baos.toByteArray()).toString(), &quot;UTF-8&quot;);
        //写入数据，这里传入服务器接受的名字，如果是自己的服务器可以进行更改，利用到了URLEncode转换成以URL编码的格式
        //Base64进行加密处理注意输入的字符尽可能的进行URLencode编码以防止出错 这里写入的数据与地址栏中?后面的参数格式一致
     data += &quot;&amp;&quot; + URLEncoder.encode(&quot;key&quot;, &quot;UTF-8&quot;) + &quot;=&quot; + URLEncoder.encode(IMGUR_KEY, &quot;UTF-8&quot;);
        //添加别的数据 流入title description等
    URLConnection conn = url.openConnection();//建立连接
    conn.setDoOutput(true);//设置可以使用输出流
    conn.addRequestProperty(&quot;Authorization&quot;, &quot;Client-ID &quot;+IMGUR_CLIENT_ID);
        //添加头文件参数这里还可以添加Content-type connection等
    OutputStreamWriter wr = new OutputStreamWriter(conn.getOutputStream());
        //getoutputstream这一步建立输出流的写入
    //System.out.println(&quot;Sending data...&quot;);
    wr.write(data);//将缓存写入输出流
    wr.flush();//刷新缓冲区确保数据都写入
        String reply = &quot;&quot;;
    StringBuffer sb = new StringBuffer();
    InputStream inStream = conn.getInputStream();//这一步才进行关键的上传操作，创建输入流
    Scanner input = new Scanner(inStream);将输入流写入scanner当作参数
    while (input.hasNextLine()) {
    String nextLine = input.nextLine();
    sb.append(nextLine); }
    reply = sb.toString();//最后获得响应的html代码</pre>

<!--more--><span style="color: #cc99ff;">利用HttpURLConnection的android图片提交</span>

android的图片提交保险起见应该是利用HttpURLConnection的 因为内存小的关系所以做到一次性提交抱错的风险比较大。

这里关键的步骤就是分多次将数据写入缓冲区域，并且常见的就是添加BOUNDARY PREFIX LINE_END等识别信息（这里主要是为了自己开发的服务器识别，例子很多就不赘述了）关键代码如下

<pre class="brush: java; gutter: true">                        URL url = new URL(RequestURL);//请求地址
            HttpURLConnection conn = (HttpURLConnection) url.openConnection();//获取connection
            conn.setReadTimeout(TIME_OUT);//可选
            conn.setConnectTimeout(TIME_OUT);//可选
            conn.setDoInput(true); //必要
            conn.setDoOutput(true); //必要
            conn.setUseCaches(false);  //必要
            conn.setRequestMethod(&quot;POST&quot;);  //必要
            conn.setRequestProperty(&quot;Charset&quot;, &quot;UTF-8&quot;);//可选  
            conn.setRequestProperty(&quot;connection&quot;, &quot;keep-alive&quot;);//可选   
            conn.setRequestProperty(&quot;Content-Type&quot;, CONTENT_TYPE + &quot;;boundary=&quot; + BOUNDARY); //可选
                        //一上几步分别是设置读取超时时间 传输超时时间 是否可以进行输入 是否可以进行输出 是否可用缓存 
                        //确认上传方式 设置编码格式 设置传输的保持连接 这是各种识别信息 

            if(file!=null)
            {
                DataOutputStream dos = new DataOutputStream( conn.getOutputStream());
                //从连接的输出流获取数据输出流
                                StringBuffer sb = new StringBuffer();
                sb.append(PREFIX+BOUNDARY+BOUNDARY);//设置各种标识符
                sb.append(&quot;Content-Type: application/octet-stream; charset=&quot;+CHARSET+LINE_END);
                                //像写http豹纹一样添加各种
                sb.append(LINE_END);
                dos.write(sb.toString().getBytes());//将以上豹纹头写入dos数据输出流中
                InputStream is = new FileInputStream(file);//写入图片文件
                byte[] bytes = new byte[1024];//设置每次读取1024byte
                int len = 0;//这里0和-1都见过 主要是0
                while((len=is.read(bytes))!=-1)//一边赋值给长度一边确认是否不为负数
                {
                    dos.write(bytes, 0, len);//只要没读完反复的输入到数据输出流中
                }
                is.close();//关闭图片的输入流
                dos.write(LINE_END.getBytes());//各种标识符
                byte[] end_data = (PREFIX+BOUNDARY+PREFIX+LINE_END).getBytes();
                dos.write(end_data);
                dos.flush();//刷新缓存 这不很重要
                                 InputStream is = conn.getInputStream();
                int res =   conn.getResponseCode();//获取响应代码 最后就可以抓到Entity了</pre>

但是，但是，但是，非常的，极其的，很生气的，只成功了一次，不知道是我的平板没有sd卡的缘故还是网络的缘故，都不能进行完整的图片操作。。。以上代码经过千万次的筛选确保无误，全中国的帖子都在用这个方法。

<!--more-->

<span style="color: #cc99ff;">利用HttpClient进行Android的图片提交</span>

HttpClient非常常用，客户端模拟登录是经常用到，鉴别输入的参数和请求地址就可以进行模拟Post和Get操作。夙愿完成了。

首先提一下我主要利用的不是本地的服务器，采用了叫做Imgur的网站的无限上传的图床，主要优势是有方便的上传可以利用，地址也好抓，但就是外国的服务器，网络不是非常的稳定。首先注册账号，然后添加一个application获取key'和secret这是它提供的上传功能的参数[caption width="674" align="alignnone"]![](http://i1061.photobucket.com/albums/t476/ov_beeshoot/635583B7-1.jpg) upload参数[/caption]

<pre class="brush: java; gutter: true">private String pic(File fff) 
    {   InputStream inputStream=null;
                InputStream is;
                String reply=&quot;&quot;;
    try {
        inputStream = new FileInputStream(fff);//同样将图片写入输入流
        Bitmap bitmapOrg = BitmapFactory.decodeStream(inputStream);
                //这一步将输入流抓换成Bitmap格式的二进制数据
        ByteArrayOutputStream bao = new ByteArrayOutputStream();
                //创建字节数组输出流，将数据写入它来上传
        bitmapOrg.compress(Bitmap.CompressFormat.JPEG, 90, bao);
                //这一步是重中之重，千辛万苦才在国外找到了这个很少人用的方法，将二进制的Bitmap数据进行压缩，
                //传入的参数是图片格式，压缩率和输出的字节流 很给力啊!
        byte [] ba = bao.toByteArray();//将字节流抓换成字节数组
        String ba1=Base64.encodeBytes(ba);//Base64加密
        ArrayList&lt;NameValuePair&gt; nameValuePairs = new
                ArrayList&lt;NameValuePair&gt;();
                nameValuePairs.add(new BasicNameValuePair(
                                                URLEncoder.encode(&quot;image&quot;, &quot;UTF-8&quot;),ba1));//添加参数
                nameValuePairs.add(new BasicNameValuePair(URLEncoder.encode(&quot;key&quot;, &quot;UTF-8&quot;),
                                                   URLEncoder.encode(IMGUR_KEY, &quot;UTF-8&quot;)));//添加第二个参数
            try{
                HttpClient httpclient = new DefaultHttpClient();//创建HttpClient
                HttpPost httppost = new
                HttpPost(&quot;https://api.imgur.com/3/image&quot;);//请求地址
                httppost.setHeader(&quot;Authorization&quot;,&quot;Client-ID &quot;+IMGUR_CLIENT_ID);//设置请求的header
                httppost.setEntity(new UrlEncodedFormEntity(nameValuePairs));//参数写入Entity中
                HttpResponse response = httpclient.execute(httppost);//解析提交获取响应
                HttpEntity entity = response.getEntity();//响应的文本
                is = entity.getContent();
                StringBuffer sb = new StringBuffer();
                 Scanner input = new Scanner(is);
                   while (input.hasNextLine()) {
                    String nextLine = input.nextLine();
                    sb.append(nextLine);
                    // System.out.println(nextLine);
                   }
                   reply = sb.toString();
                }catch(Exception e){
                //Log.e(&quot;log_tag&quot;, &quot;Error in http connection &quot;+e.toString());
                }
    } catch (FileNotFoundException e) {
        // TODO 自动生成的 catch 块
        e.printStackTrace();
    } catch (UnsupportedEncodingException e1) {
        // TODO 自动生成的 catch 块
        e1.printStackTrace();
    }       
        return reply;
    }</pre>

[参考代码](http://blog.sptechnolab.com/2011/03/09/android/android-upload-image-to-server/)