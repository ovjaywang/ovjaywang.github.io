---
title: 'effective java笔记--静态工厂和Builder模式'
date: 2016-04-13 15:15:14
categories:
  - 代码狗
  - 学习log
tags:
  - test
---

# 静态工厂 #
```java
    public static Boolean valueOf (boolean b){
        return b?Boolean.True:Boolean.False;
    }
```
静态工厂以其名字显见，以[类名.方法名]的方式调用，其静态函数主要返回其类的对象，一般是引用值。返回什么东西，由参数决定，这个工厂给你产出来。其在API开发中使用广泛。
<!-- more -->
静态工厂最有趣的地方在于把方法**提供者**同方法**使用者**剥离开来，使用者不能明细地看到对象构造的具体过程，而是可见可得地获得所需的对象。当然好处有很多，例如返回的对象可以是该类的子类，能让被调用静态工厂方法的类不成为公有；例如有效的降低了构造对象的成本，静态工厂**「**不生产对像，一般只是对象的搬运工**」**。使用静态方法返回对象的方式在单态中使用广泛。

List、Set有着相似的方式。它们都继承自Collection，同时不能实例化为该类的对象，必须通过子类实现父类的方法来实例化不同属性的对象。ArrayList HashSet等都通过注册的方式进行映射。
>单态就是整个类能且只能有一个对象存在，为null则构造一个，不为null应返回该对象。例如购物车只能有一个、聊天平台只能有一个、饭馆的点菜平台只能有一个 这种类似的场景。

与不同参数的构造函数不同，不同参数的静态工厂可能返回同一属性的该(子)类的对象。该方式抛弃了重复冗杂的new形式，在C#常见newInstance()构造新对象，而在Java里就比较少的出现，一般都是左右对等的。

最上面这段代码最有趣的地方在于，它是Boolean的包装类，输入的是值类型而返回的是Boolean对象,当然Boolean只实例化了两个对象。但当新的子类构建，简单的静态工厂必须重写，常用的改进方法有：

## 反射机制+配置
  例如下面一段就是根据不同的类名，返回不同的对象

```java
    public static InterFace_Name staticFactory(String name)throws
                                InstantiationException,
                                IllegalAccessException,
                                ClassNotFoundException{     
        // 这边使用的是Java的Reflection机制来产生实例
        // 以后就算改变了这边的程式，客户端程式是不用更改的
        return (InterFace_Name) Class.forName(name).newInstance();
    }
```
## 注册方式(Service Provider Framework)

>注册方式需要
1. 服务具体实现类(由服务提供商实现)
2. 服务提供者实现类，1为2的实例，(由服务提供商实现)
3. 服务定义接口，定义服务内容，不包含实现。
4. 服务提供者接口，3为4的实例，定义获取提供者的方式，不包含实现。
5. 服务提供者注册类
最常见的实例就是JDBC(Java DataBase Connection)。下面这段最常见的连接mysql的标准函数就很好的体现了注册的方式。
```java
    public synchronized static Connection getCon() 
        throws ClassNotFoundException, SQLException
    {
        //服务提供者接口
          String DRIVERNAME = "com.mysql.jdbc.Driver";
          //java.sql.Driver.class这个是服务提供者接口，
          //服务提供者若使mysql，那就使用"com.mysql.jdbc.Driver"；
          //如果是sql server，那就使用"com.microsoft.sqlserver.jdbc.SQLServerDriver";
          //如果是Oracle，那就要用"oracle.jdbc.driver.OracleDriver"...
          String URL = "jdbc:mysql://URL/DataBase_Names";
          String USER = "USER";
          String PWD = "PWD"; 
          Connection connection = null;//接口 由服务提供者提供并实现具体服务 
          Class.forName(DRIVERNAME);
          //这里映射通过DriverManager判断获取的是哪个服务
          connection = (Connection) DriverManager.getConnection(URL, USER,PWD);
          //链接数据库  mysql已经在驱动管理注册了API(本机装mysql的时候)
          //这里使用者编写的服务访问getConnection这个API，
          if(connection!=null){return connection;}
          else{return null;}//具体的数据库操作逻辑   
    }  
```
上式的意义就是，使用**Java**连接上**数据库**。注意，不是某种数据库，是数据库。数据库的提供商按照定义的接口（Java提供的,增删改查等数据库操作），都可以顺利的连接上它们的数据库。  
- ``Class.forName("...")``这句实例化一个mysql等数据库提供商的<font color="red">服务提供者实现类</font>，并将这个类的实例注册到DriverManager即<font color="red">服务提供者注册类</font>。
- 通过URL指明连接的地址和端口，判断所连接的数据库类别，在利用USERNAME PWD参数连接到数据库获取操作数据库的一个连接Connection。
- Connection作为一个实现类，客户端的程序得到了这个实例就可以操作各种数据库，但其内部的原理对客户端不可见的，这就是所谓的面向接口编程。

