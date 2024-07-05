---
title: PySide6 笔记 - 布局
date: 2023-07-01 23:15:00 -0400
categories: [笔记, Python]
tags: [programming, python, gui, layout]
description: 本文简单介绍一点 PySide6 的界面布局
---

## 自己的 UI 类

```python
import sys
from PySide6 import QtWidgets

class UIMyWindow(object):
    def __init__(self, window: QtWidgets.QWidget):
        self.pbn_close = QtWidgets.QPushButton("Close", window)
        self.pbn_close.setGeometry(100, 100, 150, 50)

class MyWindow(QtWidgets.QWidget):
    def __init__(self, parent=None):
        super().__init__(parent)
        self.ui = UIMyWindow(self)
        self.resize(400, 300)
        self.ui.pbn_close.clicked.connect(self.close)

def main():
    app = QtWidgets.QApplication(sys.argv)
    win = MyWindow()
    win.show()
    sys.exit(app.exec())
```

因为 UI 类中需要在 `self` 下创建各种控件，就直接在初始化函数中创建了，而不再单独用一个 `setupUi` 函数。这样在窗口类中直接用 `self.ui = UIMyWindow(self)` 就可以初始化 UI 了。之后所有的控件都可以用 `self.ui` 获取。

## 主窗口的菜单、工具栏和状态栏

```python
import sys
from PySide6 import QtWidgets, QtCore, QtGui

class UIMyWindow(object):
    def __init__(self, window: QtWidgets.QMainWindow):
        # 状态栏直接获取
        self.status_bar = window.statusBar()
        # 菜单栏直接获取
        self.menu_bar = window.menuBar()
        # 创建和设定中央控件
        self.center_widget = QtWidgets.QWidget(window)
        window.setCentralWidget(self.center_widget)
        # 给中央控件指定一个布局
        self.vly_cw = QtWidgets.QVBoxLayout(self.center_widget)

        # 创建动作
        self.act_new = QtGui.QAction("New", window)
        self.act_new.setShortcut(QtGui.QKeySequence.StandardKey.New)
        self.act_new.setStatusTip("Create a new file")
        self.act_open = QtGui.QAction("Open", window)
        self.act_open.setShortcut(QtGui.QKeySequence.StandardKey.Open)
        self.act_open.setStatusTip("Open an existing file")

        # 添加菜单
        self.menu_file = self.menu_bar.addMenu("&File")
        self.menu_file.addAction(self.act_new)
        self.menu_file.addSeparator()
        self.menu_file.addAction(self.act_open)

        # 创建工具栏，可以多个
        self.tbar_file = QtWidgets.QToolBar(window)
        self.tbar_file.addAction(self.act_new)
        self.tbar_file.addAction(self.act_open)
        self.tbar_file.setAllowedAreas(QtCore.Qt.ToolBarArea.TopToolBarArea |
                                       QtCore.Qt.ToolBarArea.BottomToolBarArea)
        window.addToolBar(self.tbar_file)

        # 创建停靠窗（在中央控件周围的可拖动、可悬浮的窗口），可以多个
        self.dw_cont = QtWidgets.QDockWidget("Table of Contents", window)
        self.dw_cont.setAllowedAreas(QtCore.Qt.DockWidgetArea.LeftDockWidgetArea |
                                     QtCore.Qt.DockWidgetArea.RightDockWidgetArea)
        window.addDockWidget(QtCore.Qt.DockWidgetArea.LeftDockWidgetArea, self.dw_cont)
        # 给停靠窗设定控件
        self.lw_heading = QtWidgets.QListWidget(self.dw_cont)
        self.lw_heading.addItem("First")
        self.lw_heading.addItem("Second")
        self.dw_cont.setWidget(self.lw_heading)

        # 给中央控件添加控件，布局会自动将该控件设为中央控件的子控件
        self.txe_1 = QtWidgets.QTextEdit()
        self.vly_cw.addWidget(self.txe_1)

class MyWindow(QtWidgets.QMainWindow):
    def __init__(self, parent=None):
        super().__init__(parent)
        self.ui = UIMyWindow(self)
        self.resize(800, 600)

        self.ui.act_new.triggered.connect(self.act_new_triggered)
        self.ui.act_open.triggered.connect(self.act_open_triggered)

    def act_new_triggered(self):
        QtWidgets.QMessageBox.information(self, "Info", "New File")

    def act_open_triggered(self):
        QtWidgets.QMessageBox.information(self, "Info", "Open File")

def main():
    app = QtWidgets.QApplication(sys.argv)
    win = MyWindow()
    win.show()
    sys.exit(app.exec())
```

`QMainWindow` 本来就有一个 `QMenuBar` ，可以直接用 `menuBar()` 函数获取。 `QStatusBar` 也一样。

默认没有 `QToolBar` ，所以需要自己创建。

菜单栏和工具栏可以复用动作（`QAction`）。

对于 `addToolBar` 函数，

- 如果参数是一个字符串，则创建一个新的 `QToolBar` 对象并命名为该字符串，然后将该工具栏插入到主窗口的**上工具栏区域**。
- 如果参数是一个 `QToolBar` 对象，则把该工具栏插入到**上工具栏区域**。
- 如果参数是一个工具栏区域的枚举和一个 `QToolBar` 对象，则把该工具栏插入到该区域。

对于 `QDockWidget` 的 `setWidget` 函数：

- 如果在停靠窗可见的情况下添加控件，也就是在窗口运行中修改了控件，需要显式地调用 `show` 函数显示控件。
- 在设定控件之前必须给控件添加布局，否则控件将不可见。（这里可能是针对上一条的条件下说的吧，或者是针对多控件的情况下说的……）

## 关于布局

布局并非控件，但又与控件有交织，因此需要去理解一下。

首先先理解一下为什么使用 `QMainWindow` 的时候需要设定中央控件呢？（参考上例代码）

```python
...
class UIMyWindow(object):
    def __init__(self, window: QtWidgets.QMainWindow):
        ...
        # 创建和设定中央控件
        self.center_widget = QtWidgets.QWidget(window)
        window.setCentralWidget(self.center_widget)
        # 给中央控件指定一个布局
        self.vly_cw = QtWidgets.QVBoxLayout(self.center_widget)
...
```

因为初始状态下主窗口并不会自动创建中央控件（会自动创建的只有菜单栏和状态栏），所以需要我们手动创建一个然后指定，在此之后的所有控件都建立在这个中央控件之上，也就是中央控件是它们的父控件。

其实这个中央控件可以是任何 `QWidget` 的子类，但通常用 `QWidget` 就行了。这里可以把 `QWidget` 理解为一个什么特性也没有的最简单的控件，而不要理解为一个抽象基类。

所以上述 `self.center_widget = QtWidgets.QWidget(window)` 就是创建了一个 `QWidget` 控件并把它的父控件设定为主窗口。此时该控件在主窗口上还是游离的，在将该控件设定为中央控件之后，它就固定在中央了。

此后，这个中央控件需要一个布局，因此就添加一个布局。

给控件设定布局有两种方式，

1. 创建布局时直接传递控件作为参数。
2. 创建布局时不指定控件，传空参，后续用控件的 `setLayout` 函数指定布局。

在设定布局时，会自动将布局内的控件都设定为使用该布局的控件的子控件，不管它们之前的父控件是谁。

如果布局中有子布局，则子布局中控件的父控件还是使用布局的控件，只不过可以认为该父控件使用了多个布局，不同的子控件在不同的布局中。
