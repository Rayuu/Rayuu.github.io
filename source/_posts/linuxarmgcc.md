---
title: ubuntu 16.04下安装arm-linux-gcc
date: 2016-11-30 21:20:00
tags: linux
---

 ubuntu 16.04下安装arm-linux-gcc

最近在玩`micropython`，下载了`micropython`的源码后想进行编译玩一玩。

[https://github.com/micropython/micropython](https://github.com/micropython/micropython)

查看`readme.md`发现编译stm32的固件需要安装`arm-linux-gcc`等`gnu arm toolchain`交叉编译工具.

说干就干，还好电脑装了双系统，打开`ubuntu16.04`。

开始安装。

<!---more--->

### 1）、进入这个网站下载源码

[https://launchpad.net/gcc-arm-embedded](https://launchpad.net/gcc-arm-embedded)

右面有对应的版本下载，

下载完成后就是安装，查看readme.txt。

但是我的网速用浏览器下载太慢了。所以，我准备用apt-get 来安装。

[https://launchpadlibrarian.net/287100883/readme.txt](https://launchpadlibrarian.net/287100883/readme.txt)

### 2）、apt-get 安装gcc-arm-embedded

参考这篇文章 [http://jingpin.jikexueyuan.com/article/56406.html](http://jingpin.jikexueyuan.com/article/56406.html)

	sudo add-apt-repository ppa:terry.guo/gcc-arm-embedded
	 
	sudo apt-get update
	 
	sudo apt-get install gcc-arm-none-eabi

这样就安装好了，工具链路径在/usr/bin/目录下，具体可以用 ls | grep arm查看；

然后搞个链接，像这样

	cd /usr/bin
	 
	sudo ln arm-none-eabi-gcc arm-linux-gcc
	 
	sudo ln arm-none-eabi-ar  arm-linux-ar
	 
	.....

最后就安装完成了这个编译工具。

### 3）、开始编译micropython

	git clone https://github.com/micropython/micropython.git
	 
	cd stmhal/
	 
	make

还以为要编译好久，结果一会儿就编译完了。

在`stmhal/build-PYBV10`目录下的`firmware.hex` 和`firmware.dfu`的就是编译出来的固件咯！

接下来继续研究源码！！！

![](http://p1.bqimg.com/567571/aedc56efbecc1527.png)

