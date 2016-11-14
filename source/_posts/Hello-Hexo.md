---
title: Hexo双备份Github和Coding及域名绑定
date: 2016-04-10 22:51:49
tags: [hello]
categories: 槽
---
感谢[这个教程](http://zipperary.com/2013/05/28/hexo-guide-2/)
还有[这个](http://ibruce.info/2013/11/22/hexo-your-blog/?utm_source=tuicool)
解决了好多bugs的[这个](http://www.jianshu.com/p/35e197cb1273)
优化cdn图床缓冲的[这个](http://lukang.me/2015/optimization-of-hexo-2.html)
[这个](http://www.dute.me/)让我找到了便宜的狗爹优惠码

    从一开始的阵地wp搬到这里来了。后会有期了dooby.me 再见了WordPress  
    默默的是有些不舍。虽然push很慢虽然被墙了大半  
    4年光阴不复返啊。从一个逗比成长为大逗比了。

<!-- more -->
hexo坑还是挺多的。
$$ \begin{bmatrix} x \\\\ y \\\\ 1 
\end{bmatrix} \sim \begin{bmatrix} f & 0 & 0 & 0 \\\\ 0 & f & 0 & 0 \\\\ 0 & 0 & 1 & 0 \end{bmatrix} \begin{bmatrix} r\_{11} & r\_{12} & r\_{13} & t_x \\\\ r\_{21} & r\_{22} & r\_{23} & t_y \\\\ r\_{31} & r\_{32}& r\_{33} & t_z \\\\ 0 & 0 & 0 & 1 \end{bmatrix} \begin{bmatrix} p \\\\ q \\\\ 0 \\\\ 1 \end{bmatrix} \sim \begin{bmatrix} fr_11 & fr_12 & ft_x \\\\ fr_21 & fr_22 & ft_y \\\\ r\_{31} & r_32 & t_z \end{bmatrix} \begin{bmatrix} p \\\\ q \\\\ 1 \end{bmatrix} 
$$$$\sim \begin{bmatrix} h_11 & h_12&h_13 \\\\ h_21 & h_22 & h_23 \\\\ h_31 & h_32 & h_33 \end{bmatrix} \begin{bmatrix} p \\\\ q \\\\ 1 \end{bmatrix}$$


$$f(x,y)=\left\\{\begin{matrix}
\frac{4}{m}& 0<x<\frac{a}{2},0<y<\frac{\pi }{2} \\\\
0 & Others
\end{matrix}\right.$$

# 公式和Markdown冲突#
由于下划线在LaTex公式编辑器和MarkDown中都有，因此如果在公式中有两个xiahuaxian"_"则必须注意不要发生冲突，需加入反斜杠在下划线前！！在SublimeText中也可以看到如果又两个双下划线字体已变成斜体。

$$ 2H\_2 = 2O\_2+H\_2$$

# WordPress迁移 #
这个使用hexo-migrate-wordpress插件还不错 但是切记导出的xml是文章 tag和category会自动建好。否则使用hexo migrate wordpress 会报错
> 沙扬娜拉dooby.me

![dooby.me](http://ooo.0o0.ooo/2016/05/02/5727757865698.png)
# Github和Coding双备份 #
按照一般的教程都能配置好Github的。但是Coding对国内线路速度相对较快（好蛋疼GitCafe已经被收了）配置文档差不多是这样
```markdown
deploy:
   type: git
   repo: 
      github: https://github.com/{YOUR_ID}/{YOUR_ID}.github.io.git,master
      coding: https://git.coding.net/{YOUR_ID}/{BLOG　NAME}.git,master
```
当然也需要先将ssh的key添加上去（同一个码即可），Coding不需要同一个名字..然而最近发现Coding的项目展示页面收费了……虽说是注重团队协作和代码交易 但是个人用户的托管真的完全要丢掉了吗？差不多1块钱1天的样子 相当不划算

# SEO是极好的 #
切莫贪杯 google就好 但是 切记！标题不要带有‘&’。年少无知，这个符号不能正常解析出xml会报错

# 域名绑定 #
切记@解析的是{YOUR_ID}.github.io的地址而不是github.com的 CNAME需要放在source下才能被generate。dnspod被企鹅注资后ui变得相当土了

# 专注写作 #
放弃各种挂饰 插件 统计吧 seo和评论就足够了。多写作多讨论 当然很想适时的学一发nodejs因为折腾了好久ig的照片还是没能get进来。


{% gist 86cc40065a691b7c4534e483249583dd transValue.java %}

{% blockquote @DevDocs https://twitter.com/devdocs/status/356095192085962752 %}
NEW: DevDocs now comes with syntax highlighting. http://devdocs.io
{% endblockquote %}

{% blockquote David Levithan, Wide Awake %}
Do not just seek happiness for yourself. Seek happiness for all. Through kindness. Through mercy.
{% endblockquote %}

{% blockquote Seth Godin http://sethgodin.typepad.com/seths_blog/2009/07/welcome-to-island-marketing.html Welcome to Island Marketing %}
Every interaction is both precious and an opportunity to delight.
{% endblockquote %}

{% codeblock hello.java java https://gist.github.com/ovjaywang/86cc40065a691b7c4534e483249583dd %}
void transValue(int a,int b){
        a ^= b;
        b ^= a;
        a ^=b;
    }
{% endcodeblock %}

{% codeblock %}
alert('Hello World!');
{% endcodeblock %}

{% pullquote [class] %}
blah blah blah
{% endpullquote %}

2016.04.11