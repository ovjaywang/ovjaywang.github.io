---
title: effective java--builer构造器
date: 2016-04-22 17:10:48
categories:
  - 代码狗
  - 学习log
tags:
  - java
---
一个实体类往往有不同的属性 属性有的用于构造器 视为构造时的必需参数 而有的可有可无 类似json和xml一般，有多少个属性，每个属性有多少个赋值都不确定 。e.g.动物 名称string name和是否有尾巴istail是必须的，是否吃草iseatgrass 多重weight 交配手段sexmethod 科目等为非必须字段。解决这一构造问题一般有三种方式。
<!-- more -->
1.  可以采用不同的构造函数，一般的，把调用最多的构造出来，其余的按字段个数构造1-n个。诚然，有的字段并不需要，可能赋为空或者0。这样严重的影响了效率，同时字段多时构造函数也异常的多。

2.  采用类似c#的方式 采用一个空字段构造器，使用setter和getter赋值和取值。这样的好处避免了过度的编写构造函数。但是在异步交互中，不能保证该对象的正确性和一致性。因此很可能需要冷冻。例如，名字叫草泥马的动物，是不具有排卵受精的性爱方式的。这样的setter不能及时维护数据的统一。又或者必须给予字段的动物名字和是否有尾巴在构造中并未赋值，造成的结果很可能在后面的调用中无法get到对应属性报异常。

3. 强悍的builder.在实体类中，再编写一个builder用于构造和赋值，必须的字段出现在构造函数中，需要赋值的字段在后面以.setter的方式调用，相当简洁高效，同时易于观察和修正。类似jQuery和python中一样。棒呆。

最丑的长字段实体类原本是这么写的

```
class Animal{
    private string name;//必须参数
    private bool isTail;//必须参数
    private bool isEatGrass;//可选参数
    private double weight;//可选参数
    privaet int sexMethod;//可选参数

    Animal(string name,bool isTail)//这两个字段我特么就想要{
        this.name = name;
        this.isTail = isTail;
    }
    Animal(string name,bool isTail)//这两个字段我特么就想要{
        this.name = name;
        this.isTail = isTail;
    }
    Animal(string name,bool isTail,bool isEatGrass){
        this.name = name;
        this.isTail = isTail;
        this.isEatGrass = isEatGrass;
    }
    Animal(string name,bool isTail,bool isEatGrass,double weight){
        this.name = name;
        this.isTail = isTail;
        this.isEatGrass = isEatGrass;
        this.weight = weight;
    }
    Animal(string name,bool isTail,bool isEatGrass,double weight,int sexMe){
        this.name = name;
        this.isTail = isTail;
        this.isEatGrass = isEatGrass;
        this.weight = weight;
        this.sexMethon = sexMe;
    }
}
```
在某处调用就可能是这样

```java
Animal animal = new Animal("caonima",true,true,-1,3);
//由于我并不想知道体重，但又因为构造函数的关系不得不加上。
//所以相当繁琐。当字段达到几十个时，调用构造函数都分不清参数是否对应上。
```
​将上文的动物实体类以builder的方式写出来，大概是这样的

```
class Animal{
    private final string name;
    private final bool isTail;
    private final bool isEatGrass;
    private final double weight;
    private final int sexMethod;

    public static class Builder{//类内嵌套的静态类，可以通过类名的类名调用构造函数
        private final string name;
        //注意这里 final只对必需字段定义 表明不可修改 区分不同对象
        private final bool isTail;
        //给非必需字段设置默认值
        private bool isEatGrass =false;
        private double weight = 0.0;
        privaet int sexMethod =1;
        public Builder(string name,bool isTail){
            //构造函数只取必需字段
            this.name = name;
            this.isTail = isTail;
        }
        public Builder setISEatGrass(bool isEatGrass){
            this.isEatGrass = isEatGrass;
            return this;
        }
        public Builder setISEatGrass(double weight){
            this.weight = weight;
            return this;
        }
        public Builder setSexMethod(int sexMetho){
            this.sexMetho = sexMetho;
            return this;
        }

        public Animal build(){//在builder类内声明一个构造器 返回animal对象
            return new Animal(this);
        }
    }
    //当然 在animal类的最后 也需要把builder的参数取出来
    private Animal(Builder builder){
        name = builder.name;
        isTail = builder.isTail;
        isEatGrass = builder.isEatGrass;
        weigth = builder.weigth;
        sexMethod = builder.sexMethod;
    }
}
```
在某处构造该类对象时，即可像下面一样声明

```
    Animal dog = new Animal.Builder("旺财",true).setIsEatGrass(false).setSexMethod(3).setWeight(20.0)
```
builder类似一个抽象工厂 客户端可以将这样一个builder传给方法，在方法中生成需要数量的对象。1.5+版本可以使用泛型定义。

```
public interface Builder{
    public T build();
}
```
在类似生成二叉树，kd树的时候，往往用户只需要将数据push进入，然后调用一个builder整棵树就构造好了。这种方式往往类似于带限制通配符类型的builder.

​

还有一点值得注意的是，新增字段时，不比增加更多的构造函数 ！

caution!

1. 增加了类长度，当字段较多时才较好的使用。

2. 创建对象首先得创建构造器，在强调性能的地方会比较麻烦。

