---
title: "Python-PIL库的使用"
description: 读书时的碎碎念
date: 2018-01-24T21:14:23+08:00
tags:
    - 开发
    - Python
    - pyqt5
Categories:
    - 开发
---

为了对付比赛，发现别人用qt写出来的应用挺好看的。昨天粗略看了一遍，记得不准确，今天重新开始看文档，一步步来吧。

# 运行一个最基本的窗口

```
from PyQt5 import QtWidgets 
# 导入PyQt5库的QtWidgets通用窗口类

class firstwindow(QtWidgets.QWidget):
#新建一个类，继承自QtWidgets.QWidget类方法
    def __init__(self):
        super(firstwindow,self).__init__()
        #这里要重载一下mywindows,同时也包含了QtWidgets.QWidget的预加载项

import sys
app = QtWidgets.QApplication(sys.argv)
# pyqt 窗口必须在QApplication方法中使用，否则会报错
# QWidget:Must construct a QApplication before a QWidget

windows = firstwindow()  #新建一个firstwindow对象，命名为windows

windows.show() #让窗口显示出来

sys.exit(app.exec_())  #启动事件循环，类似于Tkinter的mainloop()函数
```

# Qt Designer

因为QtDesigner很方便，可以帮助开发，于是我们接下来就用用看Designer，新建一个Widget然后保存，是ui文件，我们需要用一行代码把它转成py文件

在Python36/Lib/site-packages/PyQt5里打开cmd，

pyuic5 ui文件名 -o 目标py文件名

> pyuic5 d:/python/pyqt/widget.ui -o d:/python/pyqt/widget.py

转换后代码:

```
# -*- coding: utf-8 -*-

from PyQt5 import QtCore, QtGui, QtWidgets

class Ui_Form(object):
    def setupUi(self, Form):
        Form.setObjectName("Form")
        Form.resize(400, 300)

        self.retranslateUi(Form)
        QtCore.QMetaObject.connectSlotsByName(Form)

    def retranslateUi(self, Form):
        _translate = QtCore.QCoreApplication.translate
        Form.setWindowTitle(_translate("Form", "Form"))
```

它就是一个类，但我们依旧可以把它当作一个ui，要运行一个窗口。我们依旧是要用QtWidgets.QWidget类的对象的show方法。

所以代码这样写:

```
import sys
app = QtWidgets.QApplication(sys.argv)
window = QtWidgets.QWidget() #新建一个QtWidgets.QWidget类对象

ui = Ui_Form() # 把这个Ui_Form()对象实例化
ui.setupUi(window) #用这个ui的setupUi方法，参数是我们刚才新建的窗口window

window.show()
sys.exit(app.exec_())
```

其它都不变，因为ui带来的改变，只是增加了两行相应的代码而已。

如果想让逻辑和界面分离的话，可以尝试重新写一个py导入ui转换后的这个py文件。

```
#比如ui转换后保存为ui_window.py
from PyQt5 import QtWidgets
from ui_window import Ui_Form
import sys

class ui_window(QtWidgets.QWidget,Ui_Form): #多态继承
    def __init__(self):
        super(ui_window,self).__init__()
        self.setupUi(self)
if __name__ == '__main__':
    app = QtWidgets.QApplication(sys.argv)
    window = ui_window()
    window.show()
    sys.exit(app.exec_())
```

注：

感觉学了一点web前端以后再看类似应用界面ui设计的会轻松很多…

比如各种布局，边距什么的，应该会理解的更好。所以在Qt Designer里的设计和布局就按个人喜好来吧。哪个布局不懂多尝试就好了。接下来说的都是非ui相关的。

# 按键事件绑定

在designer里刚才的ui_window基础上，拖动一个PushButton到窗口里然后开启绑定模式(Edit Buddies)将这个PushButton绑定一个事件（比如点击后将整个窗口关闭），然后转换代码。

