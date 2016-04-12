---
layout:     post
title:      "Git教程"
subtitle:   ""
date:       2016-04-08
author:     "WunWun"
header-img: "img/git-lesson.jpg"
tags:
    - Git
    - 教程
---

## Git基本用法

#### 安装Git

Ubuntu上一个命令就够了

  sudo apt-get install git-core

Windows用户去[这里](http://msysgit.github.io/)下载.

#### 配置Git

  git config --global user.name "James"

  git config --global user.email "james@gmail.com"

#### 创建代码仓库

先创建目录，然后在这个目录下面输入如下命令：

  git init

仓库创建完成后，会在目录下生成一个隐藏的.git文件夹，这个文件夹就是用来记录本地所有的 Git 操作的（要不然，你以为Git怎么会知道你所有的操作）。

#### 提交本地代码

代码仓库建立完之后就可以提交代码了，其实提交代码的方法也非常简单，只需要使用add 和 commit 命令就可以了。 add 是用于把想要提交的代码先添加进来，而 commit 则是真正地去执行提交操作。

  git add test.txt // add 是用于把想要提交的代码先添加进来
  git add .        // 表示添加所有的文件

  git commit -m "First commit."  //在 commit 命令的后面我们一定要通过-m 参数来加上提交的描述信息。

## Git进阶

#### 查看修改内容

查看文件修改情况

  git status

查看文件的更改内容

  git diff //查看到所有文件的更改内容

  git diff test.txt  //查看 test.txt 这个文件的更改内容

#### 撤销未提交的修改

  git checkout test.txt  //这种撤销方式只适用于那些还没有执行过 add 命令的文件

已经执行过 add 命令的文件，应该先对其取消添加，然后才可以撤回提交。

  git reset HEAD test.txt
  
  git checkout test.txt 

#### 查看提交记录

  git log //查看历史提交信息

  git log 2e7c0547af28cc1e9f303a4a1126fddbb704281b -1 //在命令中指定该记录的 id，并加上-1 参数表示我们只想看到一行记录

  git log 2e7c0547af28cc1e9f303a4a1126fddbb704281b -1 –p //在命令中加入-p参数,查看这条提交记录具体修改了什么内容

## Git高阶用法

#### 分支的用法

  git branch –a //查看当前的版本库当中有哪些分支

  git branch newBranch //创建一个分支

  git checkout newBranch //切换到 newBranch 这个分支上

  git checkout master

  git merge newBranch //两行命令，就可以把在 newBranch 分支上修改并提交的内容合并到master 分支上了。

  git branch -D newBranch //删除分支

#### 与远程版本库协作

  git clone https://github.com/exmaple/test.git //将代码下载到本地

  git push origin master //把本地修改的内容同步到远程版本库上

  git fetch origin master //将远程版本库上的代码同步到本地，不过同步下来的代码并不会合并到任何分支上去，而是会存放在到一个 origin/master 分支上

  git merge origin/master //merge 命令将 origin/master 分支上的修改合并到主分支上

  git pull origin master //pull 命令则是相当于将 fetch 和 merge 这两个命令放在一起执行了，它可以从远程版本库上获取最新的代码并且合并到本地









