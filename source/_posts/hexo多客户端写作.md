---
title: Hexo多客户端写作
date: 2016-04-27 11:50:01
tags:
---

> Hexo是很棒的静态博客，简单的安装配置即可专注写作。  
> 但是强迫症就是很爱折腾。倘若：
> 
> + 主机烧了数据备份怎么破？
> + 换了系统想继续之前的写作怎么破?
> + 纯粹就是想多客户端写作怎么破？
> 

我就纯属蛋疼的。喜欢在办公和休息的地方都写作；或者我一win本
一mac 都想写文章。但Hexo的静态部署的原则不像WordPress一样，能关联博客地址，利用数据库进行博客更新。好了，这里就来解决这个问题。┑(￣Д ￣)┍

<!-- more -->
#  Hexo写作的部署逻辑&其他

##  Hexo的写作特点
Hexo最大特点就是静态博客，即部署到服务器的博客内容是静态写好的html（由markdown转义），而不像类似wordpress的数据都存放在服务器端数据库，每次更新即一次读写数据库的操作。所以如果需要更改博客内容、发布新内容，必须

1. 更改源文件source文件夹下_post及其他各页面文件夹的md文件
2. 通过``hexo g``或``hexo generate``在public生成静态文件。
3. 利用``hexo-deployer-git``插件及``hexo d``命令将public的文件自动的部署到创建好的``user-name.github.io``项目中。
> 在github项目中可以看到，自动上传的只有public文件夹中的文件，包括以年份分类的静态页面文件夹、tag、categories及其他辅助css、js、img等，还有域名指引的CNAME和rss sitemap等。
4. 在博客中看到更新的内容。当然，本地目录中，即存放了md源文件，也包含了生成的public文件。其实，本地并不需要public里的内容，只要有写作的源文件，generate一下就能生成。

##  Hexo的缺憾
那么问题来了，如果我在公司和贫民窟都想写作怎么办。总不能在两个本地仓库hexo init都部署到pages项目。在公司更新一篇blog里面全都是公司写的，然后回到地下室又写一篇更新又全是小黑屋的内容。  

尽管Hexo让人专注写作，随时随地用md写一篇博，等到来到同步的机器才更新一次也让人很不爽。对于远程仓库只存放静态文件的方案，就想到了两个思路

###  同步静态文件
> 新仓库同步静态文件->hexo g生成静态文件->合并静态文件->hexo d部署

1. 在两地分别hexo init建立hexo博客目录，在_config.yml中都部署到同一个远程仓库，在每次generate时，都先将远程仓库的静态文件同步下来（新建一个单独的folder专门用于同步静态文件）。请勿使用hexo博客中的public文件夹用于同步，每次generate会覆盖同名文件。若另一台主机曾做过修改并提交，这次修改的静态文件将被本地未修改的静态文件覆盖。具体操作：
第一次克隆远程hexo主分支：

```
git clone git@github.com:user-name/user-name.github.io.git
```

之后每次hexo仓库发布首先在静态文件仓库获取远程仓库最新静态文件(无条件merge)

```
git pull origin master
```

2. 在静态文件同步仓库中，将自上次部署后，本地修改过的静态文件**删掉**（不记得怪我咯┑(￣Д ￣)┍）；同时在hexo blog仓库中``hexo -g``生成本地静态文件。
3. 将剔除过期静态文件的public仓库中文件复制到hexo的public中。
4. 使用``hexo -d``部署
> 以上步骤2中的删除操作是为了保证静态文件的数据都是最新的，保证博客内容的一致性。

