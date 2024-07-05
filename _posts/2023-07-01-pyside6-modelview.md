---
title: PySide6 笔记 - 模型视图
date: 2023-07-01 23:28:00 -0400
categories: [笔记, Python]
tags: [programming, python, gui, modelview]
description: 本文简单介绍一点 PySide6 的模型视图
media_subpath: /assets/img/pyside6-modelview/
---

## 模型视图

### 介绍

我可能暂时也没完全理解模型视图相对于传统控件的本质区别。这里就简单说一下，**模型定义了操作数据的方式，视图连接到模型呈现出数据来**。视图不关心数据本身是什么结构，它只会跟模型要数据，然后按照自己的方式呈现出来。一个模型可以用多种视图呈现，这里的好处你在一个视图上修改了数据，其他视图会同步，因为它们用的是同一个模型。

就说这么多吧。

### 一些简单的例子

#### 只读表格

```python
import sys
from PySide6 import QtWidgets, QtCore, QtGui

class MyModel(QtCore.QAbstractTableModel):
    def __init__(self, parent=None):
        super().__init__(parent)

    def rowCount(self, parent: QtCore.QModelIndex | QtCore.QPersistentModelIndex = ...) -> int:
        return 2

    def columnCount(self, parent: QtCore.QModelIndex | QtCore.QPersistentModelIndex = ...) -> int:
        return 3

    def data(self, index: QtCore.QModelIndex | QtCore.QPersistentModelIndex, role: int = ...):
        if role == QtCore.Qt.ItemDataRole.DisplayRole:
            return f"Row{index.row() + 1}, Column{index.column() + 1}"

def main():
    app = QtWidgets.QApplication(sys.argv)
    tbv_main = QtWidgets.QTableView()
    model = MyModel()
    tbv_main.setModel(model)
    tbv_main.resize(400, 200)
    tbv_main.show()
    sys.exit(app.exec())
```

对于 `QAbstractTableModel` 来说，当它被连接到视图时，视图就会要两个东西：

- 要显示多少行多少列
- 每一个格子里要显示什么内容

所以 `rowCount` 和 `columnCount` 返回要显示多少行多少列， `data` 返回在特定索引上按照特定角色要显示的数据。

#### 不同的角色

```python
import sys
from PySide6 import QtWidgets, QtCore, QtGui

class MyModel(QtCore.QAbstractTableModel):
    def __init__(self, parent=None):
        super().__init__(parent)

    def rowCount(self, parent: QtCore.QModelIndex | QtCore.QPersistentModelIndex = ...) -> int:
        return 2

    def columnCount(self, parent: QtCore.QModelIndex | QtCore.QPersistentModelIndex = ...) -> int:
        return 3

    def data(self, index: QtCore.QModelIndex | QtCore.QPersistentModelIndex, role: int = ...):
        row = index.row()
        col = index.column()

        match role:
            case QtCore.Qt.ItemDataRole.DisplayRole:
                if row == 0 and col == 1:
                    return "<--left"
                if row == 1 and col == 1:
                    return "right-->"
                return f"Row{row + 1}, Column{col + 1}"
            case QtCore.Qt.ItemDataRole.FontRole:
                if row == 0 and col == 0:
                    bold_font = QtGui.QFont()
                    bold_font.setBold(True)
                    return bold_font
            case QtCore.Qt.ItemDataRole.BackgroundRole:
                if row == 1 and col == 2:
                    return QtGui.QBrush(QtCore.Qt.GlobalColor.red)
            case QtCore.Qt.ItemDataRole.TextAlignmentRole:
                if row == 1 and col == 1:
                    return int(QtCore.Qt.AlignmentFlag.AlignRight | QtCore.Qt.AlignmentFlag.AlignVCenter)
            case QtCore.Qt.ItemDataRole.CheckStateRole:
                if row == 1 and col == 0:
                    return QtCore.Qt.CheckState.Checked

def main():
    app = QtWidgets.QApplication(sys.argv)
    tbv_main = QtWidgets.QTableView()
    model = MyModel()
    tbv_main.setModel(model)
    tbv_main.resize(400, 200)
    tbv_main.show()
    sys.exit(app.exec())
```

