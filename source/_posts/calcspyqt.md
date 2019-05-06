---
title: 利用pyqt4编写cs计算工具
date: 2016-12-03 22:24:05
tags: python
---

利用pyqt4编写cs计算工具

之前用Python做了一个通道号查找的小程序，只不过没有做界面。

最近的一个项目要用到帧校验（CS），为了方便，写了一个计算的小程序。

<!--- more --->

该程序用了QT4来做界面，也算是我的第一个QT小程序。

现在来分享一下编写过程中的心得体会，以及遇到的坑。

Python的环境是2.7。


CS校验和的C代码如下：

```cpp

	/******************************************************************
	*校验码 将所有相加MOD 256
	*******************************************************************/
	unsigned char Get_csckNum( unsigned char *Ptr, unsigned char Len )
	{
		register unsigned char Num;
		Num = 0;
		while( Len > 0 )
		{
			Num += *Ptr;
			printf("%d--%d\n",Len,Num);
			Ptr++;
			Len--;
		}
		return Num;
	}
```

转为Python代码
```python

	def CalCS(self,inputstr):
        input = inputstr.replace(" ", "")
        content = []
        for i in range(0, len(input), 2):
            content.append(input[i:i + 2])
        print content
        int_list = [int(i, 16) for i in content]
        print int_list
        num = 0
        for i in range(0, len(int_list)):
            num = num + int(int_list[i])
        return hex(num % 256)
```

## 一、软件界面设计

我用的是QT Designer来进行界面的绘制，第一次用。但是感觉直接写代码来绘制图形界面提升的会更快。
首先打开QT designer，添加需要用到的小组件。
最终我的设计界面如下：