###  同步源文件（大招）
想到上述方式需要进行2步骤的主要原因就是本地的源文件不同步，只能通过生成的静态文件进行合并的方式，为毛不将源文件同步呢？！参考[这篇文章](http://crazymilk.github.io/2015/12/28/GitHub-Pages-Hexo%E6%90%AD%E5%BB%BA%E5%8D%9A%E5%AE%A2/)的思路，同步源文件就是通过远程仓库对源文件融合统一。

##  呵呵哒

本文为了完成多客户端的同Hexo博的管理和发布，对ssh、git安装、Hexo安装、配置插件主题管理、域名绑定**不涉及**。
+ 系统不限制 
+ 是否写过博部署过没差
+ 最好Hexo3.X,2.X没试过出事自负

#  真·多客户端

##  创建远程仓库
+ 对于没有进行创作的hexo博客，首先创建同名仓库，再创建一个hexo branch。此时俩分支都是空的  
+ 对于已经进行创作的hexo博客，直接创建一个hexo 分支。此时，hexo clone了master的静态文件。

##  设置主分支
将hexo branch设为主分支，用于同步源文件；master branch作为静态文件的存放。（由于github pages会检索master分支作为该域名下的部署分支，因此必须把master名字保留）

![](http://ooo.0o0.ooo/2016/04/27/5720a2a70a238.png)

##   同步远程仓库并初始化
在本地（包含第一次所有源文件的主机）新建一个文件夹（就算已经写过博客的也这么搞）同步远程仓库并初始化

```
git clone git@github.com:your-Id/your-Id.github.io.git    
```

此时输入``git branch``可以看到两个分支，红色分支为主分支。
若是非空白博客，同步后需要先删掉里面的静态文件.直接删光，把原博客的文件全部复制过来**（好像没有发现更换远程仓库主分支后本地同步更换的git bash。求解?）**;若是空白博客，那么需要再令建一个文件夹，（因为hexo init操作会覆盖原.git文件导致与远程仓库失联）在该文件夹git bash执行

```
npm install -g hexo
hexo init
npm install
hexo g
npm install hexo-deployer-git--save
```

好了，里面该有的差不多都有了。全部复制到源文件仓库。**注意把hello worl删掉啊。**关键的就是，config.yml里deployer的branch是master，而本地仓库关联的是hexo就对了。
然后执行

```
git add .
git commit -m "init"
git push -u origin hexo
```
值得注意的是，有的时候提示主题的文件夹并未加入同步操作，显示

```shell

Changes not staged for commit
```
即主题文件夹的内容尚未有之前版本，应该加入tracked。应在提交前输入``git add themes/themes_name/``.最后那个斜杠别忘了加。
这就完成源文件的同步了！(￣▽￣)"" 执行``hexo d``操作将本地public静态文件部署。

#  发博日常

##   更博
如果你不能确定是否抽风需要经常换客户端，因此需要每次都要同步本地源文件；如果能忍住，，或者有规律的换客户端，那么平时更新不需要经常同步。  
写博前操作(**特么狠重要啊**)：

```
git pull origin hexo
```

更新操作：(同步源文件到hexo分支)

```
git add .
git commit -m "hehe"
git push origin hexo
```

这样每次更博都需要多几个步骤，嫌麻烦就要忍住不手痒了。在手机、pad写md再同步到电脑也是个不错的习惯。

##   换客户端
easy.只要远程仓库存在了一份完整的源文件，那同步就很容易了。似上步第一次同步，使用``git clone git@github.com:your-Id/your-Id.github.io.git``拷贝仓库。然后执行

```
npm install -g hexo
npm install
npm install hexo-deployer-git 
```

<font color="red">**不用hexo init!!!**</font>

##  后记

###  多地部署
pacman的公式和代码支持好差啊~还有需要多地备份的可以尝试coding（page之前免费的现在要钱了 = =）.在站点部署文件中更改deploy：

```markdown
deploy:
   type: git
   repo: 
      github: https://github.com/{YOUR_ID}/{YOUR_ID}.github.io.git,master
      coding: https://git.coding.net/{YOUR_ID}/{BLOG　NAME}.git,master
```
同理也能部署到自己的ftp服务器上。

###   <font color="red">bug1 file-name-too-long</font>  
在拷贝原始数据的时候，提交远程仓库时可能会报modules的文件夹无法同步。这个情况在windows下安装msysgit进行git bash会报这一问题。[这里](https://github.com/msysgit/git/pull/110)也提到了这个异常。但这并不是git的问题，而是msysgit的问题。使用其他git方式对文件路径较长的提交就不会报错。实测在github for windows下提交该modules文件就不会报错。

###  <font color="red">bug2 could not read Username for  'https://XXX'</font>  
当需要推送另一台设备的静态文件时，可能极少情况会碰上这个问题。有的说是bash的版本问题，不识别这种方式的推送。然而我用的同一版本的msysgit..

在[这里]()也发现了这个问题，尝试了一下使用在https中加入用户名和密码的方式能够良好的推送。。当然配置文件别泄露了。。更改配置文件的deploy如下：

```markdown
deploy:
   type: git
   repo: 
      github: https://{YOUR_ID}:{Your_Pwd}@github.com/{YOUR_ID}/{YOUR_ID}.github.io.git,master
      coding: https://{YOUR_ID}:{Your_Pwd}@git.coding.net/{YOUR_ID}/{BLOG　NAME}.git,master
```