```
#在setupUi方法里增加了四行代码，前三行是添加这个button按钮
#着重看第四行，它是给button的clicked属性绑定了一个事件(Form.close)，Form.close也就是将窗口的关闭方法。
#pushButton除了clicked属性还是pressed属性，还有released属性等等。。，相同的Form的动作也不止一个关闭，具体在Edit Buddies模式里可以看到。

        self.pushButton = QtWidgets.QPushButton(Form)
        self.pushButton.setGeometry(QtCore.QRect(40, 60, 93, 28))
        self.pushButton.setObjectName("pushButton")
        self.pushButton.clicked.connect(Form.close)
        # self.按钮.点击.链接（窗口.关闭）

#在retranslateUi方法里增加了一行代码，因为我修改了一下Button实例的名字
        self.pushButton.setText(_translate("Form", "Button"))
```

所以要执行自己的函数，也就是要修改一下Form.close

比如，在之前为了逻辑与界面分离而写的一个runwindow脚本里这样修改(先把Form.close这一行代码去掉，也就是按钮点击以后不让它关闭窗口)。

在ui_window类里新建一个函数

```
....
    self.pushButton.clicked.connect(self.my_func)

def my_func(self): #因为是在类里所以需要有参数self
    print('按钮绑定函数已被执行！')
```

这样点击按钮就执行了一次print。

# 信号(signal)和槽(slot)

pyqt5里其实有一个信号和槽的概念。相关的概念其实还有更多，但我想先了解这两个应该就能满足很多需求了。

我说一下个人理解的，

信号：当你打开了这个应用exe以后做的一个动作（比如对一次按钮的点击，或者一次输入）当你做了这个动作以后，你就发射（emit）了一个信号。当然信号和槽都是可以自定义的，你也可以选择不发射信号。

槽：它可以是一个函数（一般我们也用它写函数），当我们做了一个动作发射了信号以后，肯定需要有东西接收啊，当然因为槽也可以自定义，所以也可以选择无视这些信号（但这样没有意义，因为这样不如不发射信号）。这个用来接收的东西就是槽。它和发射信号的函数事先绑定，从而一发射它就能接收到，然后执行这个槽函数。

其实，化简一下这两个概念，就能很好理解。 点击一次，传参（也可以没参数）给响应的函数然后执行这个函数。

而这个很简单的解释在pyqt里是需要用信号和槽来帮助编写的。我们来看一个基本的栗子。

```
#依旧是在原先runwindow脚本的基础上

from PyQt5 import QtWidgets,QtCore #因为要使用信号所以要导入QtCore类
from ui_window import Ui_Form
import sys,time

class ui_window(QtWidgets.QWidget,Ui_Form):
    signal1 = QtCore.pyqtSignal() #定义信号1，要发射多个信号可以多次定义
    def __init__(self):
        super(ui_window,self).__init__()
        self.setupUi(self)

        self.my_button.clicked.connect(self.emit_func)
        #将按钮绑定到emit_func（发射信号）这个函数上，也就是点击按钮就发射信号

        self.signal1.connect(self.response_func) #这行也很重要！这一行确定了信号发给哪个函数！

    def emit_func(self):
        print('发射信号！\n等待5s...')
        time.sleep(5)
        self.signal1.emit() #这一步是真正的发射信号

    def response_func(self):
        print('收到信号!执行响应函数！')

#剩下的和之前一样，让窗口显示出来的就不写了...
```

运行一下看效果，发现点击以后，首先执行和按钮的点击属性绑定的那个函数，而在这个函数里我们发射了信号1，然后因为这个信号1和槽函数绑定过，所以槽函数接收到了信号1，开始执行槽函数。

传参: 发射信号同时还想要传递参数的话，可以这样改.

```
.......
    signal = QtCore.pyqtSignal(str)  
    #定义信号，发射时带的参数为字符串型参数
    #这个参数类型可以是:str int list float object tuple dict

    self.signal.connect(self.response_func)

def response_func(self,param): #槽函数定义接收参数param
    print('接收参数',param)
```

> 一个动作可以发射多个信号，一个信号可以发给多个槽，一个槽也可以绑定多个信号。

# 消息框控件 QMessageBox

先看一个简单的例子,在runwindow基础上（接下来不具体说都是在runwindow基础上修改）