在每次触发 `dataChanged` 的信号时，都会运行 `data` 函数 7 次（可能是因为有 7 个不同的角色吧），每次运行都会传递一个不同的 `role` 值给 `data` 函数。

对于不同的角色，返回的值类型也不一样， `DisplayRole` 通常就是要显示在视图上的文字内容，其它的基本都是不同的格式，比如字体、背景、对齐方式等。

#### 格子里的时钟

```python
import sys
from PySide6 import QtWidgets, QtCore, QtGui

class MyModel(QtCore.QAbstractTableModel):
    def __init__(self, parent=None):
        super().__init__(parent)
        self.timer = QtCore.QTimer(self)
        self.timer.setInterval(1000)
        # 每一秒都会发射一次 timeout 信号
        # 所以每一秒都会调用 timer_hint 函数一次
        self.timer.timeout.connect(self.timer_hint)
        self.timer.start()

    def timer_hint(self):
        top_left = self.createIndex(0, 0)
        # 每次调用该函数时，都会发射一个数据改变的信号
        # 这个信号描述了哪些范围里的数据改变了，以及什么角色改变了
        # 信号发出后模型会按照指定的范围和角色重新调用 data 函数，可能会调用多次
        self.dataChanged.emit(top_left, top_left, [QtCore.Qt.ItemDataRole.DisplayRole, ])

    def rowCount(self, parent: QtCore.QModelIndex | QtCore.QPersistentModelIndex = ...) -> int:
        return 2

    def columnCount(self, parent: QtCore.QModelIndex | QtCore.QPersistentModelIndex = ...) -> int:
        return 3

    def data(self, index: QtCore.QModelIndex | QtCore.QPersistentModelIndex, role: int = ...):
        row = index.row()
        col = index.column()
        if role == QtCore.Qt.ItemDataRole.DisplayRole and row == 0 and col == 0:
            return QtCore.QTime.currentTime().toString()

def main():
    app = QtWidgets.QApplication(sys.argv)
    tbv_main = QtWidgets.QTableView()
    model = MyModel()
    tbv_main.setModel(model)
    tbv_main.resize(400, 200)
    tbv_main.show()
    sys.exit(app.exec())
```

具体看注释。

#### 关于表头

```python
import sys
from PySide6 import QtWidgets, QtCore, QtGui

class MyModel(QtCore.QAbstractTableModel):
    def __init__(self, parent=None):
        super().__init__(parent)

    def rowCount(self, parent: QtCore.QModelIndex | QtCore.QPersistentModelIndex = ...) -> int:
        return 2

    def columnCount(self, parent: QtCore.QModelIndex | QtCore.QPersistentModelIndex = ...) -> int:
        return 3

    def data(self, index: QtCore.QModelIndex | QtCore.QPersistentModelIndex, role: int = ...):
        if role == QtCore.Qt.ItemDataRole.DisplayRole:
            return f"Row{index.row() + 1}, Column{index.column() + 1}"

    def headerData(self, section: int, orientation: QtCore.Qt.Orientation, role: int = ...):
        if role == QtCore.Qt.ItemDataRole.DisplayRole and orientation == QtCore.Qt.Orientation.Horizontal:
            match section:
                case 0:
                    return "first"
                case 1:
                    return "second"
                case 2:
                    return "third"

def main():
    app = QtWidgets.QApplication(sys.argv)
    tbv_main = QtWidgets.QTableView()
    model = MyModel()
    tbv_main.setModel(model)
    tbv_main.resize(400, 200)
    tbv_main.show()
    sys.exit(app.exec())
```

表头的数据是通过 `headerData` 函数定义的， `section` 其实就是表头索引， `orientation` 是行表头或者列表头。

#### 一个可以编辑的小例子

