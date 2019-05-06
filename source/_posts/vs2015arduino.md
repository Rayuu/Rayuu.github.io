---
title: 使用VS2015 visual micro安装开发arduino
date: 2017-08-08 21:31:12
tags: arduino
---

使用vs2015可以更加高效的开发arduino，所以尝试安装了一下。

 -  打开VS2015->菜单栏工具->扩展和更新。


![1.png](https://img.rayu.me/2017/08/2361705456.png)

 - 搜索arduino

安装arduino IDE for visual studio.

![2.png](https://img.rayu.me/2017/08/4282969379.png)

- 重新打开VS2015

配置一下arduino ide的目录即可马上使用。

![3.png](https://img.rayu.me/2017/08/2235480008.png)

在菜单栏“文件”下面点“新建”，会出现一个arduino project选项。

![4.png](https://img.rayu.me/2017/08/1864746094.png)

然后输入工程名称即可新建工程。

![5.png](https://img.rayu.me/2017/08/1938886152.png)

- 配置

选板子型号和串口。

![6.png](https://img.rayu.me/2017/08/2479133433.png)

如果不想下载程序后调试，就可以关闭automatic debugging.
具体操作如下图：

![7.png](https://img.rayu.me/2017/08/4252945488.png)

在菜单栏“工具”->“选项”，也可以配置visual micro

![8.png](https://img.rayu.me/2017/08/4053589983.png)

至于这个debugger调试不是很方便，只能在loop那里断点，查看变量参数。

编译生成的HEX文件在debug或者release下面。

这个visual micro的破解方法见另一篇文章。