![](http://7xlqnm.com1.z0.glb.clouddn.com/2016/12/684297600.png)

界面绘制完成后，保存会生成一个后缀名为.ui的文件。

## 二、Python程序的编写
 
界面完成后，进行框架的导入。
开始编写程序。
```python
	from __future__ import division
	import sys
	from PyQt4 import QtCore, QtGui,uic
	 
	from uicode import *
	qtCreatorFile = "calc.ui" # Enter file here.
	Ui_MainWindow, QtBaseClass = uic.loadUiType(qtCreatorFile)
	 
	class MyApp(QtGui.QMainWindow, Ui_MainWindow):
	    def __init__(self):
	        QtGui.QMainWindow.__init__(self)
	        Ui_MainWindow.__init__(self)
	        self.setupUi(self)
	        self.calc_button.clicked.connect(self.CalculateTax)
	    def CalculateTax(self):
	        content = str(self.content_box.toPlainText())
	        total = self.CalCS(content)
	        total_string = "" + str(total)
	        self.results_window.setText(total_string)
	 
	    def CalCS(self,inputstr):
	        input = inputstr.replace(" ", "")
	        content = []
	        for i in range(0, len(input), 2):
	            content.append(input[i:i + 2])
	        # print content
	        int_list = [int(i, 16) for i in content]
	        # print int_list
	        num = 0
	        for i in range(0, len(int_list)):
	            num = num + int(int_list[i])
	        return hex(num % 256)
	 
	if __name__ == "__main__":
	    app = QtGui.QApplication(sys.argv)
	    window = MyApp()
	    window.show()
	    sys.exit(app.exec_())
```

至此，最简单的程序就编写完成了。

## 三、生成exe可执行程序
 
利用pyinstaller直接生成。
命令代码：

	pyinstaller -w -F main.py

具体见下面链接
[http://blog.rayuu.com/pyinstaller_py2exe_exe.html](http://blog.rayuu.com/pyinstaller_py2exe_exe.html)
但是在生成的过程中会报错，原因是.ui的界面代码没有打包到exe文件里面去。

所以不得不另辟蹊径。

在网上找了好久资料。最后的解决方法如下：

> 1、首先尝试在QT Designer下直接生成代码。

在菜单栏，窗体，查看代码下。

![](http://7xlqnm.com1.z0.glb.clouddn.com/2016/12/2989482191.png)

结果运行报错。

![](http://7xlqnm.com1.z0.glb.clouddn.com/2016/12/815996677.png)

可能实在安装过程中出了错误，安装位置好像不能有中文和空格。

接着尝试第二种方式。

> 2、利用uic路径下的pyuic.py生成代码

在生成ui文件后，利用\Python27\Lib\site-packages\PyQt4\uic下面的pyuic.py进行生成代码。

	pyuic4 -o ui_xxx.py xxx.ui

也可以用pycharm进行转换。

也可以写一个批处理文件进行快速转换，把下面的批处理文件放置在UIC目录下。

接着把需要转换的ui文件拖入到批处理文件打开，就可以转换成功了。

我的是在D盘目录下，所以里面的代码自行修改。

	@echo off
	@cd /d "%~dp0"
	pyuic4 %1 > %~n1.py


这样代码就转换成功了。

本次生成的界面代码文件名称为 `uicode.py`

下面修改代码
```python
	
	from __future__ import division
	import sys
	from PyQt4 import QtCore, QtGui,uic
	 
	# 从uicode.py里面导入界面代码
	from uicode import *
	 
	class MyApp(QtGui.QMainWindow):
	 
	    def __init__(self,parent=None):
	        QtGui.QWidget.__init__(self,parent)
	        # 重载
	        self.ui=Ui_MainWindow()
	        self.ui.setupUi(self)
	        self.ui.calc_button.clicked.connect(self.CalculateTax)
	 
	    def CalculateTax(self):
	        content = str(self.ui.content_box.toPlainText())
	        total = self.CalCS(content)
	        total_string = "" + str(total)
	        self.ui.results_window.setText(total_string)
	 
	    def CalCS(self,inputstr):
	        input = inputstr.replace(" ", "")
	        content = []
	        for i in range(0, len(input), 2):
	            content.append(input[i:i + 2])
	        # print content
	        int_list = [int(i, 16) for i in content]
	        # print int_list
	        num = 0
	        for i in range(0, len(int_list)):
	            num = num + int(int_list[i])
	        return hex(num % 256)
	 
	if __name__ == "__main__":
	    app = QtGui.QApplication(sys.argv)
	    window = MyApp()
	    window.show()
	    sys.exit(app.exec_())
```
最后重新打包生成exe文件就可以直接运行了。

只是打包的库比较多，所以最后的程序大小有10几兆。

最后：

另外，如果界面需要加载资源文件的话，还要进行另外的操作，把资源文件转为.qrc后缀的文件。

[http://www.cnblogs.com/dcb3688/p/4237121.html](http://www.cnblogs.com/dcb3688/p/4237121.html)

转换资源文件用的是`Pyqt`的`pyrcc4 `命令

	pyrcc4 qrcfile.qrc -o  pyfile.py


> 1.Pycharm集成pyrcc4

我们使用Pycharm来集成pyrcc4，这样更利于我们高效开发

首先在菜单里面找到 `File => settings => Tools => External Tools`   （外边工具设置）

选择添加Add 

Name 填写： Rcc2Py

Group： 自已任意填写，我填写的是PyQt4

下面的Options默认

在Tools settings 里面这样填写：

Program 就是你安装Pyqt4的路径

Parameters 是指转换的参数      $FileName$ -o $FileNameWithoutExtension$.py

Working directory 表示输出在当前的工作目录   $FileDir$

> 2.转换qrc为py

选择要转换的qrc文件，右键，找到group 为（PyQt4） 目录下的Rcc2Py 

![](http://7xlqnm.com1.z0.glb.clouddn.com/2016/12/3304270685.jpg)

转换完成后，同级目录下就多出一个与qrc文件同命名的py文件。

> 3、引用资源py文件

py文件生成好了如何来引用使用呢？

说对了，引用就这么简单

	import uicode

使用的时候 冒号 “   ： ”  加 图片的路径， 如：

	:/img/firefox.png

运行试试，发现图片不显示，为什么呢，因为qrc文件添加过程中，我加了一个 “前缀” prefix。  
所以，如果在qrc文件中不添加前缀 使用   `:/img/firefox.png  ` 是可以的，但添加了前缀生成的qrc文件 qresource标签会多一个属性

	qresource prefix="picture"

在这里，正确的使用是：

	:picture/img/firefox.png

附：`uicode.py`
```python

	# -*- coding: utf-8 -*-
	 
	# Form implementation generated from reading ui file 'C:\Users\jx007\Desktop\PyQt_first-master\calc.ui'
	#
	# Created by: PyQt4 UI code generator 4.11.4
	#
	# WARNING! All changes made in this file will be lost!
	 
	from PyQt4 import QtCore, QtGui
	# 导入资源
	import sources
	try:
	     _fromUtf8 = QtCore.QString.fromUtf8
	except AttributeError:
	     def _fromUtf8(s):
	          return s
	 
	try:
		_encoding = QtGui.QApplication.UnicodeUTF8
		def _translate(context, text, disambig):
			return QtGui.QApplication.translate(context, text, disambig, _encoding)
	except AttributeError:
	     def _translate(context, text, disambig):
	          return QtGui.QApplication.translate(context, text, disambig)
	 
	class Ui_MainWindow(object):
	     def setupUi(self, MainWindow):
	           MainWindow.setObjectName(_fromUtf8("MainWindow"))
				MainWindow.resize(700, 240)
				icon = QtGui.QIcon()
				# icon.addPixmap(QtGui.QPixmap(_fromUtf8("ico.png")), QtGui.QIcon.Normal, QtGui.QIcon.Off)
				icon.addPixmap(QtGui.QPixmap(_fromUtf8(":ico/ico.png")), QtGui.QIcon.Normal, QtGui.QIcon.Off)
				MainWindow.setWindowIcon(icon)
				self.centralwidget = QtGui.QWidget(MainWindow)
				self.centralwidget.setObjectName(_fromUtf8("centralwidget"))
				self.gridLayout = QtGui.QGridLayout(self.centralwidget)
				self.gridLayout.setObjectName(_fromUtf8("gridLayout"))
				self.label_3 = QtGui.QLabel(self.centralwidget)
				font = QtGui.QFont()
				font.setPointSize(20)
				font.setBold(True)
				font.setWeight(75)
				self.label_3.setFont(font)
				self.label_3.setObjectName(_fromUtf8("label_3"))
				self.gridLayout.addWidget(self.label_3, 0, 1, 1, 1)
				self.label = QtGui.QLabel(self.centralwidget)
				font = QtGui.QFont()
				font.setPointSize(10)
				font.setBold(True)
				font.setWeight(75)
				self.label.setFont(font)
				self.label.setObjectName(_fromUtf8("label"))
				self.gridLayout.addWidget(self.label, 1, 0, 1, 1)
				self.content_box = QtGui.QTextEdit(self.centralwidget)
				font = QtGui.QFont()
				font.setFamily(_fromUtf8("Times New Roman"))
				font.setPointSize(10)
				font.setBold(False)
				font.setWeight(50)
				self.content_box.setFont(font)
				self.content_box.setObjectName(_fromUtf8("content_box"))
				self.gridLayout.addWidget(self.content_box, 1, 1, 1, 1)
				self.calc_button = QtGui.QPushButton(self.centralwidget)
				self.calc_button.setObjectName(_fromUtf8("calc_button"))
				self.gridLayout.addWidget(self.calc_button, 1, 2, 1, 1)
				self.label_2 = QtGui.QLabel(self.centralwidget)
				font = QtGui.QFont()
				font.setPointSize(10)
				font.setBold(True)
				font.setWeight(75)
				self.label_2.setFont(font)
				self.label_2.setObjectName(_fromUtf8("label_2"))
				self.gridLayout.addWidget(self.label_2, 2, 0, 1, 1)
				self.results_window = QtGui.QTextEdit(self.centralwidget)
				font = QtGui.QFont()
				font.setFamily(_fromUtf8("Times New Roman"))
				font.setPointSize(14)
				font.setBold(False)
				font.setWeight(50)
				self.results_window.setFont(font)
				self.results_window.setObjectName(_fromUtf8("results_window"))
				self.gridLayout.addWidget(self.results_window, 2, 1, 1, 1)
				MainWindow.setCentralWidget(self.centralwidget)
				self.menubar = QtGui.QMenuBar(MainWindow)
				self.menubar.setGeometry(QtCore.QRect(0, 0, 700, 23))
				self.menubar.setObjectName(_fromUtf8("menubar"))
				MainWindow.setMenuBar(self.menubar)
				self.statusbar = QtGui.QStatusBar(MainWindow)
				self.statusbar.setObjectName(_fromUtf8("statusbar"))
				MainWindow.setStatusBar(self.statusbar)
	 
				self.retranslateUi(MainWindow)
				QtCore.QMetaObject.connectSlotsByName(MainWindow)
	 
	     def retranslateUi(self, MainWindow):
	          MainWindow.setWindowTitle(_translate("MainWindow", "CS Calculator By:Rayu 2016-11-24", None))
	          self.label_3.setText(_translate("MainWindow", "CS Calculator", None))
	          self.label.setText(_translate("MainWindow", "Text", None))
	          self.calc_button.setText(_translate("MainWindow", "Calculate", None))
	          self.label_2.setText(_translate("MainWindow", "Result", None))
	          self.results_window.setHtml(_translate("MainWindow", "<!DOCTYPE HTML PUBLIC \"-//W3C//DTD HTML 4.0//EN\" \"http://www.w3.org/TR/REC-html40/strict.dtd\">\n"
	"<html><head><meta name=\"qrichtext\" content=\"1\" /><style type=\"text/css\">\n"
	"p, li { white-space: pre-wrap; }\n"
	"</style></head><body style=\" font-family:\'Times New Roman\'; font-size:14pt; font-weight:400; font-style:normal;\">\n"
	"<p style=\" margin-top:0px; margin-bottom:0px; margin-left:0px; margin-right:0px; -qt-block-indent:0; text-indent:0px;\"><span style=\" font-family:\'SimSun\'; font-size:9pt;\">/******************************************************************</span></p>\n"
	"<p style=\" margin-top:0px; margin-bottom:0px; margin-left:0px; margin-right:0px; -qt-block-indent:0; text-indent:0px;\"><span style=\" font-family:\'SimSun\'; font-size:9pt;\">*校验码 将所有相加MOD 256 </span></p>\n"
	"<p style=\" margin-top:0px; margin-bottom:0px; margin-left:0px; margin-right:0px; -qt-block-indent:0; text-indent:0px;\"><span style=\" font-family:\'SimSun\'; font-size:9pt;\">*******************************************************************/</span></p>\n"
	"<p style=\" margin-top:0px; margin-bottom:0px; margin-left:0px; margin-right:0px; -qt-block-indent:0; text-indent:0px;\"><span style=\" font-family:\'SimSun\'; font-size:9pt;\">unsigned char Get_csckNum( unsigned char *Ptr, unsigned char Len )</span></p>\n"
	"<p style=\" margin-top:0px; margin-bottom:0px; margin-left:0px; margin-right:0px; -qt-block-indent:0; text-indent:0px;\"><span style=\" font-family:\'SimSun\'; font-size:9pt;\">{</span></p>\n"
	"<p style=\" margin-top:0px; margin-bottom:0px; margin-left:0px; margin-right:0px; -qt-block-indent:0; text-indent:0px;\"><span style=\" font-family:\'SimSun\'; font-size:9pt;\"> register unsigned char Num;</span></p>\n"
	"<p style=\" margin-top:0px; margin-bottom:0px; margin-left:0px; margin-right:0px; -qt-block-indent:0; text-indent:0px;\"><span style=\" font-family:\'SimSun\'; font-size:9pt;\"> </span></p>\n"
	"<p style=\" margin-top:0px; margin-bottom:0px; margin-left:0px; margin-right:0px; -qt-block-indent:0; text-indent:0px;\"><span style=\" font-family:\'SimSun\'; font-size:9pt;\"> Num = 0;</span></p>\n"
	"<p style=\" margin-top:0px; margin-bottom:0px; margin-left:0px; margin-right:0px; -qt-block-indent:0; text-indent:0px;\"><span style=\" font-family:\'SimSun\'; font-size:9pt;\"> while( Len > 0 )</span></p>\n"
	"<p style=\" margin-top:0px; margin-bottom:0px; margin-left:0px; margin-right:0px; -qt-block-indent:0; text-indent:0px;\"><span style=\" font-family:\'SimSun\'; font-size:9pt;\"> {</span></p>\n"
	"<p style=\" margin-top:0px; margin-bottom:0px; margin-left:0px; margin-right:0px; -qt-block-indent:0; text-indent:0px;\"><span style=\" font-family:\'SimSun\'; font-size:9pt;\"> Num += *Ptr;</span></p>\n"
	"<p style=\" margin-top:0px; margin-bottom:0px; margin-left:0px; margin-right:0px; -qt-block-indent:0; text-indent:0px;\"><span style=\" font-family:\'SimSun\'; font-size:9pt;\"> printf("%d--%d\\n",Len,Num);</span></p>\n"
	"<p style=\" margin-top:0px; margin-bottom:0px; margin-left:0px; margin-right:0px; -qt-block-indent:0; text-indent:0px;\"><span style=\" font-family:\'SimSun\'; font-size:9pt;\"> Ptr++;</span></p>\n"
	"<p style=\" margin-top:0px; margin-bottom:0px; margin-left:0px; margin-right:0px; -qt-block-indent:0; text-indent:0px;\"><span style=\" font-family:\'SimSun\'; font-size:9pt;\"> Len--;</span></p>\n"
	"<p style=\" margin-top:0px; margin-bottom:0px; margin-left:0px; margin-right:0px; -qt-block-indent:0; text-indent:0px;\"><span style=\" font-family:\'SimSun\'; font-size:9pt;\"> }</span></p>\n"
	"<p style=\" margin-top:0px; margin-bottom:0px; margin-left:0px; margin-right:0px; -qt-block-indent:0; text-indent:0px;\"><span style=\" font-family:\'SimSun\'; font-size:9pt;\"> return Num;</span></p>\n"
	"<p style=\" margin-top:0px; margin-bottom:0px; margin-left:0px; margin-right:0px; -qt-block-indent:0; text-indent:0px;\"><span style=\" font-family:\'SimSun\'; font-size:9pt;\">}</span></p></body></html>", None))
```