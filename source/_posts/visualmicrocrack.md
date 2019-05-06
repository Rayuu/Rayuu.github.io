---
title: 最新版visual micro 破解
date: 2017-08-08 21:33:30
tags: arduino
---

安装完成visual micro 会有一个7天的试用期。
在网上找了一个dll爆破的帖子，借来参考一下。



> 参考链接：http://www.52pojie.cn/forum.php?mod=viewthread&tid=460921  visual micro爆破

在安装目录下搜索 

`Visual.Micro.Processing.Sketch.dll`

![1.png](https://img.rayu.me/2017/08/4144619126.png)

把它复制出来先备份。

然后 relfector 打开这个dll，

## **记住是在原目录下打开哦！**

![2.png](https://img.rayu.me/2017/08/3080921199.png)


Visual.Micro.Utils.LicenseShared  下有一个 ProductActivated(String) : Boolean 
选中，然后再打开“tools”，选中reflexil v2.1具体版本无所谓

插件链接分享 

链接：http://pan.baidu.com/s/1cAjM5w 密码：v2hk

这是reflexil 2.1插件，点击工具tools addins就可以添加了。

![3.png](https://img.rayu.me/2017/08/3459621069.png)


添加完成后如下图；

![4.png](https://img.rayu.me/2017/08/360295244.png)


右下角会加载出下图所示：

![5.png](https://img.rayu.me/2017/08/1581867039.png)

在上图空白处点右键，选中replace all with code,把activationmanager里面改为

**return true**

![6.png](https://img.rayu.me/2017/08/747689246.png)

最后左下角点击compile编译，再点击ok.并保存

![7.png](https://img.rayu.me/2017/08/4290066799.png)

就会出现一个 

`Visual.Micro.Processing.Sketch.Patched.dll`

把这个dll在原目录下替换为原来的就可以了。 名称需要和原来的一样。这样就不会提示注册了。


