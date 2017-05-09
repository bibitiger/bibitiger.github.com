---
layout: post
title: "使用QPainter创建等待动画在PyQt中"
date: 2017-05-09 19:35:24 +0800
comments: true
categories: "Python"
---


##原文地址：[blog.bibitiger.cn/blog/2017/04/11/install-php71-mac/](http://blog.bibitiger.cn/blog/2017/05/09/make-loading-animation-by-qpainter/)

<br/>

---

具体的代码和使用可查看[github仓库](https://github.com/bibitiger/testWaitProgress)

实现旋转动画还可以使用QGraphicsItemAnimation来实现，这个以后再说，今天说说用QPainter实现这个效果。

<!--more-->


### 先看下效果

![gradient](http://7xtz1f.com2.z0.glb.clouddn.com/image/qtProgress/ProgressGradient.gif-shuiyinBlack)

首先解决动画的问题，QPainter是Qt的绘画板，经常和PainterEvent混合使用，在页面重新渲染或者调用update等的情况下都会触发PainterEvent。在这个前提下，先创建一个我们自己的QWidget，这里需要注意的是必须给我们的QWidget设定最小size，否则在layout布局的时候会显示不出来。重写QWidget的__init__,我是这样做的：

```
    def __init__(self, width, height, parent = None):
        super(MyRotateProgress, self).__init__(parent)
        self.setMinimumSize(width, height)
```

这一步作为我们先画出一个十个指针的圆形环绕图，这里我们重写paintEvent：

```
    def paintEvent(self, QPaintEvent):
        super(MyRotateProgress, self).paintEvent(QPaintEvent)
        painter = QPainter(self)
        # 反走样，边缘平滑
        painter.setRenderHint(QPainter.Antialiasing, True)
        #radius取最小边长
        radius = self.width() if self.width() < self.height() else self.height()
        if radius <= 0:
            return
        radius = radius/6
        # 将圆等分为10份，分别画出10条线
        per = 360/10;
        # 将画板的焦点坐标放置到中心，这样我们的旋转将围绕中心来进行
        painter.translate(self.width()/2, self.height()/2)
        #将一个一个的直线画出
        for i in range(10):
        	  #记住当前画板状态
            painter.save()
            #设置画笔，Qpen(黑色，宽度三像素)
            painter.setPen(QPen(Qt.black, 3))
            #旋转
            painter.rotate(per*i)
            #划线
            painter.drawLine(QPoint(0, 0 + radius), QPoint(0, 0 + radius*2))
            #由于我们将画板旋转了角度，所以还原画板到上一次记忆的状态
            painter.restore()
```

好，现在我们已经画出了一个雏形了，接下来让他动起来就可以出效果了。那接着使用QTimer来计时，并且如前文所说在update的时候会触发paintEvent这个事件。那如下，就可以实现旋转了：

```
from PyQt5.Qt import *

class MyRotateProgress(QWidget):
    PenStyleNormal = 0
    PenStyleGradient = 1
    PenstylePerGradient = 2

    def __init__(self, width, height, parent = None):
        super(MyRotateProgress, self).__init__(parent)
        self.setMinimumSize(width, height)
        self.now = 0;
        self._timer = QTimer(self)
        self._timer.timeout.connect(self.onTime)
        self._timer.start(150)
        self._PenStyle = MyRotateProgress.PenStyleNormal

    def onTime(self):
        self.now += 1
        self.now = 0 if self.now >= 10 else self.now
        self.update()

    def paintEvent(self, QPaintEvent):
        super(MyRotateProgress, self).paintEvent(QPaintEvent)
        painter = QPainter(self)
        # 反走样，边缘平滑
        painter.setRenderHint(QPainter.Antialiasing, True)
        #radius取最小边长
        radius = self.width() if self.width() < self.height() else self.height()
        if radius <= 0:
            return
        radius = radius/6
        # 将圆等分为10份，分别画出10条线
        per = 360/10;
        # painter.setPen(QPen(Qt.white, 3))
        painter.translate(self.width()/2, self.height()/2)
        # painter.rotate(per)
        # painter.drawLine(QPoint(0, 0 + radius), QPoint(0, 0 + radius*2))
        print('get pen begin')
        for i in range(10):
            painter.save()
            # self.getPenStyle(self._PenStyle, i)
            painter.setPen(self.getPenStyle(self._PenStyle, i))
            # painter.translate(self.width()/2, self.height()/2)
            painter.rotate(per*i)
            painter.drawLine(QPoint(0, 0 + radius), QPoint(0, 0 + radius*2))
            painter.restore()

    def getPenStyle(self, PenStyle, PenPoint):
        radius = self.width() if self.width() < self.height() else self.height()
        if radius <= 0:
            return
        radius = radius / 6

        if PenStyle == MyRotateProgress.PenStyleNormal:
            if PenPoint == self.now:
                return QPen(Qt.white, 3)
            else:
                return QPen(Qt.black, 3)
        elif PenStyle == MyRotateProgress.PenStyleGradient:
            point = PenPoint-self.now if PenPoint-self.now >= 0 else 10+PenPoint-self.now
            print('{}:{}'.format('self.now',self.now))
            print('{}:{}'.format('PenPoint', PenPoint))
            print(point)
            return QPen(QColor(255*point/10, 255*point/10,255*point/10), 3)
        elif PenStyle == MyRotateProgress.PenstylePerGradient:
            if PenPoint == self.now:
                gradient = QRadialGradient(0,radius, radius, 0, radius)
                gradient.setColorAt(0.2,Qt.white)
                gradient.setColorAt(0.6,Qt.gray)
                return QPen(gradient, 3)
            else:
                gradient = QRadialGradient(0,radius, radius, 0, radius)
                gradient.setColorAt(0.2,Qt.gray)
                gradient.setColorAt(0.6,Qt.black)
                return QPen(gradient, 3)

    def closeEvent(self, QCloseEvent):
        print('timer close')
        if self._timer.isActive():
            self._timer.stop()

    def hideEvent(self, QHideEvent):
        print('timer hide')
        if self._timer.isActive():
            self._timer.stop()

    def showEvent(self, QShowEvent):
        print('timer show')
        if self._timer.isActive() != True:
            self._timer.start()

    @property
    def PenStyle(self):
        return self._PenStyle

    @PenStyle.setter
    def PenStyle(self, Value):
        self._PenStyle = Value
        self.update()
```

当然我们用的使用不能就这样用，既然我们是等待和loading的动画，那模板和提示字是不可避免的，这时候我们就需要用到QDialog了，我们需要重写QDialog，并且把他的尺寸设定为目标窗口的大小，并且进行排版，废话不多说，直接上代码：

```
from PyQt5.Qt import *
import  MyRotateProgress

class MyWaitProgressDialog(QDialog):
    '''MyWaitProgressDialog(string,QWidget)'''

    def __init__(self, value, targetWindow):
        super(MyWaitProgressDialog, self).__init__()
        self.decTile = QLabel()
        #设置为居中
        self.decTile.setAlignment(Qt.AlignHCenter)
        self.Dec = value
        self._targetWindow = targetWindow
        self._rotate = MyRotateProgress.MyRotateProgress(80,80)
        self._rotate.PenStyle = MyRotateProgress.MyRotateProgress.PenStyleGradient
        # self._rotate.setParent(self)
        # palette = QPalette(Qt.gray)
        self._rotate.setAutoFillBackground(True)
        # self._rotate.setPalette(palette)
        vbox = QVBoxLayout()
        #设置为居中
        vbox.setAlignment(Qt.AlignHCenter)
        vbox.addStretch(1)
        vbox.addWidget(self._rotate)
        vbox.addWidget(self.decTile)
        vbox.addStretch(1)
        hbox = QHBoxLayout()
        hbox.addStretch(1)
        hbox.addLayout(vbox)
        hbox.addStretch(1)

        # self.webview.resizeEvent().connect
        self._oldTargetFlags = self._targetWindow.windowFlags()
        # self._targetWindow.setWindowFlags( self._oldTargetFlags & Qt.FramelessWindowHint)

        self.setLayout(hbox)
        #去掉标题栏
        self.setWindowFlags(Qt.Dialog | Qt.FramelessWindowHint)
        self.setGeometry(targetWindow.geometry())
        self.setWindowOpacity(0.8)

    @property
    def Dec(self):
        return self._dec

    @Dec.setter
    def Dec(self, value):
        assert isinstance(value, str), "webUrl value %r not match %s" (value, QUrl)
        self._dec = value
        self.decTile.setText(value)

    def closeEvent(self, QCloseEvent):
        print("closeEvernt")
        self._targetWindow.setWindowFlags(self._oldTargetFlags)

    def paintEvent(self, QPaintEvent):
        print('paint Event')
```

然后我们使用的时候用exec就是模态的了：

```
self.progress = MyWaitProgressDialog('begin...',self)
        self.progress.exec()
```