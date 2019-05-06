date: 2015-03-22
title: chrome科学上网
categories: chrome
tags: chrome
---

*  `chrome`科学上网方式
* （仅供喜欢折腾的chrome用户使用，使用chrome也是一种信仰。）
* 由于xxx原因，无法上一些外国网站，比如说上google搜一些学术性的问题等，这是百度比不上的，现在百度搜索一些东西都是推广啊什么的，很恶心的。所以推荐如下科学上网方式。
* 工具： ``chrome浏览器+goagent+SwitchySharp``


<!--more-->


   用了几年的chrome了，chrome体验还是很不错的。<br>
   下面是我从chrome吧里面找的一些资源。<br>

* chrome下载地址 (http://pan.baidu.com/share/link?uk=3508695471&shareid=2338828618)

* 然后你可以先替换后斯特斯，这样可以保证你先能进入应用商店  
(http://pan.baidu.com/share/link?uk=3508695471&shareid=2772930274)  解压密码chrome<br>
* 把上面的hosts解压出来，windows用户找到C:\WINDOWS\system32\drivers\etc下的hosts，替换掉原来的。<br>如果有杀毒提醒，不用管，点信任即可。<br>
* 其他操作系统的hosts位置如下：
* Android：/system/etc/hosts
* Linux及其他类Unix操作系统：/etc

* 替换完成后，我们发现可以上应用商店，和登录google账号了。
* 然后打开应用商店，搜索SwitchySharp，点击添加至chrome。然后在浏览器右上角会出现一个灰色的地球。
* 然后下载：switchyoptions.bak 链接：(http://pan.baidu.com/s/1sCw5g)
* 下面我们来配置，单击那个灰色的地球，点“选项”，然后“导入导出”，接着“从文件恢复”，然后找到刚刚下载的备份文件导入即可。

* 最后就是配置goagent了。
* 下载链接: (http://pan.baidu.com/s/1mgmSwkg) 密码: 9kvz
* 解压后在goagent-goagent-b9ff04c文件夹下，打开local目录。然后配置proxy.ini
* 把里面的appid换成你的gae应用id即可。里面有readme.md，自己可以阅读。
* 最后用管理员运行goagent.exe即可成功运行。
* 如果ip还是被封了，那么你还可以下载里面的GoGo Tester2.3.9.exe，搜索ip，然后复制ip到proxy.ini，找到iplisit.  把ip地址放在google_cn=后面，还有google_hk=后面，粘贴2遍即可。
* 这样配置就完成了。可以轻松打开youtube和facebook等一些网站了。
* 最后，你还可以在右下角设置ie代理。这样ie浏览器同样也可以上了。