```python
import sys
from PySide6 import QtWidgets, QtCore, QtGui

COLS = 3
ROWS = 2

class MyModel(QtCore.QAbstractTableModel):
    # 定义一个信号
    edit_completed = QtCore.Signal(str)

    def __init__(self, parent=None):
        super().__init__(parent)
        self.grid_data = [[""] * COLS for _ in range(ROWS)]

    def rowCount(self, parent: QtCore.QModelIndex | QtCore.QPersistentModelIndex = ...) -> int:
        return ROWS

    def columnCount(self, parent: QtCore.QModelIndex | QtCore.QPersistentModelIndex = ...) -> int:
        return COLS

    def data(self, index: QtCore.QModelIndex | QtCore.QPersistentModelIndex, role: int = ...):
        if role == QtCore.Qt.ItemDataRole.DisplayRole and self.checkIndex(index):
            return self.grid_data[index.row()][index.column()]

    def setData(self, index: QtCore.QModelIndex | QtCore.QPersistentModelIndex, value, role: int = ...) -> bool:
        if role == QtCore.Qt.ItemDataRole.EditRole:
            if self.checkIndex(index) is False:
                return False
            self.grid_data[index.row()][index.column()] = str(value)
            result = []
            for row in range(ROWS):
                for col in range(COLS):
                    result.append(self.grid_data[row][col])
            # 每次修改数据都会发射这个信号
            self.edit_completed.emit(" ".join(result))
            return True
        return False

    def flags(self, index: QtCore.QModelIndex | QtCore.QPersistentModelIndex) -> QtCore.Qt.ItemFlag:
        return QtCore.Qt.ItemFlag.ItemIsEditable | super().flags(index)

class MainWindow(QtWidgets.QMainWindow):

    def __init__(self, parent=None):
        super().__init__(parent)
        self.tbv_main = QtWidgets.QTableView(self)
        self.setCentralWidget(self.tbv_main)
        model = MyModel(self)
        self.tbv_main.setModel(model)
        self.resize(400, 200)
        # 这个信号连接到设定窗口标题的函数上，所以每次修改数据都会改动标题
        model.edit_completed.connect(self.setWindowTitle)

def main():
    app = QtWidgets.QApplication(sys.argv)
    win = MainWindow()
    win.show()
    sys.exit(app.exec())
```

这里用 `self.grid_data` 模拟一个数据库， `setData` 函数中的 `value` 参数其实就是编辑后的数据，一般来说是字符串类型的。

`flags` 函数返回的数值表示可以如何操作这个格子，比如可选择，可编辑，可拖拽，可放置等。

因为这里我们修改的是文本数据，所以 `setData` 是在 `EditRole` 下修改数据的。如果修改的是复选框，可就得在 `CheckStateRole` 下修改了。

### 高级一点的话题

#### 树结构

```python
import sys
from typing import List
from PySide6 import QtWidgets, QtCore, QtGui

class MyModel(QtGui.QStandardItemModel):

    def flags(self, index: QtCore.QModelIndex | QtCore.QPersistentModelIndex) -> QtCore.Qt.ItemFlag:
        return ~QtCore.Qt.ItemFlag.ItemIsEditable & super().flags(index)

class MainWindow(QtWidgets.QMainWindow):

    def __init__(self, parent=None):
        super().__init__(parent)
        self.trv_main = QtWidgets.QTreeView(self)
        # self.model = QtGui.QStandardItemModel(self)
        self.model = MyModel(self)
        self.setCentralWidget(self.trv_main)

        prepared_row = self.prepare_row("first", "second", "third")
        item = self.model.invisibleRootItem()
        item.appendRow(prepared_row)
        second_row = self.prepare_row("111", "222", "333")
        prepared_row[0].appendRow(second_row)

        self.trv_main.setModel(self.model)
        self.trv_main.expandAll()

        self.resize(400, 200)

    @staticmethod
    def prepare_row(first: str, second: str, third: str) -> List[QtGui.QStandardItem]:
        return [QtGui.QStandardItem(first), QtGui.QStandardItem(second), QtGui.QStandardItem(third)]

def main():
    app = QtWidgets.QApplication(sys.argv)
    win = MainWindow()
    win.show()
    sys.exit(app.exec())
```

这里的模型其实就是一个基本的 `QStandardItemModel` 类，只不过稍微修改了一下禁止修改格子数据。这个稍后解释。

