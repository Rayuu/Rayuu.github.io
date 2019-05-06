---
title: git基本命令推送到github
date: 2016-07-29 20:03:54
tags: git
categories: git
---

# 使用git上传下载github上的代码

> 1、安装git
> 2、在本地创建ssh key

<!--more-->

    ssh-keygen -t rsa -C "your_email@youremail.com"

比如 `ssh-keygen -t rsa -C "rayu@engineer.com"`

邮箱就是你的github邮箱。
之后会要求确认路径和输入密码，我们这使用默认的一路回车就行。
成功的话会在~/下生成.ssh文件夹，进去，打开id_rsa.pub，复制里面的key。
回到github，进入Account Settings，左边选择SSH Keys，Add SSH Key,title随便填，粘贴key。
添加完成。
> 3、连接到github

     ssh -T git@github.com

输入一个yes,会看到：`You’ve successfully authenticated, but GitHub does not provide shell access` 。这就表示已成功连上github。

> 4、设置常用的用户名和密码

    git config --global user.name "your name"
    git config --global user.email "your_email@youremail.com"

这里的用户名和邮箱分别是github的用户名和邮箱。

> 5、进入仓库


    git remote add origin git@github.com:yourName/yourRepo.git

 后面的yourName和yourRepo表示你再github的用户名和刚才新建的仓库，加完之后进入.git目录，打开config，这里会多出一个remote “origin”内容，这就是刚才添加的远程地址，也可以直接修改config来配置远程地址。

> 6、提交上传

接下来在本地仓库里添加一些文件，比如README，

	    git add README
		#git add .
		touch git.md
	    git commit -m "first commit"

上传到github：

       git push origin master

   `git push`命令会将本地仓库推送到远程服务器。
   `git pull`命令则相反。

使用`git status`可以查看文件的差别，
使用`git add `添加要commit的文件，
之后`git commit`提交本次修改，`git push`上传到github。
或者`git push origin master`

> 7、克隆仓库


找到想要克隆的仓库。


    git clone git@github.com:AloneMonkey/weekly.git

提交上传的命令见第六步。

# PS: 

### 一、hexo新建md文件的命令是：

    hexo new "git"

会自带md后缀，哈哈，然后`hexo d -g`

### 二、出现错误解决方法

    Please, commit your changes or stash them before you can merge.
    Aborting

`git pull`时出现上述情况，解决方法如下：

这是因为如果系统中有一些配置文件在服务器上做了配置修改,然后后续开发又新添加一些配置项的时候,
在发布这个配置文件的时候,会发生代码冲突:

如果希望保留生产服务器上所做的改动,仅仅并入新配置项, 处理方法如下:

    git stash 
    git pull 
    git stash pop

然后可以使用`git diff -w +文件名` 来确认代码自动合并的情况.

反过来,如果希望用代码库中的文件完全覆盖本地工作版本. 方法如下:

    git reset --hard 
    git pull

其中git reset是针对版本,如果想针对文件回退本地修改,使用
 git checkout HEAD file/to/restore  

 进阶：

    git stash save "work in progress for foo feature"

当你多次使用’git stash’命令后，你的栈里将充满了未提交的代码，这时候你会对将哪个版本应用回来有些困惑，

’`git stash list`’ 命令可以将当前的Git栈信息打印出来，
你只需要将找到对应的版本号，
例如使用’git stash apply stash@{1}’
就可以将你指定版本号为stash@{1}的工作取出来，
当你将所有的栈都应用回来的时候，可以使用’`git stash clear`’来将栈清空。

转自：[http://blog.csdn.net/wh_19910525/article/details/7784901](http://blog.csdn.net/wh_19910525/article/details/7784901)