这里举一个炸裂的例子。
1. 具体服务接口定义了live和die，然后实现类实现了具体的方法
2. 服务提供者接口定义了获取服务实例的函数，然后服务提供者实现类实现了注册方式获取实例
3. 注册类则对ClassName和服务提供者接口进行绑定。相当于，对不同的服务提供商通过名字进行映射
4. 调用类则通过常规步骤，调用具体服务。

HaInterface.java
{% codeblock lang:java %}
    package com.gua.com;
    public interface HaInterface {
        public void Live();
        public void Die();
    }
{% endcodeblock %}  
HaClass.java
{% codeblock lang:java %}
    package com.gua.com;
    public class HaClass implements HaInterface{
        @Override
        public void Live() {
            System.out.println("给你们一些人生经验");
        }
        @Override
        public void Die() {
            System.out.println("又想搞大新闻!");
        }
    }
{% endcodeblock %}  
Xuyimiao.java
{% codeblock lang:java %}
    package com.gua.com;
    public interface Xuyimiao {
        public HaInterface HahaGo();
    }
{% endcodeblock %}  
WoyaoXuyimiao.java
{% codeblock lang:java %}
    package com.gua.com;
    public class WoyaoXuyimiao implements  Xuyimiao{
        static{
            MingwangManager.registerProvider("辣妹子辣", new WoyaoXuyimiao());  
        }
        @Override
        public HaInterface HahaGo() {
            return new HaClass(); 
        }
    }
{% endcodeblock %}  
MingwangManager.java

```
package com.gua.com;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;
public class MingwangManager {
    public MingwangManager(){};
    private static final Map<String, Xuyimiao> providers = new ConcurrentHashMap<String, Xuyimiao>();  
    public static void registerProvider(String name, Xuyimiao p) {
            providers.put(name, p);  
        }  
    public static HaInterface getService(String name) {       
            Xuyimiao p = providers.get(name);  
            if (p == null) {  
                throw new IllegalArgumentException(  
                        "No provider registered with name:" + name);  
            } 
            return p.HahaGo();
        }  
}
```  
TestHa.java
{% codeblock lang:java %}
    package com.gua.com; 
    public class TestHa {  
        public static void main(String[] args) throws ClassNotFoundException {
            Class.forName("com.gua.com.WoyaoXuyimiao");  
            HaInterface hi = MingwangManager.getService("辣妹子辣");  
            hi.Live();  
            hi.Die();  
        }
    }
{% endcodeblock %}
## 这种方式已经不推荐-工厂模式
[工厂方法](https://zh.wikipedia.org/wiki/%E5%B7%A5%E5%8E%82%E6%96%B9%E6%B3%95)  [抽象工厂](https://zh.wikipedia.org/wiki/%E6%8A%BD%E8%B1%A1%E5%B7%A5%E5%8E%82)



参考文章:[JAVA 服务提供者框架介绍]、[构造器和静态方法]
[JAVA 服务提供者框架介绍]:(http://liwenshui322.iteye.com/blog/1267202) 
[构造器和静态方法]:(http://blog.csdn.net/csdn0123/article/details/7388445)

<script src="../../../../js/highlight.min.js"></script>
<link href="../../../../css/monokai_sublime.min.css" rel="stylesheet">
<script>hljs.initHighlightingOnLoad();</script>  