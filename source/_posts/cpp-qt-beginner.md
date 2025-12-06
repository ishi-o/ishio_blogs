---
title: "Qt: 基本用法"
date: 2024-06-27
categories: [Programming, C/Cpp, Qt]
tags: [beginner, Qt, GUI]
---
<!-- placeholder -->
<!-- more -->
# `Qt`基础

在`Qt5`后，`Qt`不断开发`QML`和`Qt Quick`，同时在抛弃传统的基于`C++`的`Qt Widgets`，致力于更快捷、直观地绘制应用

当然，`Qt`的优势在于极致的跨平台和前后端的顺畅结合，`Qt Widgets`和`Qt Quick`的设计思路是相似的；多是`QML/C++`混合开发，纯粹的`Qt Widgets`在前端开发上是较为低效的

本文并不想介绍`QML`，而是介绍`Qt Widgets`

## 元对象系统

### 简介

[元对象系统](https://www.cnblogs.com/WushiShengFei/p/9820835.html)(`Meta-Object System, MOS`)是`Qt`的核心机制，元对象即用于描述对象的对象；它提供以下功能：

- 信号-槽：允许对象间安全的通信，方便响应事件
- 反射系统：允许程序自己获取(运行时获取)对象的信息，在`java、dotNet`中也有类似的概念
- 动态属性：允许程序在运行时添加属性

所有继承`QObject`的类中，只要私有声明了宏`Q_OBJECT`，就可使用`MOS`

### 信号与槽

- 关键字：借助`MOC`(元对象编译器)，`Qt`为了实现信号槽机制，添加了`slots,signals`关键字(宏)

  - **信号和槽都是成员函数**，一般声明如下：

    ```c++
    class Class : public QObject {
        Q_OBJECT
    signals:
        ...
    private slots:
        ...
    };
    ```

    `signals`宏定义为`public QT_ANNOTATE_ACCESS_SPECIFIER(qt_signal)`，一定是公有的

    `slots`则可以在任意域中，但一般是保护或私有的

  - 一般，每个`QObject`的子类中的信号都应该有对应的**用于发送**该信号的成员函数(`Qt`自带类中一般定义为公有槽函数)

  - 一般控件有继承自父类的`Qt`实现的信号-槽和发送-响应机制

- 自定义信号：

  - 无返回值，可以有参数
  - 只需声明，无需实现(`MOC`自动实现)

- 自定义槽：

  - 返回值及参数必须和相连接的信号一致
  - 一旦声明，必须实现

- 发送信号：

  - 通常在成员函数中，可以使用`emit`关键字发送**自定义**信号，同时传递参数：

    ```c++
    emit mySignal(...);
    ```

  - 需要主动发送信号时，通常调用封装好的发送函数，而不是直接使用`emit`

    ```c++
    // 官方命名习惯(例如按钮类):
    public slots:
     click(); // 发送信号.
    signals:
     clicked(); // 信号.
    ```

- 连接方式：

  - 手动连接：调用`connect()`，四个必须填写的参数依次为发送者指针、信号、接收者指针、槽

    有三种输入信号-槽参数的方式：

    - 旧版本：调用宏函数`SIGNAL()`和`SLOT()`，处理输入的函数名后返回特定字符串

      连接失败后无反馈，例：

      ```c++
      // 参数不用添加限定符.
      connect(ui->lineEdit, SIGNAL(textChanged(QString)),
              this, SLOT(doSomething(QString)));
      ```

    - 新版本：传递函数指针，好处是连接失败后会报错，如：

      ```c++
      connect(ui->lineEdit, &QLineEdit::textChanged,
              this, &MainWindow::doSomething);
      ```

    - `lambda`表达式(匿名函数)：好处是不用额外声明槽函数

      ```c++
      connect(ui->button, SIGNAL(textChanged(QString)), [this](const QString&) {
          ...
      });
      ```

  - 连接函数的第五个参数表示连接方式，默认为自动关联，常用的有：

    - `AutoConnection`：信号槽在同一线程则为`DirectionConnection`，否则为`QueuedConnection`
    - `DirectionConnection`：信号发射后直接调用槽，槽函数执行完毕后再执行`emit`语句后的代码
    - `QueuedConnection`：槽在接收者的线程中执行(若在不同线程，则发送线程继续进行)

  - 默认连接：`Qt`编译器会尝试让所有名为`on_objName_signalName()`的槽函数进行连接

    类似于调用：

    ```c++
    connect(&objName, &SenderClass::signalName,
            this, &thisClass::on_objName_signalName);
    ```

    注：`objName`命名不能太长，否则连接失败；连接失败后不会终止程序，但会打印错误信息

- 常用控件的信号：

  ```python
  QPushButton:    # 按钮
      clicked()      # 被点击
  QLineEdit:     # 行编辑器
      textChanged(const QString&)  # 文本被修改, 同时发送修改后的文本
      returnPressed()     # 焦点在控件上时, 按下回车
   cursorPositionChanged()   # 光标移动
      editingFinished()    # 失去焦点/焦点在该控件上时, 按下回车
      selectionChanged()    # 选中区域被修改
  QDialog:     # 对话框
      accepted()/rejected()   # 因用户确认/拒绝对话框而被关闭
      finished()      # 被关闭
  QStackedWidget/QToolBox: # 堆栈窗口(多窗口器)/工具箱
      currentChanged(int idx)   # 当前窗口/选项修改, 同时发送新窗口的索引
  ```

- 可能用到的控件的信号：

  ```python
  QSplitter:     # 窗口分割器
      splitterMoved(int pos, int idx) # 分割条被移动, 同时发送移动后的位置、被移动条的索引
  QAction/QToolButton:  # 动作/工具按钮
      triggered()      # 被触发
  QSpinBox/QDoubleSpinBox: # 整数旋钮/浮点数旋钮
      valueChanged(int/double val) # 值被修改, 同时发送新的值

### 事件系统

事件和信号槽的区别：

- 信号本身没有和底层硬件、操作系统交流的能力，部分信号还需要通过事件发出(如`clicked()`)
- 关注点：
  - 信号槽：发出、接收信号的**对象**
  - 事件处理：发生的是**哪种事件**，并不关注对象间的通信
- 来源：
  - 信号：程序里的函数，由对象发出
  - 事件：**系统的底层消息**
- 本质：
  - 信号槽：成员函数
  - 事件：对象

`Qt`的事件循环：

- 产生：应用启动(`QApplication::exec()`)后，`Qt`即开启事件循环，不断检测底层消息并将它们转换为事件
- 分发：事件通过`QApplication::notify()`传入到**事件队列**，按队列的顺序取出事件，调用事件分发函数`bool event(QEvent* event)`
- 处理：`event()`会分情况调用不同的事件处理函数
- 事件循环里，事件的产生和处理是异步的

事件也可手动发送，它们不经过事件循环

有些类(如按钮类)的处理函数已经封装好了，而有些类(如窗口类)的键鼠事件处理函数没有语句，因此有需要时，需要**重写**它们，一般有两种处理方式：

- 发送信号，将事件的处理解耦成多个槽函数，可扩展性强，便于跨对象、跨线程通信
- 直接在函数内部处理，可用于事件处理较为简单的情况

常见的事件处理函数：

```python
# 启用某种属性的函数,某些事件需要特定属性才能产生
setAttribute(Qt::WidgetAttribute, bool)

# 以下函数返回值为void
键盘:
    keyPressEvent(QKeyEvent* event)    # 按键按下
    keyReleaseEvent(QKeyEvent* event)   # 按键松开
鼠标:
    mousePressEvent(QMouseEvent* event)   # 鼠标按下
    mouseReleaseEvent(QMouseEvent* event)  # 鼠标松开
    mouseDoubleClickEvent(QMouseEvent* event) # 鼠标双击
 wheelEvent(QWheelEvent* event)    # 鼠标滚轮滑动
    enterEvent(QEvent* event)     # 鼠标移进
 leaveEvent(QEvent* event)     # 鼠标移出
    hoverMoveEvent(QHoverEvent* event)   # 鼠标悬停(需启用WA_Hover)
焦点:
    focusInEvent(QFocusEvent* event)   # 获得焦点
 focusOutEvent(QFocusEvent* event)   # 失去焦点
拖动:
    setAcceptDrops(bool)      # 设置控件能否拖动
    dragEnterEvent(QDragEnterEvent* event)  # 被拖动的物体进入可放置区域
    dragMoveEvent(QDragMoveEvent* event)  # 被拖动的物体移动
    dragLeaveEvent(QDragLeaveEvent* event)  # 被拖动的物体离开可放置区域
    dropEvent(QDropEvent* event)    # 放下
```

传入的事件本身是带有信息的，例如：

```python
QKeyEvent:
    key()    # 返回Qt::Key枚举值, 表示按下的是哪个键
QMouseEvent:
    pos()    # 返回QPoint, 表示鼠标相对于该控件的位置
    globalPos()   # 返回QPoint, 表示鼠标在屏幕上的位置
```

关于事件分发函数`event()`的重写：

```c++
// event()是事件分发函数, 是有返回值的(表示事件是否已处理)
// 重写event()的目的是实现 事件过滤 或 想在事件处理前执行一些代码 或 分发自定义事件
bool event(QEvent* event) override {
    switch (event->type()) {
        // 想处理的事件
        case QEvent::KeyPress: 
            ...
            keyPressEvent((QKeyEvent*)event);
            // 标记为已处理(Qt默认调用accepted, 可以在处理函数里手动调用ignore())
            if (((QKeyEvent *)event)->isAccepted())
                return true;  // 则不传递
            break;
        case 想忽略的事件: return true; // 虽然未处理, 但可以直接返回true表示已处理
    }
 return QWidget::event(event); // 未处理, 交给父类处理
}
```

## 项目初始化

通过项目初始化来介绍`Qt`在控件上的一些机制和默认

### `.ui`文件

在创建`Qt`项目时，应该自选生成`.ui`文件，能轻易地在`Qt Designer`中修改它，并定义窗口和控件对象、以及它们的初始化

- 该文件使用`xml`语言，在`Qt Designer`上修改比直接编辑快的多

- 其能在`C++`程序中起作用的原因是，`Qt`编译器会通过`uic`(`ui`编译器)将`name.ui`翻译成头文件`ui_name.h`，其中定义了类`Ui_name`：
  - 该类定义了所有在`QD`设计的、除窗口类外的控件
  
  - 定义了成员函数`setupUi(QMainWindow*)`，用于初始化各种控件
  
  - 自动生成的项目主窗口类中，默认定义`Ui_name`类的指针`ui`
  
    所有构造函数的第一行应调用**`ui->setupUi(this)`**，以确保能安全调用`ui`中的控件

### 主窗口类及内存机制

学习自动创建的主窗口头文件格式，便于学会自定义控件类

设创建项目时定义的主窗口类为`MainWindow`，则应有头文件定义：

```c++
class MainWindow : public QMainWindow {
    Q_OBJECT
public:
    MainWindow(QWidget* parent = nullptr);
    ~MainWindow();
};
```

- `Q_OBJECT`：`Qt`定义的宏，只有声明了它，才能使用元对象系统

  任何继承了`QObject`类的子类都应**私有声明**该宏

- 简单窗口类可直接继承自`QWidget`

  复杂窗口类(如包含菜单栏、对话框)应继承`QMainWindow`，才能正常显示

构造函数的实现至少是这样的：

```c++
MainWindow::MainWindow(QWidget* parent)
: QMainWindow(parent),
 ui(new Ui_MainWindow) {
    ui->setupUi(this);
}
```

作为参数传入的`parent`和`this`是有必要的，因为`Qt`有自己的一套**回收内存**的机制：

- 半自动回收：所有`QObject`类及其子类对象都定义了`parent`和链表(用于存储子控件的指针)

  习惯上，所有控件类对象都使用堆内存，并在构造时指定父控件

  - 当释放父控件的内存时，会先释放所有子控件的内存，因此只要记得释放根控件的内存，就不会出现内存泄露
  - 而释放子控件的内存后，它会从其父控件的链表中删去，因此不会出现重复释放内存的情况

  - 所有这些对象都有**`QObject::setParent()`**方法，用于设置父控件

- 手动释放：对于**父控件在不同线程**、或**非`QObject`系**的占用**堆内存**的对象，无法半自动回收，应在其所在线程里调用**`deleteLater()`**方法，调用后会在**线程结束时释放其内存**

### 外部资源管理

`Qt`中通过`.qrc`(`Qt Resource`)文件管理外部资源，该文件使用`xml`语言，在`QC`中能更直观地使用

```xml
<!DOCTYPE RCC>
<RCC>
 <qresource prefix="/Icons">
        <file>images/name.jpg</file>
    </qresource>
</RCC>
```

直接编码也不难理解，这些标签的含义是：

- `RCC`：其内容会被`rcc`(`Qt`的资源编译器)识别，并把资源编译到可执行文件里
- `qresource`：用于指定其所属文件的前缀，前缀默认为`"/"`
- `file`：其内容为资源的真正的路径，例如上述语句引入了处于`./images/name.jpg`路径(相对于该资源文件)的图片

`.qrc`文件通过独特的相对路径管理资源，即**冒号+前缀+文件路径**，例如通过`":/Icons/images/name.jpg"`访问上述语句中的文件

使用资源的方式：

```c++
setWindowIcon(QIcon(路径)) // 设置窗口图标，仅窗口类可使用
    
setIcon(QIcon(路径))   // 设置控件图标，仅支持图标的控件可使用
    
QFile qssFile(":/path/name.css"); // 引入外部样式表
qssFile.open(QFile::ReadOnly);
qApp->setStyleSheet(qssFile.readAll()); // 加载到整个应用程序(所有QWidget类对象都可加载)

QSound::play(":/path/name.wav"); // 播放wav音频文件(静态播放,只能在主线程中调用)
QSound* musicPlayer {new QSound(路径)}; // 加载进内存后播放
musicPlayer->play();

QMediaPlayer* musicPlayer {new QMediaPlayer}; // 可播放wav、mp3、mp4等
musicPlayer->setMedia(QUrl::fromLocalFile(路径));
musicPlayer->play();
```

## 样式

关于样式，推荐通过`QD`进行初始化，或一律通过外部样式表统一，而不是通过`C++`编码修改

如果有`CSS`基础，许多属性会很熟悉

### 控件

所有`QWidget`系对象都有(由以下类组合而成)：

- `font`：存储字体族、字体大小、粗体、斜体等(构造函数的参数就是这四个属性)

  对象可调用`setFont(QFont)`方法修改

- `sizePolicy`：存储大小和伸缩的策略

  - 伸展策略：实质是枚举`QSizePolicy::Policy`，可通过`setSizePolicy()`修改

    - 基于`sizeHint`有不同的策略，以下称其为默认值

    - `Fixed`：控件大小始终为默认值

    - `Minimun/Maximun`：控件的默认值为最小/最大尺寸，大小可以大于/小于默认值

      最小/最大尺寸可通过`setMinimunSize()/setMaximunSize()`设置

    - `Preferred`：控件大小是默认值，但可以放大/缩小

    - `Expanding`：控件可以随意放大/缩小

    - 当`Preferred`和`Expanding`的控件同时存在于布局中时，前者会把空间让给后者且尽量保持默认值

  - 伸展因子：一般通过布局来管理，伸展因子越大，拉伸控件时改变的空间越多

- `focusPolicy`：存储获取焦点的策略，实质是枚举`Qt::FocusPolicy`

  - `StrongFocus`：默认值，也最常用，可通过鼠标点击、`Tab`键获取焦点
  - `TabFocus/ClickFocus`：`Tab`键/鼠标点击获取焦点
  - `NoFocus`：不可通过键鼠获取焦点

- `geometry`：不重要，一般通过布局来控制它

### 布局

`Qt`提供以下几种布局管理器：

- `QHBoxLayout/QVBoxLayout`：水平/垂直布局
- `QGridLayout`：网格布局
- `QFormLayout`：表单布局

布局有助于控件随窗口拉伸而自动拉伸，以下操作可在`QD`快速实现：

- 窗口调用`setCentralWidget()`设置中心控件
- 该中心控件调用`setLayout()`设置布局

### 窗口

如要限制窗口的放大/缩小键，可通过`setWindowFlags()`设置，参数为枚举`Qt::WindowType`

- `WindowMaximunButtonHint/WindowMinimunButtonHint`：最大化/最小化
- `WindowCloseButtonHint`：关闭键

## 更多常用方法

```python
QWidget:
    resize()   # 重置大小
    setFocus()   # 主动吸引焦点
    setHidden()   # 设置是否隐藏控件
QMainWindow:
    setWindowTitle() # 设置标题
    show()    # 显示窗口
    close()    # 关闭窗口
QDoubleSpinBox:
    setDecimals()  # 设置浮点数精度(实质是字符串)
文本编辑/显示器:
    setText()   # 设置文本, 会解析HTML内容
    setPlainText()  # 设置纯文本
 text()    # (QLineEdit)获取文本
    toPlainText()  # (其它)获取文本
按钮类:
    setEnabled()  # 设置是否启用
布局管理器:
    addWidget()   # 添加控件
    columnCount()/rowCount() # 列/行数
QStackedWidget:
    setCurrentIndex() # 设置当前窗口
QString:
    number()   # 静态成员, 将整型/浮点型转化为QString
QMessageBox:
    information()  # 静态成员, 呼出简单的小窗口(可添加Ok/Cancel按钮并返回用户的选择)

qDebug() << Data  # 打印信息
```