![three-models](three-models.png)

这张图描述三种视图在对待数据上的一些相似性。虽然它们呈现数据的方式不一样，但是我们可以这样理解：

1. 列表模型就是一个根节点下跟着一列节点，是一维的，比较好理解。
2. 表格模型是二维的，我们可以把列表模型上的每一个非根节点扩展成一个节点列表。也就是说如果对于列表模型的 `rootItem` 来说，每次都 `append` 一个 `item` ，那对于表格模型来说，就是每次对 `rootItem` 都 `append` 一行 `items` 。
3. 树模型就是在表格模型的基础上，把第一列的非根节点都可以看作根节点，在其下挂接一行行节点。此时操作第一列的非根节点就跟操作根节点一样。

所以这就是说看你怎么在模型中怎么处理数据了。

#### 题外话：关于枚举

对于可组合的枚举的定义方式：

```python
class Flag(object):
    NoFlag = 0x0
    Editable = 0x1
    Selectable = 0x2
    Droppable = 0x4
    Draggable = 0x8
    Enable = 0x10
    Visible = 0x20
    Resizable = 0x40
```

其实从二进制看就是这样：

```text
0000 0000 NoFlag
0000 0001 Editable
0000 0010 Selectable
0000 0100 Droppable
0000 1000 Draggable
0001 0000 Enable
0010 0000 Visible
0100 0000 Resizable
```

所以就是每个位都不重复，才能保证可组合，其实挺好理解的，比如如果 `Editable | Selectable` 就是 `0000 0011` 。

但如果本来一个值 `flag` 是 `Selectable | Droppable` ，也就是 `0000 0110` ，我想去掉 `Droppable` 怎么办？

可以这样： `~Droppable & flag` 。也就是对于想要去掉的枚举取反，然后“与”上原来的值。如果要去掉两个值，就连“与”两个取反的值就行。

这就是上一例中去掉 `QAbstractItemModel` 默认的可编辑格子的方式。

#### 可选择模型

```python
import sys
from PySide6 import QtWidgets, QtCore, QtGui

class MainWindow(QtWidgets.QMainWindow):

    def __init__(self, parent=None):
        super().__init__(parent)
        self.trv_main = QtWidgets.QTreeView(self)
        self.model = QtGui.QStandardItemModel(self)
        self.setCentralWidget(self.trv_main)

        root = self.model.invisibleRootItem()

        america_item = QtGui.QStandardItem("America")
        mexico_item = QtGui.QStandardItem("Mexico")
        usa_item = QtGui.QStandardItem("USA")
        boston_item = QtGui.QStandardItem("Boston")
        europe_item = QtGui.QStandardItem("Europe")
        italy_item = QtGui.QStandardItem("Italy")
        rome_item = QtGui.QStandardItem("Rome")
        verona_item = QtGui.QStandardItem("Verna")

        root.appendRow(america_item)
        root.appendRow(europe_item)
        america_item.appendRow(mexico_item)
        america_item.appendRow(usa_item)
        usa_item.appendRow(boston_item)
        europe_item.appendRow(italy_item)
        italy_item.appendRow(rome_item)
        italy_item.appendRow(verona_item)

        self.trv_main.setModel(self.model)
        self.trv_main.expandAll()

        selection_model = self.trv_main.selectionModel()
        selection_model.selectionChanged.connect(self.selectionChangedSlot)

    def selectionChangedSlot(self, new: QtCore.QItemSelection, old: QtCore.QItemSelection):
        index = self.trv_main.selectionModel().currentIndex()
        selected_text = str(index.data(QtCore.Qt.ItemDataRole.DisplayRole))
        hierarchy_level = 1
        seek_root = index
        while seek_root.parent().isValid():
            seek_root = seek_root.parent()
            hierarchy_level += 1
        show_string = f"{selected_text}, Level {hierarchy_level}"
        self.setWindowTitle(show_string)

def main():
    app = QtWidgets.QApplication(sys.argv)
    win = MainWindow()
    win.show()
    sys.exit(app.exec())
```

