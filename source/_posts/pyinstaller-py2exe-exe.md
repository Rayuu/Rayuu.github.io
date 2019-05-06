---
title: pyinstaller和py2exe生成exe文件并添加版本信息和自定义图标
date: 2016-09-29 20:45:42
tags: python
categories: python
---

# pyinstaller和py2exe把Python脚本生成exe文件，
# 并添加版本信息和自定义图标。

写了一个查找产品通道号的小程序，目前还没进行异常处理。
以下是程序源码。

另外一个地址
[http://blog.rayuu.com/pyinstaller_py2exe_exe.html](http://blog.rayuu.com/pyinstaller_py2exe_exe.html "http://blog.rayuu.com/pyinstaller_py2exe_exe.html")

<!--more-->

```

	# -*- coding:UTF-8 -*-
	import serial
	import time
	# import argparse
	import serial.tools.list_ports
	from time import sleep
	 
	 
	# 串口
	class Serial(object):
	    def __init__(self):
	        print u"+++++++++++++++++++++++++++++++++++++++++"
	        print u"+            通道号查找程序             +"
	        print u"+                V1.0                   +"
	        print u"+               by:Rayu                 +"
	        print u"+             2016.09.27                +"
	        print u"+++++++++++++++++++++++++++++++++++++++++"
	 
	    # 查找串口
	    def serial_find(self):
	        plist = list(serial.tools.list_ports.comports())
	        if len(plist) <= 0:
	            print u"串口未发现！"
	        else:
	            print u"找到的串口如下:"
	            for i in range(0, len(plist)):
	                print [i+1], plist[i]
	        print u"请输入对应的数字选择你想要使用的串口：<默认为1>"
	        number = raw_input()
	        if not number:
	            number = 1
	        number = int(number)
	        while number > len(plist):
	            print u"串口未找到，请重新输入。"
	            print u"请输入对应的数字选择你想要使用的串口："
	            number = raw_input()
	            number = int(number)
	        if number <= len(plist):
	            uart_choose = plist[number-1][0]
	        return uart_choose
	 
	    # 串口波特率
	    def serial_baudrate(self):
	        print u"请输入串口波特率：<默认115200>"
	        baudrate = raw_input()
	        if not baudrate:
	            baudrate = 115200
	        baudrate = int(baudrate)
	        return baudrate
	 
	    # 获取设备地址
	    def input_dev_addr(self):
	        print u'请输入设备地址：<如：dc00000233>'
	        data_dev = raw_input()
	        # print data_dev, type(data_dev)
	        return data_dev
	 
	    # 打开串口
	    def serial_open(self, com, baud):
	        try:
	            ser = serial.Serial(com, baud, timeout=0.1)
	        except Exception, e:
	            print u"打开串口失败，请检查串口是否被占用。\n5秒后自动退出程序"
	            time.sleep(5)
	            exit(1)
	        print u"串口", com, u"打开成功......波特率为：", baud
	        sleep(0.5)
	        return ser
	 
	    # 更改通道号
	    def serial_send_channel(self, com, baud, dev, sta):
	        # 
	        data_comm = "AB"
	        data_sum = "01"
	        # 通道号修改
	        ser = self.serial_open(com, baud)
	        start = time.clock()
	        for data_channel in range(0, 256):
	            # 转换为十六进制字符串
	            dev_channel = hex(data_channel)
	            # 把0x删掉
	            dev_channel = dev_channel[2:4]
	            if len(dev_channel) == 1:
	                dev_channel = '0' + dev_channel
	            # 连接字符串
	            content_addr = data_comm+dev_channel+data_sum
	            # print content_addr,"content_addr",type(content_addr)
	            # 十六进制发送
	            content_addr = content_addr.decode("hex")
	            # print content_addr
	            # print u"开始发送数据..."
	            ser.write(content_addr)
	 
	            lowerpower = int(sta)
	            # sta = self.serial_send_order(ser,DEV)
	            re_str = []
	            cou = 0
	            find_time = data_channel + 1
	            print u"进行第 %s" % find_time, u"次查找....."
	            i = 1
	            while i <= 10:
	                i += 1
	                for c in ser.read():
	                    # re_str += c
	                    re_str.append(c)
	                    cou += 1
	                    if cou == 10:
	                        # print re_str,"re_str"
	                        re_str = []
	                        break
	 
	            content_order = self.serial_send_order(dev)
	            if lowerpower == 1:
	                print u"低功耗设备，6秒发一次"
	                ser.write(content_order)
	                time.sleep(6)
	            else:
	                # time.sleep(0)
	                ser.write(content_order)
	 
	            line = []
	            cnt = 0
	            j = 1
	            while j <= 10:
	                j += 1
	                for d in ser.read():
	                    line.append(d)
	                    cnt += 1
	                    if cnt == 10:
	                        # print "line",line
	                        # line = []
	                        # print data_channel
	                        print u"查找结束......"
	                        end = time.clock()
	                        print u"用时: %f s" % (end-start)
	                        line[8] = ord(line[8])
	                        # print ord(line[8])
	                        ser.close()
	                        return line[8]
	        end = time.clock()
	        print u"用时: %f s" % (end - start)
	        print u"未找到通道号，请确定设备地址和工装是否正常......"
	        ser.close()
	        return -1
	 
	    # 发送通信命令
	    def serial_send_order(self, dev):
	        # 发送通信测试命令
	        data_head = "F01"
	        data_dev = dev
	        data_tail = "E3"
	        # print data_dev
	        data_addr = data_head + data_dev + data_tail
	        # data_addr = "F0"
	        data_addr = data_addr.decode("hex")
	        return data_addr
	 
	if __name__ == '__main__':
	    comm = Serial()
	    # 获取COM口
	    COMM = comm.serial_find()
	    # print COMM
	    BAUD = comm.serial_baudrate()
	    while True:
	        DEV = comm.input_dev_addr()
	        if DEV == 'q':
	            exit()
	        print u"低功耗设备请输入1,否则请直接按回车......\n输入q结束程序......"
	        sta = raw_input()
	        if not sta:
	            sta = 0
	        channel = comm.serial_send_channel(COMM, BAUD, DEV, sta)
	        if channel == -1:
	            print u"查找失败......请重新尝试......"
	        else:
	            channel = hex(channel)
	            print u"通道号为: ", channel
	 
	        # comm.serial_send_order(DEV)
	        # print BAUD
	        # comm.serial_open(COMM, BAUD)
	        # parser = argparse.ArgumentParser(description="通道号查找程序")
	        # parser.add_argument('--port', action='store', dest='port', type=int, required=True)
	        # given_args = parser.parse_args()
	        # port = given_args.port
	

```


## 程序编写完成后，生成exe可执行文件。


### 首先利用py2exe进行转换。


> 1).新建setup.py

