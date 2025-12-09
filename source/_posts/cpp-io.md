---
title: "Cpp: I/O"
date: 2024-01-19
categories: [Programming, C/Cpp]
tags: [quick start, I/O API]
---
<!-- placeholder -->
<!-- more -->
# `C++`风格的`I/O`

`I/O`是一门语言很重要的部分，`<iostream>`之所以含有相当一部分好用的函数，是因为在`I/O`实现时各个头文件互相包含，使得只需要包含`<iostream>`就能够使用很多的东西

但如果要用`STL`、实现文件交互等，`<iostream>`并没有包含它们，推荐使用万能头文件`<bits/stdc++.h>`，包含几乎所有会用到的东西

## `stdInput`

### `cin`

在`C++`中，`istream`类的对象`cin`(在`<iostream>`中声明)替代了`scanf`，它自动创建标准输入缓冲区，从标准输入流提取输入

这些对象被称为流对象，它将流和对象联系起来，一个非`iostream,fstream`类流对象只和一个缓冲区、两端(程序端、输出输入端)相关联

最简单的获取输入的代码如下，它是格式化流插入符(即会将读取的字符串转换成后接对象的相应类型，除了它，其它方法都是非格式化读取)，接受一个**引用**：

```c++
cin >> identifier;     // 有回显、有缓冲、忽略空格与换行,遇空格或回车停止
```

自然想到，如何读取空格，实现面向行的输入？`cin`有两种方法可以实现：

```c++
cin.getline(char*,std::streamsize);// 读取size-1个字符后或遇到换行符停止,且丢弃换行符
cin.get(char*,std::streamsize);  // 读取size-1个字符后或遇到换行符停止,将换行符保留在缓冲区
```

`std::streamsize`是`long int`的重命名，即有符号整型；它们的共同特性是都不会将换行符读入到字符串中，且可以添加第三个`char`型参数表示自定义终止字符

如果需要只读取缓冲区内的任意一字符，`cin`有重载的`get()`方法，其一的用法类似`getchar()`：

```c++
ch = cin.get();      // 读取一字符,返回它的字符编码;如遇到EOF,则返回eof(一个负数)
cin.get(char& ch);     // 将读取的字符赋值给ch;如遇到EOF,则返回0
```

它们常用于抛弃掉缓冲区的换行符，但一次调用只能丢弃一个字符，`cin`提供`ignore()`方法来一次性抛弃多个字符：

```c++
cin.ignore(std::streamsize n,delim='\n'); // 抛弃n个字符后或遇到delim并抛弃它后停止
```

`operator>>(),cin.getline(),cin.get(...),cin.get(char),cin.ignore()`都返回`cin`的引用，这意味着它们可以**链式调用**：

```c++
cin.getline(a,b).getline(a,b);  // 读取两行(或是读取一串较长行)
cin >> a >> b;
cin.get(ch).get();     // 连续读取两字符
(cin.get(ch)>>a).get();
cin.get().get(ch);     // 错误,get(void)返回的不是istream引用
```

如果非要给`cin`改名，可以选择创建引用`istream& in=cin`(一般不会直接使用`istream`或`ostream`的构造函数，只有在创建文件流对象的时候会间接调用`i(o)stream(sb&)`、或是想实现多缓冲区输入输出时会直接使用它，在这里不作讨论)

### `string`的输入

上述方法都不支持`string`类对象，然而显然`string`类更常用，于是`<string>`中有一个全局函数：

```c++
getline(istream&,string&,char='\n');
```

三个参数，第一个为自定义的输入流，通常是标准输入流`cin`，第三个参数为指定的结束符(默认为换行符)

有些编译器在使用`string`类后默认添加`<string>`，所以就算没有包含它也能运行，但`getline()`是`<string>`的函数，应该养成习惯事先包含它，以免出错

## `stdOutput`

### `cout`

