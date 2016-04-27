---
title: OWL的语义语法及Jena初步
categories:
  - 什么都学一下
  - 代码狗
  - 学习log
tags:
  - jena
  - 本体
  - 语义网
id: 304
date: 2013-07-21 05:07:26
---

1.  **OWL演化**

OWL是语义WEB的层级结构核心，在语义层次上解决WEB信息共享和交换的基础。它是RDFS的衍伸，并基于DAML+OIL的拓展。RDFS相当于二元谓词，相当于类和属性的层级体系，但是存在以下几点局限性：

*   属性的局部范围——rdfs：range对所有的类定义了属性的值，但是不能仅仅对某几个类进行约束。eg.不能描述牛吃草，而别的动物也可能吃肉。
*   不相交——有些类是不相交的。eg.例如男人和女人是不相交的，但是在RDFS中只能声明为“人”的子类。
*   类的布尔组合——可以通过类的布尔组合，如并、交、补来定义新类。eg.将“人”定义为“男人”和“女人”的并。
*   基数约束——可以对属性值的数量进行限制。eg.一个人有一个人的父亲和一个母亲；一门课至少由一位老师任教等。
*   属性性质——可以声明一个属性是传递的（如大于）或者一个属性的值是唯一的（母亲），或者一个属性是另一个属性的逆（吃与被吃）
OWL支持一下三种描述本体的条件

*   良定义语法
*   形式化语义
*   有效的推理支持，包括：

1.  <span style="color: #ff0000;">类成员关系的推理</span>-x是类C的实例，类C是类D子类，则x也是D的一个实例
2.  <span style="color: #ff0000;">类的等价关系推理</span>-A等价于B，B等价于C，则A等价于C
3.  <span style="color: #ff0000;">一致性推理</span>-x是A类的一个实例，类A是类B与类C的交的子集，类A是类D的一个子类，而B与D不相交。这里出现了不一致性。因为A为空，却又实例x
4.  <span style="color: #ff0000;">分类推理</span>-如果已经声明“属性-值”对是实例属于类A的充分条件，则如果x满足该条件，即具有这样的“属性-值”对，则x一定是A的实例

**2.OWL三种分类**

*   <span style="line-height: 15px;"><span style="color: #ff0000;">OWL-FULL</span>：提供了最强大的表达能力和使用RDF语法的最大自由，但是没有可计算性的保证。OWL FULL中类即是<span style="color: #ff0000;">个体的集合<span style="color: #000000;">也可以是<span style="color: #ff0000;">个体<span style="color: #000000;">。与OWL DL和Lite相比，个体和类是分离的集合，即一个资源不能同时是个体和类，但在Full中可以。OWL FULL允许本体对预定义的RDF或OWL词汇表的增强，但是难以实现OWL FULL中所有构子的推理。</span></span></span></span></span>

它的优点是和RDF完全兼容，所有合法的RDF文档都是合法的OWL FULL文档，所有RDF(S)结论都是OWL FULL结论。OWL FULL的缺点是该语言的表达能力太强，以致推理变得不可判定。如果将保持对RDF的兼容性作为主要目标，则应该选择FULL。OWL FULL将RDF拓展为一个完备的本体语言，克服了OWL DL和RDFS在语义上的冲突，但是OWL FULL消除了基数限制中对传递型属性的约束，因此不能保证推理是可判定的。

*   <span style="line-height: 15px;"><span style="color: #ff0000;">OWL DL</span>：OWL保留了强大表达能力的同时，也具有<span style="color: #ff0000;">计算完备性</span>（所有结论都是可以判定的）和<span style="color: #ff0000;">可判定性</span>（所有的计算都在有限时间内终止）。DL对应于描述逻辑（Description Logic），而描述逻辑是OWL的形式化基础。</span>

<span style="line-height: 15px;">OWL DL的特点是它支持有效的推理，缺点和Lite一样，失去了RDF的兼容性。一个RDF文档在成为OWL DL文档之前必须进行适当的拓展和约束，一个合法的OWL DL文档是一个RDF文档。如果用户保证推理的可判定型和本体语言的强表达能力作为主要目标，则可以选用OWL DL。OWL DL忽略了对RDFS的兼容性，它主要针对概念，属性，个体之间关系的描述，以保证强大的语义表达能力。DL采用了RDFS的数据类型机制和内嵌的XMLSchema数据类型。</span>

