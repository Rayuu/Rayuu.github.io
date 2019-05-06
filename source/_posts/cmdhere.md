---
title: windows右键添加cmd菜单
date: 2016-06-01 12:09:55
tags: windows
categories: 技术分享
---
windows右键添加cmd菜单

在windows右键添加cmd菜单，如下图。
原文地址：[http://blog.rayuu.com/cmdhere.html](http://blog.rayuu.com/cmdhere.html)
<!--more-->

![效果图](http://7xlqnm.com1.z0.glb.clouddn.com/2016/06/3150704417.jpg)

这样使用cmd进入某个路径就方便多了！！！！

# 步骤：
# 1、创建一个记事本文件，将以下文字复制进去，然后保存。
```bash
Windows Registry Editor Version 5.00 
[HKEY_CLASSES_ROOT\*\shell\cmdhere] 
@="Cmd&Here"
[HKEY_CLASSES_ROOT\*\shell\cmdhere\command] 
@="cmd.exe /c start cmd.exe /k pushd \"%l \\..\" "
[HKEY_CLASSES_ROOT\Folder\shell\cmdhere] 
@="Cmd&Here"
[HKEY_CLASSES_ROOT\Folder\shell\cmdhere\command] 
@="cmd.exe /c start cmd.exe /k pushd \"%l\" "
```
# 2、将记事本的文件的后缀 txt 改成 reg。

# 3、双击运行这个文件，提示添加注册表项，选择“是”。

# 4、然后再随便找个文件，点击右键，可以看见多了一个“cmdhere”的菜单，如上图所示！

点进去就是命令行了。