与`cin`的大部分内容类似，`cout`是`ostream`类的对象，它与标准输出流相关联，格式化输出如下，`operator<<`接受**引用或常量**：

```c++
cout << identifier << constant;
```

由于`string`类定义的`operator<<()`能完整输出整个字符串，所以输出部分不像输入一样有专门的函数

对应`putchar()`，`cout`也有类似的方法：

```c++
cout.put(char)  // 输出单个字符
```

`operator<<(),cout.put(char)`都返回指向`cout`的引用，所以`cout`也可以链式输出：

```c++
(cout << str << endl).put(48)
```

标准输出缓冲区采用行缓冲，它遇到换行符或`eof`时将自动刷新缓冲区，此外还提供了两个控制符：

```c++
cout << flush << endl;  // flush直接刷新缓冲区,endl在缓冲区中插入换行符后再刷新它
```

### 精细控制输出

`C++`提供一种不逊于`printf()`那样精细控制输出位数、输出形式的，更简洁的输出方式，那就是许多控制符

除了刷新缓冲区的`flush`和`endl`(在`<ostream>`中)，其它常用的控制符如下：

- `hex,oct,dec`：将整型数据以十六、八、十进制输出，在`<bits/basic_ios.h>`中
- `setprecision(int)`：永久设置数据的精度(显示数据的总位数)
- `setfill(char)`：永久设置字段中填充的字符(默认为空格)
- `setw(int)`：暂时设置字段的宽度，在输出下一数据后恢复默认字段值(即0)，用于对齐画面等

除了第一条，其它三条控制符都需要包含头文件**`<iomanip>`**

