---
title: git的小tips
date: 2016-06-14 20:59:12
tags: git
---
Git是当下最流行最舒适的版本控制系统。这里大致把git容易忘记的事情做小结。

# Git Tips

---

## Git的机制
记录每次commit的快照及镜像，而非版本增量。完全的分布式及提倡多分支非线性开发。

- 类似CVS Subversion记录次提交时更新的文件及更新了哪些内容。
- Git则将快照记录在微型文件系统中，并使用指针(索引)指向当前快照以区别版本。

Git每次提交不必联网 大部分操作在本地完成 只在完成一部分功能或需要共享时提交远程仓库 节约了流量 提升了效率 无网络状态亦可提交更新及浏览历史。

## Git的四个状态
[Git的四个状态](!http://ooo.0o0.ooo/2016/06/14/57600bfe92c91.png) 
- 未追逐 新添加之前未有版本信息的文件 将未追踪的添加到版本管理使用**add**命令
- 已追踪有三类（已修改 未修改 已暂存）
- 未修改 一个时间点做了快照之后未修改的就是未修改 只要做出任意改动就会变成
- 已修改 此时若修正好 需要把文件添加到暂缓存区 同样使用**add** 命令
- 已暂存 此时暂时存储在内存中 需要把它永久的存在硬盘的git本地系统中 使用**commit**提交命令

使用下命令显示本目录下的文件跟踪情况

```shell
git status
```
使用下命令查看修改和比较两个版本差异

```shell
git diff
```

## 跟踪一个有趣的远程仓库
其实要做的只是 copy下来；过一段时间、知道它有更新的时候就pull一下，拉取下来

```shell
git clone git://github.com/[user_name]/[project_name].git
..
git pull origin project_name
## 或者
git fetch origin project_name
git merge
```

键入

```shell
git remote show origin
```
可以查看到以下几个关键的东西
- local branch pushed with 'git push' 缺省推送分支
- caching 那些远端分支还未同步到本地
- stale tracking branches那些分支已经在远端被删除
- Remote branch merged with 'git pull' while on branch  git pull时自动合并那些分支

## 小逼格
### 加标签

```shell
git tag [tag_name] #写入tag
git tag #显示所有tag
git show [tag_name] -lw #显示某个tag名字的提交
git tag -a [tag_name] [commit_code] #给忘记添加标记的提交起个标签
```
### 查看日志

```shell
git log 
e.g. 
git log --pretty=format:"%h - %an, %ar : %s" 
```
上式以一个很优美的显示方案：哈希字符串、对象、时间、提交的说明 一行一个提交显示。
| 选项        | 说明           |
| :-------------:|:-------------:|
| -p  |  按补丁格式显示每个更新之间的差异。| 
| --word-diff |  按 word diff 格式显示差异。|
|--stat  |显示每次更新的文件修改统计信息。|
|--shortstat    | 只显示 --stat 中最后的行数修改添加移除统计。|
|--name-only   |  仅在提交信息后显示已修改的文件清单。|
|--name-status  | 显示新增、修改、删除的文件清单。|
|--abbrev-commit  |   仅显示 SHA-1 的前几个字符，而非所有的 40 个字符。|
|--relative-date   |  使用较短的相对时间显示（比如，“2 weeks ago”）。|
|--graph  |   显示 ASCII 图形表示的分支合并历史。|
|--pretty  |  使用其他格式显示历史提交信息。可用的选项包括oneline，short，full，fuller 和 format（后跟指定格式）。|
|--oneline |  --pretty=oneline --abbrev-commit 的简化用法。|
| zebra stripes | are neat      |
## 自动补全-命令别名
直接看[这里](http://www.fenby.com/courses/sections/ji-qiao-he-qiao-men/)