```
#只写增加的代码

from PyQt5.QtWidgets import QMessageBox #导入消息框类

......
    self.my_button.clicked.connect(self.show_msg)
    def show_msg(self):
        result = QMessageBox.information(self,('标题'),('显示信息'),QMessageBox.StandardButtons(QMessageBox.Yes|QMessageBox.No))
        # QMessageBox.information()显示消息框，其它参数不介绍了，最后一个参数QMessageBox.StandardButtons(),添加按钮，比如QMessageBox.Yes
```

QMessageBox的对话框只是图标不同，其它没有太大差别：

1. QMessageBox.information 信息框
2. QMessageBox.question 问答框
3. QMessageBox.warning 警告框
4. QMessageBox.critical 危险框
5. QMessageBox.about 关于

## 参数

> QMessageBox.XXX(QWidget对象,(‘标题’),(‘信息’),QMessageBox.StandardButtons(要显示的按钮))

> QMessageBox.about对于最后一个参数是没有的，其它相同

## 按钮种类:

- QMessageBox.About
- QMessageBox.Apply
- QMessageBox.Cancel
- QMessageBox.Close
- QMessageBox.Discard
- QMessageBox.Help
- QMessageBox.Ignore
- QMessageBox.No
- QMessageBox.NoToAll
- QMessageBox.Ok
- QMessageBox.Open
- QMessageBox.Reset
- QMessageBox.RestoreDefaults
- QMessageBox.Retry
- QMessageBox.Save
- QMessageBox.SaveAll
- QMessageBox.Yes
- QMessageBox.YesToAll

按自己需求来用就好，另外每个按钮都有一个返回值，

```
result = QMessageBox.information(...,QMessageBox.StandardButtons(QMessageBox.Yes))
print(result)
```

可以根据返回值来确认用户点击了哪个按钮。

# 标准输入框控件 QInputDialog

例子（designer里拖四个PushButton到Widget上）:

```
......
        self.button1.clicked.connect(self.input)

def input(self):
    result = QInputDialog.getText(self,('标题'),('提示'),QLienEdit.Normal,('默认文字'))
```

例子里是获取输入文本框，类似的还有三个:

1. QInputDialog.getInt(self,(‘标题’),(‘提示’),0,-65535,65536,1)

   > 0是默认数值，默认范围-65535~65536,1是步长

2. QInputDialog.getDouble(self,(‘标题’),(‘提示’),0,-65535,65536,10)

   > 其它同上，10是小数点后位数

3. item = [‘item1’,’item2’,’item3’]

   QInputDialog.getItem(self,(‘标题’),(‘提示’),item,1,True)

   > 这个可以当成有选择项的输入框，item是自定义列表，1是默认选中项目，True/False指 列表框是否可编辑

它们都是有返回值的，返回一个list,list[0]是输入的内容(字符串)，list[1]是个布尔值，代表是否输入

# 标准文件打开保存框 QFileDialog

先看例子吧

1. 打开文件框

   ```
   #一般都是和按键绑定的函数，所以接下来都只写一个函数
   from PyQt5.QtWidgets import QFileDialog
   
   def open_file(self):
       result = QFileDialog.getOpenFileName(self,('标题'),('d:/'),('jpg(*.jpg);;bmp(*.bmp)'),None)
   
   #d:/是默认搜索目录
   #打开单个文件。返回list，result[0]是文件路径，result[1]是过滤类型（字符串，和代码里的定义相同，比如: 'jpg(*.jpg)'）
   ```

   > 

   打开多个文件的话，改成getOpenFileNames就好了,区别是返回的list[0]是一个字符串列表（多个文件名）
   要过滤多个文件类型用两个分号 ;; 隔开类型。

2. 保存文件框

   ```
   def save_file(self):
       result = QFileDialog.getSaveFileName(self,('标题'),('d:/'),('*.exe;;*.*'),None,QFileDialog.Options(QFileDialog.DontConfirmOverwrite))
   
   #返回值是个list，list[0]是文件名，list[1]是文件类型('*.exe')
   ```

事实上，无论是打开文件框还是保存文件框，它都是只返回一个文件名和文件类型的list，并不会执行打开和保存操作，这是需要注意的…它只是提供一个图形化界面给用户不错的体验而已。实际打开或保存文件还是要我们通过获取到的文件名来用python写具体处理。