这些控制符实际上是调用`iostream`类的祖宗类`ios_base`的成员函数`setf()`和部分成员，来实现控制输出的功能，其它控制符详见[`fmtflags`](#~fmtflags)<a id="fmtflags"></a>

注：`fmtflags`不是控制符，而只是部分控制符**调用了`setf(fmtflags)`**而已，`ios_base::fmtflags`和后续将提到的`ios_base::iostate,ios_base::openmode`都是一串二进制掩码(`bitmask`)，这意味着它们可以**通过`'|'`运算符叠加**，在`ios_base`中提供了它们的一些枚举常量

`ios`(即`basic_ios<char>`)类是`ios_base`类的派生类，能使用它的所有公开和保护性成员，所以直接用`ios::badbit`等也是正确的，所以在**除定义外的说明和实例中都使用`ios::`**

## 流状态

### 状态位

所有流对象都有称为流状态的属性，流的打开或关闭由三个状态位决定：

- `badbit`：系统性错误时(如文件不存在、文件不允许访问等)被设置，这种错误一般无法恢复
- `failbit`：格式错误时(如输入字符赋给整型对象、`get(str)`读取到空行等)被设置，这种错误可以恢复
- `eofbit`：遇到`eof`时被设置

`ios_base`类中有`ios_base::badbit,ios_base::failbit,ios_base::eofbit,ios_base::goodbit`四种标准状态常量，它们以三位二进制数表示，分别为`001,100,010,000`

当这三个状态位都被清除(即`0`)时，`goodbit`将被清除，表示这个流一切正常，是打开的

### 检查状态的方法

可通过对应的方法(`bad(),fail(),eof(),rdstate()`)查看这四个位的状态(具体应查看头文件定义，实际上有略微不同)

`operator!()`和`rdstate()`效果一样，而`good()`是与`operator!()`相反的方法，当`goodbit=0`时返回`true`

这些方法都在`ios`中定义

需要清楚的是，`rdstate()`的返回值是`ios_base::iostate`对象，而其它方法都返回布尔类型，这意味着`rdstate()`方法还可以通过与标准状态常量对比进行错误检测

这些方法都可用于检测错误，例如`rdstate()`可用于粗略地检查所有错误类型，出错则返回真

这些流并不会在出错时报错，这涉及到`ios::exceptions()`方法，它默认返回`goodbit`，只有当**`f.exceptions() & f.rdstate()`不为零**时，程序才会抛出`ios_base::failure`错误

如果需要让流报错，应改变`exceptions()`的返回值(使用它的重载函数`exceptions(iostate)`)：

```c++
// 示例
f.exceptions(ios::failbit);  // 设置成100,则一旦流的failbit被设置,将报错
try {
    ...
} catch (ios::failure& message){
    cerr << message.what();
}
```

### 清除状态的方法

`ios::clear()`和`ios::setstate()`方法可将流设置成正常，原理有所不同：

- `clear()`将该流的状态强制改成提供的状态，默认提供`0`，即`ios::goodbit`
- `setstate()`将该流的状态和提供的状态相叠加，相当于两个二进制数进行了**按位或**操作，实际上，进行`f.setstate(state)`相当于进行`f.clear(f.rdstate() | state)`

```c++
// 定义
file.clear(ios_base::iostate=0);  // 默认参数为000,不设置任何位,即清除所有三位
file.setstate(ios_base::iostate);  // 必须提供参数,只会设置它而不影响其它两位
// 示例
file.clear(ios::eofbit);    // 参数为010,设置eofbit,清除其它两位
file.clear(ios::eofbit | ios::badbit) // 参数为011,清除failbit,设置其它两位
```

当然，可以用其他流的状态作为参数，只是这没有意义

常用不带参数的`clear()`方法来强制打开流，而`setstate()`常被流对象自动调用，来叠加到自身的流状态里

## 文件`I/O`

实现文件`IO`需要包含头文件`<fstream>`，创建输入流则创建`ifstream`对象，创建输出流则创建`ofstream`对象，只不过流的另一端由键盘和显示器变成了文件

因为`ifstream,ofstream`分别为`istream,ostream`的派生类，所以可以像`cin,cout`一样使用方法，而除此之外，文件`IO`有额外的成员和方法

### 创建文件流和关闭文件

`ifstream`和`ofstream`类中额外定义了默认构造函数和带字符串(有`char*`类型和`string&`类型的两个重载)参数的构造函数

以`ifstream`为例，初始化一个文件流对象如下：

```c++
// 定义
ifstream fin;  // 默认构造函数,这会创建一个新的缓冲区(fb)并将fb的引用传递给基类初始化器
// 将字符串传递给构造函数,它将创建缓冲区并以默认为ios::in的模式调用open("fileName")
ifstream fin(char*,ios_base::openmode=ios_base::in);
ifstream fin(string&,ios_base::openmode=ios_base::in);
```

一个文件流对象除了继承`istream/ostream`的用于**格式化输入输出的流装置**外，还额外包含一个私有的`filebuf`对象，用于**与文件进行底层交互**；例如文件流对象含有的成员函数`open()`和`close()`，它们实质上是**调用`filebuf`类**的`open()`和`close()`，只是在类里面重写了一遍方便使用罢了

`fstream`类的对象较为特殊，它继承了两个缓冲区，目的在于用一个对象实现对文件的输入和输出

它们的析构函数是空的，这意味着文件是通过方法`close()`关闭的，即关闭文件不影响流对象的重复使用，而流的打开与关闭和**流状态**有关；这也更说明，流不是一个容器，而数据也只会**暂时**存储在缓冲区中

### 文件模式

对于一个文件流对象`fio`，如果要**检查它的流是否异常**(而不是文件是否打开)，一般不用像`cin,cout`一样的`operator!(),rdstate()`方法，而是成员函数**`is_open()`**(还是调用`filebuf`类的成员，但这次是间接调用了`file`类的成员)，它能检查出“文件打开方式错误”一类的异常

`C++`的文件模式设计与`C`有许多共通之处，但更好理解和记忆：

<table border="2" width="50px">
    <tr>
     <th>ios_base::openmode(以下省略ios::)</th>
        <th>C风格</th>
        <th>意义</th>
     <th>文件存在</th>
        <th>文件不存在</th>
    </tr>
    <tr>
        <td>in(ifstream默认值)</td>
        <td>"r"</td>
        <td>读打开</td>
        <td>正常</td>
        <td>异常(failbit+badbit)</td>
    </tr>
    <tr>
        <td>out(ofstream默认值;默认加上trunc)</td>
        <td>"w"</td>
        <td>写打开</td>
        <td>清空文件</td>
        <td>创建空文件</td>
    </tr>
    <tr>
        <td>ate</td>
        <td>fseek(file*,0,SEEK_END)</td>
        <td>跳到eof</td>
        <td colspan="2">文件必须已经打开</td>
    </tr>
    <tr>
        <td>app(默认加上out)</td>
        <td>"a"</td>
        <td>追加写打开</td>
        <td>不会被清空,并在文件尾开始添加内容</td>
        <td>创建空文件</td>
    </tr>
    <tr>
        <td>trunc</td>
        <td>-</td>
        <td>清空文件</td>
        <td colspan="2">文件必须已经写打开</td>
    </tr>
    <tr>
        <td>binary</td>
        <td>"b"</td>
        <td>二进制模式打开</td>
        <td colspan="2">文件必须已经打开</td>
    </tr>
    <tr>
        <td>in | out(fstream默认值)</td>
        <td>"r+"</td>
        <td>读写打开</td>
        <td>不清空文件,并在文件开头开始添加内容</td>
        <td>异常(failbit+badbit)</td>
    </tr>
    <tr>
        <td>in | out | trunc</td>
        <td>"w+"</td>
        <td>读写打开</td>
        <td>清空文件</td>
        <td>创建空文件</td>
    </tr>
    <tr>
        <td>other | ate</td>
        <td>"other"后,fseek(file*,0,SEEK_END)</td>
        <td>打开文件后跳到eof</td>
        <td colspan="2">视other而定</td>
    </tr>
    <tr>
        <td>other | binary</td>
        <td>"other+b"</td>
        <td>二进制other模式打开</td>
  <td colspan="2">视other而定</td>
    </tr>
</table>

以下为常用模式：

```c++
// 示例
ofstream("tmp/old");  // 创建空文件或覆盖旧文件,并写打开; 和显式声明out或out|trunc等价
ofstream("file",ios::app);  // 追加写打开; 和显式声明out|app等价
ifstream("file");   // 只读打开; 和显式声明in等价
fstream("file");   // 读写打开文件,不清空文件; 和显式声明in|out等价
fstream("tmp/old",ios::in|ios::out|ios::trunc);// 创建空文件或覆盖旧文件,并读写打开; 不常用
// 用于图像或其它二进制资源的读写:
ofstream("Icon/other",ios::out|ios::binary);
ifstream("Icon/other",ios::in|ios::binary);
```

### 读写与异常处理

一般来说，只要打开或关闭文件正常，那么文件流就会保持正常，对于文本内容，像使用`cin,cout`一样即可；而对于**二进制**内容，可以用`read()`(读方法，在`istream`类中)和`write()`(写方法，在`ostream`类中)，它们和其它输入输出方法的区别是在读取/输出时不会添加`\0`，且`read()`函数不提供默认或指定的终止符(`getline()`默认遇回车停止)：

```c++
fin.read(char*,int);  // 读取指定个字符到char*中,不支持string类
fout.write(char*,int);  // 向文件写入指定个字符,不支持string类
```

当然在关闭文件后，为了防止读到`EOF`或其他问题影响这个流对象的再使用，应选择改变`exceptions()`，或是不管不顾直接`clear()`强行打开流

## 附

以阅读头文件的方式回顾所学(`Doxygen`风格注释)，以下是部分内容在头文件内的定义(小标题是头文件名)：

### 各类关联

重命名：

<img src=".\1.1.1.1.typedef.png" alt="1.typedef" style="zoom:50%;" />

<img src=".\1.1.1.2.typedef.png" alt="2.typedef" style="zoom:50%;" />

继承关系：

<img src=".\1.1.2.classes.png" alt="classes" style="zoom: 80%;" />

### `ios_base.h`

以下都属于`ios_base`类：

流状态常量：

<img src=".\1.2.1.ios_base_Ios_Iostate.png" alt="ios_base_Ios_Iostate" style="zoom:50%;" />

<img src=".\1.2.2.ios_base_iostate.png" alt="ios_base_iostate" style="zoom:50%;" />

<img src=".\1.2.3.ios_base_bit.png" alt="ios_base_bit" style="zoom:50%;" />

[控制符相关](#fmtflags)<a id="~fmtflags"></a>：

<img src=".\1.2.4.1.ios_base_fmtflags.png" alt="1.ios_fmtflags" style="zoom:50%;" />

<img src=".\1.2.4.2.ios_base_fmtflags.png" alt="2.ios_fmtflags" style="zoom:50%;" />

<img src=".\1.2.4.3.ios_base_fmtflags.png" alt="3.ios_fmtflags" style="zoom:50%;" />

<img src=".\1.2.4.4.ios_base_fmtflags.png" alt="4.ios_fmtflags" style="zoom:50%;" />

<img src=".\1.2.5.ios_base_setf.png" alt="ios_setf" style="zoom:50%;" />

文件模式：

<img src=".\1.2.6.1.ios_base_openmode.png" alt="1.ios_base_openmode" style="zoom:50%;" />

<img src=".\1.2.6.2.ios_base_openmode.png" alt="1.ios_base_openmode" style="zoom:50%;" />

### `basic_ios.h`

<img src=".\1.3.1.ios_method.png" alt="ios_method" style="zoom:50%;" />

<img src=".\1.3.2.ios_clear()&setstate().png" alt="ios_clear()&setstate()" style="zoom:50%;" />

<img src=".\1.3.3.1.ios_checkErr.png" alt="1.ios_checkErr" style="zoom:50%;" />

<img src=".\1.3.3.2.ios_checkErr.png" alt="2.ios_checkErr" style="zoom:50%;" />

<img src=".\1.3.4.1.ios_exceptions.png" alt="1.ios_exceptions" style="zoom:50%;" />

<img src=".\1.3.4.2.ios_exceptions.png" alt="2.ios_exceptions" style="zoom:50%;" />

以上是`ios`类中的成员，下面这个控制符以及`oct,dec`等是本头文件内的内联函数：

<img src=".\1.3.5.ios_hex.png" alt="ios_hex" style="zoom:50%;" />

### `istream`&`ostream`

以`istream`为例，`operator<<`与其类似：

<img src=".\1.4.1.iostream_exceptions.png" alt="iostream_exceptions" style="zoom:50%;" />

`get(...)`与其类似：

<img src=".\1.4.2.iostream_strInput.png" alt="1.4.2.iostream_strInput" style="zoom:50%;" />

<img src=".\1.4.3.iostream_ignore.png" alt="1.4.3.iostream_ignore" style="zoom:50%;" />

基类初始化器：

<img src=".\1.4.5.iostream_ctor.png" alt="iostream_ctor" style="zoom:50%;" />

控制符`flush`和`endl`都在`<ostream>`里，它们是内联函数：

<img src=".\1.4.4.iostream_endl.png" alt="1.4.4.iostream_endl" style="zoom:50%;" />

### `ifstream`&`ofstream`

以`ifstream`类为例：

<img src=".\1.5.1.fstream_ctor.png" alt="fstream_ctor" style="zoom:50%;" />

<img src=".\1.5.2.fstream_detor.png" alt="fstream_detor" style="zoom:50%;" />

### `fstream`

这些是`filebuf`类里的：

<img src=".\1.6.1.fstream_open.png" alt="fstream_open" style="zoom:50%;" />

<img src=".\1.6.2.fstream_close.png" alt="fstream_close" style="zoom:50%;" />