*   <span style="line-height: 15px;"><span style="color: #ff0000;">OWL Lite</span>：它是支持分类体系和简单约束的owl语言，例如基数约束，但是基数的值只能是1和0。虽然它的表达能力较弱，但是提供了词典和分类体系提升为本体的快速解决方案。可以通过对OWL DL子语言进行限制，得到OWL Lite子语言，OWL Lite的计算复杂度比DL低。Lite除去了DL中的枚举类、不相交声明和任意基数约束。</span>

<span style="line-height: 15px;"> OWL Lite的特点是易学易用，如果关注的重点是一个简洁的本体语言，就可以选用OWL Lite。OWL Lite增强了OWl DL中的公理约束，可以保证快速高效的推理。OWL Lite可以利用已经实现的逻辑推理系统进行推理。</span>

[![owl11](http://farm6.staticflickr.com/5461/9335813301_870c4db228.jpg)](http://www.flickr.com/photos/ovbeeshoot/9335813301/ "Flickr 上 Liar1992 的 owl11")

**3.OWL的类**

-类提供了对具有相似特性的资源进行分组的抽象机制，和RDFS的类相似。

-类包括<span style="color: #ff0000;">类的内涵</span>和<span style="color: #ff0000;">类的外延</span>，内涵是类的描述和类的公理的总和，外延是类的实例的集合体。

-OWL Lite和OWL DL中，个体不能为类，类和个体属于不相交的论域，属性和数据值也是如此。但是在owl FULL中允许自由使用RDFS，一个类或者元类可能同时为另一个类的实例。

OWL的类通过<span style="color: #ff0000;">类描述<span style="color: #000000;">进行描述并使用</span><span style="color: #000000;"><span style="color: #ff0000;">类公理</span>进行结合。</span></span>

*   **<span style="line-height: 28px;">本体的头部</span>**

        OWL本体是一个RDF文档，包含在一个<span style="color: #ff0000;">rdf:RDF</span>元素中。一个OWL本体可能以一系列的关于本体的声明作为开始，包含注释、版本控制以及导入其他本体等内容，称为本体头部。
<span style="color: #ff0000;">owl:Ontology</span>元素是用来声明关于当前文档的OWL元数据的，可以记录版本信息和导入文档所依赖的一些定义。rdf:about属性为本体提供名称和引用，一般取值为空表示本体的名称是owl:Ontology元素的基准URI，即本体所在的文档URI或上下文中<span style="color: #ff0000;">xml:base</span>指定的URI。<span style="color: #ff0000;">rdfs:coment</span>和<span style="color: #ff0000;">rdfs:label</span>为本体提供自然语言注释和标签。<span style="color: #ff0000;">owl:priorVersion</span>为本体版本控制提供相关信息。owl:imports用于导入其他一些文体，将内容引入为当前文档的一部分。

因此一个标注OWL文档的模式如下：

<pre class="brush: xml; gutter: true">&lt;?xml version=&quot;1.0&quot;?&gt;
&lt;rdf:RDF
    xmlns:foaf=&quot;http://xmlns.com/foaf/0.1/&quot;
    xmlns:rdf=&quot;http://www.w3.org/1999/02/22-rdf-syntax-ns#&quot;
  xmlns:xsd=&quot;http://www.w3.org/2001/XMLSchema#&quot;
    xmlns:rdfs=&quot;http://www.w3.org/2000/01/rdf-schema#&quot;
    xmlns:aaa=&quot;http://www.w3.org/2002/07/aaa#&quot;
    xmlns:abc=&quot;http://www.w3.org/2003/01/geo/abc#&quot;
    xmlns=&quot;http://geoontology.altervista.org/abc.owl#&quot;
  xml:base=&quot;http://geoontology.altervista.org/bbb.owl&quot;&gt;
  &lt;owl:Ontology rdf:about=&quot;&quot;&gt;
    &lt;rdfs:comment xml:lang=&quot;en&quot;&gt;Hi&lt;/rdfs:comment&gt;
    &lt;rdfs:comment xml:lang=&quot;it&quot;&gt;Hello.&lt;/rdfs:comment&gt;
    &lt;owl:versionInfo rdf:datatype=&quot;http://www.w3.org/2001/XMLSchema#string&quot;
    &gt;0.0.2&lt;/owl:versionInfo&gt;
    &lt;rdfs:label rdf:datatype=&quot;http://www.w3.org/2001/XMLSchema#string&quot;
    &gt;……&lt;/rdfs:label&gt;
  &lt;/owl:Ontology&gt;

&lt;owl:DatatypeProperty rdf:ID=&quot;……&quot;&gt;
             ……
  &lt;/owl:DatatypeProperty&gt;
 &lt;owl:ObjectProperty rdf:about=&quot;……&quot;&gt;
             ……
  &lt;/owl:ObjectProperty&gt;
&lt;/rdf:RDF&gt;</pre>

*   **<span style="line-height: 15px;">类描述</span>**

类描述是类公理的基本构造模块，也成为类定义。类描述通过类的名称，或者通过指定一个匿名类的外延来描述一个OWL类。以下是类描述的六种类型：

<span style="color: #ff0000;">—简单类描述</span>：<span style="color: #ff0000;"> owl：Class</span>命名实例，是rdfs：Class的子类。如：

&lt;owl:Class rdf:ID="Human"&gt;

又如：

<pre class="brush: xml; gutter: true">&lt;owl:Class rdf:ID=&quot;Person&quot;/&gt;
    &lt;owl:Class rdf:ID=&quot;Male&quot;/&gt;
    &lt;owl:Class rdf:ID=&quot;Man&quot;&gt;
    &lt;rdfs:subClassOf rdf:resource=&quot;#Person&quot;/&gt;
&lt;/owl:Class&gt;
&lt;owl:Class rdf:about=&quot;#Man&quot;/&gt;
    &lt;rdfs:subClassOf rdf:resource=&quot;#Male&quot;/&gt;
&lt;/owl:Class&gt;</pre>

这里定义了Person、Man、Male三个类，并且声明了Man是Person和Male的子类。

需要注意的是Lite和DL中，所有类的声明都<span style="color: #ff0000;">必须</span>采用<span style="color: #ff0000;">owl:Class</span>或<span style="color: #ff0000;">owl：Restriction</span>。如果对于枚举、交、并、补的类描述赋予一个RDF标识符，则这并不是一个描述类，而是一个完全类的类公理。

owl中保留了两个预定义的类，分别是<span style="color: #ff0000;">owl：Thing</span>和<span style="color: #ff0000;">owl：Nothing</span>。前者是所有个体的集合，后者是空集。所有的owl类都是owl：Thing的子类，而owl：Nothing是所有类的子类。owl子类的概念和rdfs：subClassOf相同。

<span style="color: #ff0000;">—枚举类</span>：采用<span style="color: #ff0000;">owl：oneOf</span>进行类描述，该内置OWL属性的值必须是一个个体或类的实例的列表。因此可以通过枚举实例的方法来描述类。该类的外延就是所有枚举类的个体。如下RDF/XML语法定义了一个包括地球大洲的类

<pre class="brush: xml; gutter: true">&lt;owl:Class&gt;
    &lt;owl:oneOf rdf:parseType=&quot;Collection&quot;&gt;
        &lt;owl:Thing rdf:about=&quot;#Eurasia&quot;/&gt;
        &lt;owl:Thing rdf:about=&quot;#Africa&quot;/&gt;
        &lt;owl:Thing rdf:about=&quot;#NorthAmerica&quot;/&gt;
        &lt;owl:Thing rdf:about=&quot;#SouthAmerica&quot;/&gt;
        &lt;owl:Thing rdf:about=&quot;#Australia&quot;/&gt;
        &lt;owl:Thing rdf:about=&quot;#Antarctica&quot;/&gt;    
    &lt;/owl:oneOf&gt;
&lt;/owl:Class&gt;</pre>

其中，&lt;owl:Thing rdf:about="…"/&gt;引用的是个体。

又如：

<pre class="brush: xml; gutter: true"> &lt;owl:Class rdf:ID=&quot;RGBColor&quot;&gt;
      &lt;owl:oneOf rdf:parseType=&quot;Collection&quot;&gt;
           &lt;owl:Thing rdf:about=&quot;#Red&quot;/&gt;
           &lt;owl:Thing rdf:about=&quot;#Green&quot;/&gt;
           &lt;owl:Thing rdf:about=&quot;#Blue&quot;/&gt;
      &lt;/owl:oneOf&gt;
&lt;/owl:Class&gt;</pre>

这里定义了颜色类，根据定义，所有的个体都是owl：Thing的实例OWL Lite不能使用枚举类。

—属性限制<span style="color: #ff0000;">owl：Restriction</span>，是特殊类型的类描述。它描述了匿名类，即满足限制的所有个体的类，包括值约束（value constraint）和基数约束（cardinality constraint）。

值约束是当属性作用于特定的类描述时对其值进行约束，如可能引起某些个体，它们的属性adjacentTo 的值应该是Region，相当于定义域或者值域的概念，并将其应用于类公理内，甚至就是Region的类公理内。OW中的值约束不仅仅类似rdfs：range只在类描述中起作用，也在使用该属性的所有场合有效。

基数约束在特定类的描述中对属性值的数量进行限制，例如hasPlayer在足球队有11个值，但是在篮球只有5个值。OWL还支持定义全局属性技术的构子，即owl：FunctionalProperty和owl：InverseFunctionalProperty。形式如下：

<pre class="brush: xml; gutter: true">&lt;owl:Restriction&gt;
    &lt;owl:onProperty rdf:resource=&quot;(some property)&quot;&gt;
&lt;/owl:Restriction&gt;</pre>

属性限制也可以作用于数值属性和对象属性，前者的值是数据字面量，后者的值为个体

值约束包括：

<span style="color: #000000;"><span style="color: #ff0000;">owl：allValuesFrom</span>，连接约束类和类描述货数据范围。它特定属性的值（属性的值域）要么是类描述的成员，要么是指定数据范围内的数据值，一个简单的例子：</span>

<pre class="brush: xml; gutter: true">&lt;owl:Restriction&gt;
    &lt;owl:onProperty rdf:resouce=&quot;#hasParent&quot;/&gt;
    &lt;owl:allValuesFrom rdf:resouce=&quot;#Human&quot;/&gt;
&lt;/owl:Restriction&gt;</pre>

它描述了一个OWL的匿名类，对其所有的个体而言，如果它具有属性hasParent，则属性hasParent的值必须是类Human。在OWL Lite中owl：allValuesFrom和一阶谓词逻辑中的全称量词类似。

<span style="color: #ff0000;">owl：someValuesFrom</span>，它连接约束类和描述类或者数据范围。它描述的类中至少有一个属性的值为描述类或者数据范围内的值。即可能存在部数据描述类或者数据范围的实例。以下例子描述了个体的类，这些个体的父母至少有一个属于Physician

<pre class="brush: actionscript3; gutter: true">&lt;owl:Restriction&gt;
    &lt;owl:onProperty rdf:resouce=&quot;hasParent&quot;/&gt;
    &lt;owl:someValuesFrom rdf:resouce=&quot;#Physician&quot;/&gt;
&lt;/owl:Restriction&gt;</pre>

<span style="color: #ff0000;">owl:hasValue</span>它连接约束类和一个V，V可以为个体或者数据值，它描述所有个体的一个类，其属性最少有一个值在语义上等于V，类似AllvaluesFrom的唯一值定义。Lite中没有该值约束，示例如下，描述了一个类，个体Clinton是他/她们的父母亲：

<pre class="brush: xml; gutter: true">&lt;owl:Restriction&gt;
     &lt;owl:onProperty rdf:resource=&quot;#hasParent&quot;/&gt;
     &lt;owl:hasValue rdf:resouce=&quot;#Cliton&quot;/&gt;
&lt;/owl:Restriction&gt;</pre>

基数约束包括：

<span style="color: #ff0000;">owl:maxCardinality</span>：

<span style="color: #ff0000;">owl:minCardinality</span>：

<span style="color: #ff0000;">owl:cardinality</span>：

—

—

—

第一种类型通过类名描述一个类，其余通过对类的外延进行约束来描述匿名类。第二种不多不少枚举类的个体。第三种包括满足属性限制的所有类。四五六类型包括满足类描述布尔运算的类，可以看成是与、或、非。后面四中类描述可能导致嵌套结构，因此理论上可以构造任意复杂的类描述。

参考资料

[本体语言OWL](http://blog.sina.com.cn/s/blog_6c42aaa60100nhps.html "本体语言OWL")

[语义Web原理及应用](http://www.amazon.cn/gp/product/B002NKMS7S/ref=as_li_qf_sp_asin_tl?ie=UTF8&amp;camp=536&amp;creative=3200&amp;creativeASIN=B002NKMS7S&amp;linkCode=as2&amp;tag=ovjaywang-23)![](http://ir-cn.amazon-adsystem.com/e/ir?t=ovjaywang-23&amp;l=as2&amp;o=28&amp;a=B002NKMS7S)