---
title: effective java 笔记--创建对象的几个tips
date: 2016-04-27 11:50:01
tags:
---
第一章-创建和销毁对象 的一些 小tips

<!-- more -->
# 不希望被实例化 又不想抽象化
某些类不希望被实例化出毫无意义的对象 但不编写显示构造器类会自定义一个隐式无参的构造器 因此可以用一下方式，编写不能实例化的类：

```java

  public class UtilityClass{
    private UtilityClass(){
      throw new AssertionError();//直接抛出assert验证异常 好坏
    }
  }
```
该构造器是私有的。其他类不能过访问（即使同包）。

# 不要创建没必要的对象

不可变的对象一定要重用（singleton）；重用绝逼不会变的对象（例如计时初始时间、例如物种的属性）。例如，实例化Person对象的时候判断是不是计划生育年代出生的，即判断出生年份是否在1983-2015年。

```

class Person{
   private final Date birthDate;
   private static final Date BIRTH_START;
   //这俩参数都是不变的 为了对比出生时间 
   //没必要每次都创建一个日期对象，只在初始化时调用。
   private static final Date BIRTH_END; 

//静态块的方法 第一次声明Person对象时构建（不用实例化）
// 后面占用一块内存 不用再次创建
   static{
        Calendar gmtCal = Calendar.getInstance(TimeZone.getTimeZone("GMT"));
        gmtCal.set(1983,Calendar.JANUARY,1,0,0,0);
        //这里对日期做了时区判断 
        //保证都是从某一客观时间点出生的
        BIRTH_START = gmtCal.getTime();
        gmtCal.set(2016,Calendar.JANUARY,1,0,0,0);
        //截止2015年年底 所以取2016年年初
        BIRTH_END = gmtCal.getTime();
   }

    public boolean isJiHuaShengYu(){
        return birthDate.compareTo(BIRTH_START) >=0 &&
            birthDate.compareTo(BIRTH_END) <=0;
    }
}
```

如果isJiHuaShengYu方法<font color="red">从未调用</font>，这个方法就显得不合时宜了，这块静态块的内存就一直占着，因此，可以采用延时初始化的方式，在第一次调用该方法时调用。

adapter适配器模式可以提供一个后备对象，使实例化次数减少（包括类适配器和对象适配器）；autoboxing自动装箱模式能够自动创建多余对象，所以尽量少用，而使用基本数据类型。

## 适配器Adapter

>adapter工作场景

适配器模式工作场景是：给定的一个接口不满足需求，需要其一些功能与其他的接口或类配合使用.正如已经给了电压和充电插座，但是插座电压与设备工作电压，这是时候就需要一个转换接头。

>adapter分类 

一般分为类适配器和对象适配器。**类适配器**一般继承了给定接口的被适配类，同时拓展实现了标准的接口；而**对象适配器**则直接关联，以委托的方式完成已知接口的特殊功能。

>类适配器实例：Adaptee（已知的待适配的类） Target（目标类） Adapter（适配器类）


```java

    // 已存在的、具有特殊功能、但不符合我们既有的标准接口的类
class Adaptee {
    public void specificRequest() {
        System.out.println("被适配类具有 特殊功能...");
    }
}
```

```java

// 目标接口，或称为标准接口
interface Target {
    public void request();
}

// 具体目标类，只提供普通功能
class ConcreteTarget implements Target {
    public void request() {
        System.out.println("普通类 具有 普通功能...");
    }
}
```

```java

// 适配器类，继承了被适配类，同时实现标准接口
class Adapter extends Adaptee implements Target{
    public void request() {
        super.specificRequest();
    }
}
```

```java

// 测试类
    public class Client {
        public static void main(String[] args) {
            // 使用普通功能类
            Target concreteTarget = new ConcreteTarget();
            concreteTarget.request();
            
            // 使用特殊功能类，即适配类
            Target adapter = new Adapter();
            adapter.request();
        }
    }
```

> 对象适配器实例 Adapter适配器类 被适配器同上

```

// 适配器类，直接关联被适配类，同时实现标准接口
class Adapter implements Target{
    // 直接关联被适配类
    private Adaptee adaptee;
    
    // 可以通过构造函数传入具体需要适配的被适配类对象
    public Adapter (Adaptee adaptee) {
        this.adaptee = adaptee;
    }
    
    public void request() {
        // 这里是使用委托的方式完成特殊功能
        this.adaptee.specificRequest();
    }
}
```
##
> autoboxing方式