```

	# -*- coding: utf-8 -*-
	__author__ = 'Rayu'
	from distutils.core import setup
	import py2exe
	includes = ["encodings", "encodings.*"]
	options = {"py2exe": {"compressed": 1, "optimize": 2, "includes": includes, "bundle_files": 1}}
	setup(
	version = "0.1.0",
	description = u"[利用工装查找设备通道号]",
	name = "FindChannel",
	options = options,
	zipfile = None,
	# 生成有指定图标的exe
	console = [{"script": "main.py",
	            "icon_resources": [(1, u"833.ico")]
	           }]
	# 生成无图标exe
	# windows = [{"script": "[源码文件名].py"}]
	)
	 
	# from distutils.core import setup
	# import py2exe
	#
	# setup(console=['main.py'])


```

然后运行程序  `python setup.py py2exe`

在64位系统下运行会报错： `bundle_files:1` 在64位操作系统下无效。 这句话的意思就是生成单文件程序。

所以64位操作系统下想生成单文件程序的可以改用`pyinstaller`.


> 2).利用`pip install PyInstaller`  或者去官网下载安装包。

我安装的`PyInstaller3.2`版本。

安装完成后，在命令窗口下执行：

    pyinstaller main.py

会在当前文件夹的`dist`目录下生成`main`文件夹，里面的`main.exe`就是生成的可执行文件。

