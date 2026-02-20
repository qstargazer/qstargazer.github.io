---
title: "Git分支操作"
subtitle: ""
date: 2022-03-15T09:20:00+08:00
draft: false
author: "bugxch"
authorLink: "https://bugxch.github.io"
description: ""

tags: ["git"]
categories: ["技艺"]

image: ""
---

git的一些技术的分支操作笔记。

<!--more-->
![](https://cdn.pixabay.com/photo/2018/08/14/07/06/landscape-3604825_1280.jpg)

## 分支操作总览
下面的表格是git分支操作的总结表格

| 命令                                             | 说明                                                         |
| ------------------------------------------------ | ------------------------------------------------------------ |
| `git branch X`/`git checkout -b X`               | 第1个命令仅仅创建分支但不切换，第2个命令创建并切换到新分支   |
| `git branch`                                     | 查看当前所有分支列表                                         |
| `git log --oneline --decorate --graph --all`     | 列出当前所有分支及其提交的分支图                             |
| `git branch -v`                                  | 查看所有的分支及其最后一次提交                               |
| `git branch --merged`/`git branch --no-merged` | 查看已经合并或者没有合并的分支 |
| `git checkout X`                                 | 切换到分支X                                                  |
| `git branch -d X`/`git branch -D X`              | 删除分支X，强行删除分支X（即使暂存区有代码）                 |

## 分支工作流
参考[Git分支简介](https://git-scm.com/book/zh/v2/Git-%E5%88%86%E6%94%AF-%E5%88%86%E6%94%AF%E7%AE%80%E4%BB%8B)中的介绍，想象一种典型的工作流，看看如何使用git的分支管理进行操作。想象如下的一个典型开发场景：小王所在的团队基于master管理代码，小王参与了一个新特性的开发，需要基于master分支开发新的任务，与此同时他有时需要修复master分支上的一些小bug。现在小王需要完成如下的操作
1. 创建开发分支iss53，并在该分支上开发代码；
2. 紧急修复主线上的问题，创建hotfix分支，并在上面修复代码；
3. 将hotfix的修复代码合并到主线上，并上传到主线；
4. 切换回iss53分支，继续完成新功能开发，开发完成后将该分支的所有修改提交到master分支

### 新建分支
创建分支iss53，并在该分支上工作，你在该分支上做了部分提交和修改
```shell
git branch -b iss53 #创建并切换到iss53分支
vim test3.txt #修改文件
```
![](https://git-scm.com/book/en/v2/images/basic-branching-2.png)

### 切换分支
此时发现主干需要修改紧急bug，于是将iss53分支的修改提交之后切换到master
```shell
git commit -a -m "iss53 push" #在iss53上做提交
git checkout master #切换到master分支
git branch hotfix #基于master分支新建hotfix的分支
git checkout hotfix #切换到hotfix分支
```
需要注意，在切换到master分支之前需要将iss53分支上的修改提交，或者使用git stash命令将这些没有提交的还存在在暂存区的修改处理干净。继续在hotfix分支操作，基于这个分支修改代码，
```shell
vim test3.txt #修改代码
git commit -a -m "add a new modification on iss53 branch" #提交
```

![](https://git-scm.com/book/en/v2/images/basic-branching-4.png)

### 合并分支1
在hotfix上修复代码之后，需要将修复之后的代码合入到master分支，
```shell
git checkout master #在合并修复之前，需要切换到master分支
git merge hotfix #将hotfix的修复合入到master分支
```
![](https://git-scm.com/book/en/v2/images/basic-branching-5.png)

### 删除分支
如上图所示，此时hotfix分支和master分支的代码合并，hotfix的问题已经修复，所以需要删除hotfix的分支
```shell
git branch -d hotfix #删除hotfix分支
```
注意，如果hotfix的暂存区仍然有代码，可能使用上面的命令无法删除，需要使用下面的代码删除分支
```shell
git branch -D hotfix
```

### 继续开发
![](https://git-scm.com/book/en/v2/images/basic-branching-6.png)

删除hotfix分支之后，继续返回iss53分支开发，
```shell
git checkout iss53
vim test3.txt
git commit -a -m "develop push"
```
### 合并分支2
假设在C5这个快照的节点，iss53的开发任务完成，需要将开发分支的代码回合到master分支，操作与之前切换到master，然后合并，与之前的合并分支1的操作一样
```shell
git checkout master
git merge iss53
```
合并分支之前的图如下所示

![](https://git-scm.com/book/en/v2/images/basic-merging-1.png)

此次合并与之前的合并分支1不同，iss53当前节点的父节点向上追溯无法追溯到C4这个节点，所以git在此处做了一个**合并提交**，它新建了一个commit的节点C6，将C4和C5的快照进行比较之后合并到C6，合并后的节点如下图所示

![](https://git-scm.com/book/en/v2/images/basic-merging-2.png)

如果没有发生冲突，那么一切ok，删除iss53即可，任务完成。

### 合并分支之冲突解决
对于上面的合并分支2的情况，如果C4和后面的C5修改了同一个文件的同一行，那么会有冲突发生。以下为例，可以git接冲突的方法解决后合并。

![](https://pic.imgdb.cn/item/622eb66b5baa1a80ab9dd794.png)

冲突如下

![](https://pic.imgdb.cn/item/622eb67f5baa1a80ab9de38d.png)

其中上半部分是当前的master分支的情况，下面的iss53的情况的，可以手动解决冲突后继续合并

![](https://pic.imgdb.cn/item/622eb6f45baa1a80ab9e1d97.png)

可以看到新建了校验和为b700510的节点，master和iss53分支均合入到该节点。

---
全文完🚀