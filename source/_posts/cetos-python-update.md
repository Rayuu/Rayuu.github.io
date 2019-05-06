title: centos6.4从python2.6升级python2.7.9
categories: python
date: 2016-05-25
tags: python
---

#  原文地址： [http://blog.rayuu.com/86.html](http://blog.rayuu.com/86.html "centos6.4从python2.6升级python2.7.9")



centos6.4系统自带的python版本是2.6，用起来不是很方便。

于是今天我又开始折腾了。

首先看一下系统的版本

# lsb_release -a
发现是centos6.4

<!--more-->

再查看python版本

# python  -V    
Python 2.6.6  
下面开始升级，下载源码自己编译。。。。。

操作步骤如下：

1）安装devtoolset

yum groupinstall "Development tools"
2）安装编译Python需要的包包

yum install zlib-devel
yum install bzip2-devel
yum install openssl-devel
yum install ncurses-devel
yum install sqlite-devel
 

 

 

 

3）下载并解压Python 2.7.9的源代码

cd /opt
wget --no-check-certificate https://www.python.org/ftp/python/2.7.9/Python-2.7.9.tar.xz
tar xf Python-2.7.9.tar.xz
cd Python-2.7.9
4）编译与安装Python 2.7.9

./configure --prefix=/usr/local
make && make altinstall
5）将python命令指向Python 2.7.9

ln -s /usr/local/bin/python2.7 /usr/local/bin/python   //这句好像没什么用。。。
#mv /usr/bin/python /usr/bin/python2.6  
#ln -s /usr/local/bin/python2.7 /usr/bin/python  
6）重新检验Python 版本

#python -V  
7）解决系统 Python 软链接指向 Python2.7 版本后，因为yum是不兼容 Python 2.7的，所以yum不能正常工作，我们需要指定 yum 的Python版本

#vi /usr/bin/yum  
 

将文件头部的

#!/usr/bin/python

改成
#!/usr/bin/python2.6
保存并退出大功告成。。。。