把`main`文件夹整个复制出来就可以在其他地方运行了。

如果想为程序添加自定义图标和版本信息，那么在`main.py`的目录下，会有一个`main.spec`文件，使用`notepad++`打开进行编辑。

```

	# -*- mode: python -*-
	 
	block_cipher = None
	 
	 
	a = Analysis(['main.py'],
	             pathex=['C:\\Users\\jx007\\Desktop\\commserial'],
	             binaries=None,
	             datas=None,
	             hiddenimports=[],
	             hookspath=[],
	             runtime_hooks=[],
	             excludes=[],
	             win_no_prefer_redirects=False,
	             win_private_assemblies=False,
	             cipher=block_cipher)
	pyz = PYZ(a.pure, a.zipped_data,
	             cipher=block_cipher)
	exe = EXE(pyz,
	          a.scripts,
	          a.binaries,
	          a.zipfiles,
	          a.datas,
	          name='FindChannel',
	          version='version.txt',
	          debug=False,
	          strip=False,
	          upx=True,
	          console=True , icon='833.ico')


```

在`a.datas`, 下面添加

          version='version.txt',
          icon='833.ico',

然后保存。
 
先不要执行，然后把你心仪的`ico`图标放到和`main.py`同一目录下，`version`是版本信息路径。

名称为`version.txt`

这时候我们需要编写`version.txt`

下面是一个例子：

```

	# UTF-8
	#
	VSVersionInfo(
	  ffi=FixedFileInfo(
	    # filevers and prodvers should be always a tuple with four items: (1, 2, 3, 4)
	    # Set not needed items to zero 0.
	    filevers=(6, 1, 7600, 16385),
	    prodvers=(6, 1, 7600, 16385),
	    # Contains a bitmask that specifies the valid bits 'flags'r
	    mask=0x3f,
	    # Contains a bitmask that specifies the Boolean attributes of the file.
	    flags=0x0,
	    # The operating system for which this file was designed.
	    # 0x4 - NT and there is no need to change it.
	    OS=0x40004,
	    # The general type of file.
	    # 0x1 - the file is an application.
	    fileType=0x1,
	    # The function of the file.
	    # 0x0 - the function is not defined for this fileType
	    subtype=0x0,
	    # Creation date and time stamp.
	    date=(0, 0)
	    ),
	  kids=[
	    StringFileInfo(
	      [
	      StringTable(
	        u'040904B0',
	        [StringStruct(u'CompanyName', u'Microsoft Corporation'),
	        StringStruct(u'FileDescription', u'Windows Command Processor'),
	        StringStruct(u'FileVersion', u'6.1.7600.16385 (win7_rtm.090713-1255)'),
	        StringStruct(u'InternalName', u'cmd'),
	        StringStruct(u'LegalCopyright', u'© Microsoft Corporation. All rights reserved.'),
	        StringStruct(u'OriginalFilename', u'Cmd.Exe'),
	        StringStruct(u'ProductName', u'Microsoft® Windows® Operating System'),
	        StringStruct(u'ProductVersion', u'6.1.7600.16385')])
	      ]), 
	    VarFileInfo([VarStruct(u'Translation', [1033, 1200])])
	  ]
	)


```

如果你想要自己找这个文件的话，可以参考`pyinstaller3`的文档，里面写的很详细。

下面简单介绍一下如何获取这个文件。

首先在文档里面写到了，利用 `pyi-grab_version executable_with_version_resource` 这个命令来获取命令的版本信息。

我们运行 `pyi-grab_version c:/windows/system32/cmd.exe`

然后会把一个`file_version_info.txt`记事本文件保存在你cmd命令运行时候的路径下面。
找到并打开它，就是上面所列出的内容了。
然后根据自己的需要进行修改就可以了。
最后在`main.py`的路径下执行

    pyinstaller main.spec


在`dist`文件夹下就产生了一个`exe`文件，图标也是我们想要的那个图标了，至于程序的详细信息可以通过右键查看详细信息进行查看了。

![详细信息](http://7xlqnm.com1.z0.glb.clouddn.com/2016/09/1746137199.jpg)