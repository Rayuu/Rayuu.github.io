---
title: 浏览器首页被hao.qquu8.com劫持
date: 2017-03-09 19:10:55
tags: chrome主页劫持
---

昨天帮一个同学找了一个小马激活工具，今天早上电脑一开机，打开浏览器就发现首页变为了

http://hao.qquu8.com/ 然后跳转到hao123。

<!---more--->

心想还是要支持正版系统啊！

我首先是改浏览器的配置，把chrome的首页设置为打开新的标签页，结果操作无效。

紧接着我分别删除了桌面和任务栏chrome的快捷方式，属性快捷方式目标后面跟的http://hao.qquu8.com/?m=yx&r=j这个链接。

`"C:\Program Files (x86)\Google\Chrome\Application\chrome.exe"`只保留这个，还有任务栏的地址在


    C:\Users\rayu\AppData\Roaming\Microsoft\Internet Explorer\Quick Launch

    C:\Users\rayu\AppData\Roaming\Microsoft\Internet Explorer\Quick Launch\User Pinned\TaskBar

这两个地方的http://hao.qquu8.com/也删除了。

![a](http://blog.rayuu.com/usr/uploads/2017/03/1570470093.png)

这时候打开浏览器不会跳转到hao123了，我以为战斗结束了，也就没去管他了。

过了几个小时，我把浏览器关掉重新打开，发现又出现了hao123首页。

这我就不淡定了，强迫症又犯了。

开始网上找解决方案。

参考文章：[http://xinghao.me/2016/03/01/2016-03-01-kill-hao123/](http://xinghao.me/2016/03/01/2016-03-01-kill-hao123/)

我其实就是按照上面链接里的方法来做的。

不过还是要自己记录一下是怎么做的。

### 1、用WMI Event Viewer查看WMI事件。下载地址：[http://down.cncrk.com:8080/soft/keygen/WMITools.zip](http://down.cncrk.com:8080/soft/keygen/WMITools.zip)

### 2、用管理员打开wbemeventviewer.exe，在root\CIMV2里面选择_EventConsumer -> ActiveScriptEventConsumer

![2](http://7xlqnm.com1.z0.glb.clouddn.com/2017/03/1824951337.png)

双击ActiveScriptEventConsumer.Name=”VBScriptKids_consumer”，弹出属性页面：

![3](http://7xlqnm.com1.z0.glb.clouddn.com/2017/03/3738714703.png)

![4](http://7xlqnm.com1.z0.glb.clouddn.com/2017/03/991659301.png)

 

在 ScriptText里面看见了hao.qquu8.com/?m=yx&r=j这一串代码。

然后把它删掉，或者直接在WMI event viewer中将“_EventFilter:Name="unown_filter"”项目右键删除！

这样问题就解决了。