# 主窗口 MainWindow

它不同于Widget，包含了菜单栏，工具栏，任务栏等等,也是我们比较常见的应用的模板

在designer里重新创建一个新的文件 ui_mainwindow.ui 在里面创建一个mainwindow。

pyuic5转换成py文件以后，要显示这个窗口需要有一点变动。

```
#runmainwindow.py

from PyQt5 import QtWidgets
from PyQt5.QtWidgets import QMainWindow #从QtWidgets里导入QMainWindow类
from ui_mainwindow import Ui_MainWindow
import sys

class ui_mainwindow(QMainWindow,Ui_MainWindow): #继承QMainWindow类
    def __init__(self):
        super(ui_mainwindow,self).__init__()
        self.setupUi(self)

if __name__ == '__main__':
    app = QtWidgets.QApplication(sys.argv)
    window = ui_mainwindow()
    window.show()
    sys.exit(app.exec_())
```

代码变化不大

## 【菜单栏】

直接输入文字就可以了，还可以在Action Editor里编辑快捷键Short Cut。
但是要注意的是，它的PushButton是不同的，编写函数的代码也不同，直接看例子吧

```
#菜单项名称: Open_File，我们要为它编写一个函数

#......前面都是相同的代码
        self.actionOpen_File.triggered.connect(self.show_message) #绑定函数show_message
    def show_message(self):
        print('here is message')
#这样就为Open File这个菜单项绑定了一个函数，想怎么写这个函数就看个人吧
#比如Open File打开文件，我们可以打开一个打开文件框控件，诸如此类的...
```

可以看到，MainWindow里菜单项和Widget的button项的不同之处仅仅只有绑定函数的代码不一样而已。（triggered/clicked）

# 动态加载子窗口

我们在创建了一个主窗口MainWindow的基础上，想要增加一个功能，就是能够通过一次点击，打开或者隐藏一个子窗口（这个使用场景非常多），要做什么呢？

在designer里要做的事：
拖一个布局layout（比如栅格布局gridlayout）到主窗口里，命名为 main_grid
新建一个widget窗口，命名为ui_childwindow,为了结果明显，我们在childwindow里拖点东西进去，比如几个pushbutton，这些随意。
然后我们将主窗口和子窗口的ui文件转换成py，开始编写代码:

```
from PyQt5 import QtWidgets
from PyQt5.QtWidgets import QMainWindow
from ui_mainwindow import Ui_mainwindow
from ui_childwindow import Ui_childwindow
import sys

#子窗口类的编写，子窗口是widget所以继承QtWidgets.QWidget类
class ui_childwindow(QtWidgets.QWidget,Ui_childwindow):
    def __init__(self):
        super(ui_childwindow,self).__init__()
        self.setupUi(self)

class ui_mainwindow(QMainWindow,Ui_mainwindow):
    def __init__(self):
        super(ui_mainwindow,self).__init__()
        self.setupUi(self)
        self.childwindow = ui_childwindow() #实例化一个子窗口
        self.run()
    def run(self):
        self.actionOpen.triggered.connect(self.show_childwindow)
        #给菜单项按钮绑定一个show_childwindow函数，功能和它名字一样
        self.actionExit.triggered.connect(self.hide_childwindow)
        #给菜单项按钮绑定一个hide_childwindow函数，隐藏子窗口

    def show_childwindow(self):
        self.main_grid.addWidget(self.childwindow)
        #上面这一行是将实例好的子窗口添加到了主窗口的名为main_grid的布局里！

        self.childwindow.show() #显示子窗口

    def hide_childwindow(self):
        self.childwindow.hide() #隐藏子窗口

#显示主窗口的代码和之前的一样，可参照之前的代码
```

也就是说，动态加载子窗口是很容易的，只需要三行代码。

```
self.childwindow = ui_childwindow() #实例化一个子窗口

self.main_grid.addWidget(self.childwindow) #添加子窗口到主窗口的一个布局里

self.childwindow.show() #让子窗口显示出来
```

