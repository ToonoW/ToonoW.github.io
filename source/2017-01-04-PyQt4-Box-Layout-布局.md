---
title: PyQt4 Box Layout 布局
layout: post
category: Python
---

PyQt4的布局方式有两种，绝对定位和使用布局类。

绝对定位的话就是按像素写死大小和位置，不能适应窗口的大小变化。而Box Layout就相当于不太精准的自动布局。

```python
class FrameLayout(QtGui.QWidget):

    def __init__(self):
        super(FrameLayout, self).__init__()

        self.initUI()

    def initUI(self):

        okButton = QtGui.QPushButton("OK")
        cancelButton = QtGui.QPushButton("Cancel")

        hbox = QtGui.QHBoxLayout()
        # 创建一个水平框布局，增加两个按钮，剩下空白部分的按比例分布
        hbox.addStretch(1)
        hbox.addWidget(okButton)
        hbox.addStretch(5)
        hbox.addWidget(cancelButton)

        vbox = QtGui.QVBoxLayout()
        vbox.addStretch(1)
        vbox.addLayout(hbox)
        # 将水平框布局加入到垂直框布局中
        vbox.addStretch(1)

        self.setLayout(vbox)
        # 使用设置好的布局

        self.setWindowTitle('box layout')
        self.resize(300, 150)

app = QtGui.QApplication(sys.argv)
ex = FrameLayout()
ex.show()
sys.exit(app.exec_())
```

addStretch，表示剩下的空白处按比例填充，所以按钮处显示效果是**1/6\*总空白长+ok+5/6\*总空白长+cancel**