从视图上获取选择模型，然后把选择模型上的 `selectionChanged` 信号绑定到自定义的信号函数上，在信号函数中获取当前格子的内容，填到窗口标题上。

## Delegate 初步

其实 Delegate 是啥呢，就是当我们想要在视图上编辑一个数据时，跳出来的那东西。

比如说，如果我们想在表格视图上编辑一个格子里的文字，那跳出来的可能是一个文本输入框。如果我们想编辑的是一个日期，可能跳出来的是一个日历。这个就是 Delegate。

本质来说，你可以自己绘制跳出来的这个东西，或者，你也可以继承 `QStyledItemDelegate` 类来创建一个基于控件的 Delegate。这里我们只讨论后者。

```python
import sys
from PySide6 import QtWidgets, QtCore, QtGui

class SpinBoxDelegate(QtWidgets.QStyledItemDelegate):

    def createEditor(self, parent: QtWidgets.QWidget, option: QtWidgets.QStyleOptionViewItem,
                     index: QtCore.QModelIndex | QtCore.QPersistentModelIndex) -> QtWidgets.QWidget:
        editor = QtWidgets.QSpinBox(parent)
        editor.setFrame(False)
        editor.setMinimum(0)
        editor.setMaximum(100)
        return editor

    def setEditorData(self, editor: QtWidgets.QWidget, index: QtCore.QModelIndex | QtCore.QPersistentModelIndex):
        value = int(index.model().data(index, QtCore.Qt.ItemDataRole.EditRole))
        if isinstance(editor, QtWidgets.QSpinBox):
            editor.setValue(value)

    def setModelData(self, editor: QtWidgets.QWidget, model: QtCore.QAbstractItemModel,
                     index: QtCore.QModelIndex | QtCore.QPersistentModelIndex):
        if isinstance(editor, QtWidgets.QSpinBox):
            editor.interpretText()
            value = editor.value()
            model.setData(index, value, QtCore.Qt.ItemDataRole.EditRole)

    def updateEditorGeometry(self, editor: QtWidgets.QWidget, option: QtWidgets.QStyleOptionViewItem,
                             index: QtCore.QModelIndex | QtCore.QPersistentModelIndex):
        editor.setGeometry(option.rect)

class MyWindow(QtWidgets.QWidget):

    def __init__(self, parent=None):
        super().__init__(parent)
        self.resize(800, 400)

        self.vly_1 = QtWidgets.QVBoxLayout(self)
        self.tbv_1 = QtWidgets.QTableView()
        self.vly_1.addWidget(self.tbv_1)

        model = QtGui.QStandardItemModel(self)
        root = model.invisibleRootItem()
        for r in range(1, 5):
            items = [QtGui.QStandardItem(str(c * r)) for c in range(1, 3)]
            root.appendRow(items)

        self.tbv_1.setModel(model)
        delegate = SpinBoxDelegate(self)
        self.tbv_1.setItemDelegate(delegate)

def main():
    app = QtWidgets.QApplication(sys.argv)
    win = MyWindow()
    win.show()
    sys.exit(app.exec())
```

上述代码中，我们用 `QStandardItemModel` 创建了一个 4×2 的模型，用表格视图呈现出来，然后用视图的 `setItemDelegate` 函数设定我们自己的 Delegate 。

在这个 Delegate 中，我们首先要提供一个编辑器，也就是我们要编辑数据的那个控件，这里我们重写 `createEditor` 函数。上述例子中我们只创建了一个数字调整框，但在某些情况下，你可能会需要根据索引的不同创建不同类型的控件。

然后我们需要重写一个从模型中拷贝数据到编辑器的函数，也就是 `setEditorData` 函数。我们根据索引找到数据，然后赋值到控件上。

之后我们还要重写一个把编辑器中的数据赋值到模型中的函数，也就是 `setModelData` 函数，当我们触发某种完成编辑的信号后就会调用这个函数。

还有，控件的形状大小我们也得关心一下，一般我们希望它与格子大小一致就行，所以在 `updateEditorGeometry` 函数中，直接赋值了 `option.rect` 。