# 底端状态显示 statusbar

新建一个MainWindow以后就自带了一个statusbar属性，一般可以用它来显示一些状态

```
self.statusbar.showMessage('this is statusbar.')
```

使用很简单

# RadioButton 控件

单选按钮，如果在一个布局里放三个按钮，那同时只能一个被选中，这是自动实现的。

radiobutton的属性:

1. isChecked() 返回True/False,判断按钮是否被选中
2. setChecked(True) True/False设置按钮是否被选中

其它和普通PushButton基本无异，比如clicked属性，依旧可以用来connect自己的函数。

# CheckBox 控件

复选框按钮，可以复选，除此以外和RadioButton,PushButton无异。

# 单行文本输入框控件 QLineEdit

属性：

1. textChanged 很实用的一个属性，用来绑定文本状态change以后需要执行的函数
2. text() 返回文本内容，字符串类型
3. setText(‘文本’) 设置内容

> example

```
self.lineEdit.textChanged.connect(self.showDynamicText)
def showDynamicText(self):
    print('content:',self.lineEdit.text())
```

# combobox控件

1. currentText()

   ```
   self.combobox.currentText() #返回当前选项值(str)
   ```

1. currentTextChanged属性 如果用户选择了combobox中的其它选项，则会触发该事件。

   ```
   self.combobox.currentTextChanged.connect(self.my_func)
   ```

# listWidget控件

即列表框，我通常用它来显示一些结果。

大小还有其他一些什么设置都可以在qt designer里实现。

这里讲一个比较有用的需要代码实现的，就是自己给listWidget添加新的值

```
item = QtWidgets.QListWidgetItem() #实例化一个QListWidgetItem()对象
self.listWidget.addItem(item) #把实例好的这个对象添加进listWidget里
self.listWidget.item(0).setText('新添加值') #给新对象添加一个值。
```

1. len(self.listWidget)

   可以用来来获得它的长度，但它并不能用列表表示..

2. self.listWidget.row(item)

   可以获得一个item在列表里的行数(从0开始)

3. self.listWidget.removeItemWidget(item)

   移除一个item，无返回值（将item这个占用删除）

   删除listWidget里的一个item的值

   ```
   self.listWidget.removeItemWidget(self.listWidget.takeItem(len(self.listWidget)-1)) #删除listWidget最后一项item，并解除它的占用
   len(self.listWidget) #这时候再看list长度，发现的确减了1，同时显示的最后一项item也消失了。
   ```

对listWidget里的一个item赋鼠标操作，本来想这么写的…后来发现只需要一行代码就好了。

listwidget本身就有doubleClicked属性，connect一个函数就相当于给所有listWidget的item都赋了这么一个函数…很方便…

唉，我花了2个小时找文章..结果偶然在动作模式里发现pyQt5的这个功能…真是想吐血。

```
self.listWidget.doubleClicked.connect(self.your_function)
```

# 样式表 stylesheet

可以直接在designer里设置，比较方便，具体语法可以参照css

background-color:red #背景颜色:红

color:red #字体:红

不在designer里设置的话就是这样

```
self.setStyleSheet('QWidget{background-color:red}')
```

# 控件获取焦点

一般输入控件都是会有setFocus()属性，想让它获取焦点只要，x.setFocus()就好

# QPlainTextEdit

纯文本输入框控件

1. addPlainText

   可以从文本最后添加内容，每次add最后自动会有个换行

2. setText

   直接设置文本，原有文本会被覆盖

# some tips

(写在最后，也当作提醒自己一些地方要注意)

- 不管什么控件，要绑定一个函数，一般都是要这个控件的某一个动作后面跟.connect(要绑定的函数)，直接在动作里填函数是完全没效果的，可能会让你想不停地捶地。
- 如果要在一个窗口里，给它的一个比如按钮绑定一个事件，这个事件要弹出一个新窗口(非MessageBox)，有一种方法是新建一个窗口的ui，然后在要绑定的窗口的**init**方法里就要先生成这个窗口（创建这个新窗口的实例），绑定事件里再只要选择让这个窗口show()或者hide()就可以了。 否则会出现很多问题。