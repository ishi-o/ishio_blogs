---
title: "Java: IO"
date: 2025-04-17
categories: [Programming, Java, Java SE]
tags: [Java, I/O API]
---
<!-- placeholder -->
<!-- more -->
# `Java IO`

## `java.io`

### 文件类

- `File`：文件类对象可表示一个文件，包括普通文件和目录文件，在文档中称为“抽象路径”
- `File`类对象只能修改文件的属性，不能读写文件
- `File`构造方法：提供文件路径或其它`File`类对象
- `File`文件操作在大多数情况下不常用且不好用，更多是查询文件的属性，很多时候应该使用`Files`或`apache`提供的`io`方法，因此这里不介绍
- `RandomAccessFile`：随机访问文件类和文件类完全不同，不能修改文件的元属性，但是能随机访问或修改文件的内容
- `RandomAccessFile`构造方法：提供文件路径或其它`File`类对象，并提供用字符串表示的模式参数
  - `"r"`：只读打开，文件不存在或发生写操作会抛出`IOException`受检异常
  - `"rw"`：读写打开，文件不存在会创建，此处的“写”是覆盖
  - `"rwd"`：读写打开，并在缓存满时或调用`close()`时同步
  - `"rws"`：读写打开，并在每一次写操作后同步
  
- `RandomAccessFile`常用方法：
  - `long getFilePointer()`：返回当前的文件指针
  - `seek(long pos)`：将文件指针移到`pos`处
  - `close()`：关闭并释放这个随机访问流
  - `Type readType()`和`writeType(Type x)`：支持读写并自动解析一个基本数据类型的数据
  - `String readLine()`：读一行文本
  - `readFully(byte[] b, int off, int len)`：读取当前指针后最多`len`个字节，读入`b+off`中
    此函数有`readFully(byte[] b)`重载，将当前指针后的所有字节读入`b`中
  - `writeBytes(String)`和`writeChars(String)`：将字符串以`byte`序列形式、`Unicode`编码形式写入文件
  - `String readUTF()`和`writeUTF(String s)`：以`UTF8`编码读写字符串
  - `getFD()`：获取`FileDescriptor`对象
- `FileDescriptor`：不含任何操作功能，而是文件资源在`java`中抽象的描述符
  - `boolean valid()`：检查描述符是否有效
  - `sync()`：将缓冲区内容同步到硬盘，上述`RandomAccessFile`的同步就是调用`getFD().sync()`完成的

### 流的概念

- 流是`java`提供的另一种操作数据输入输出的机制，和`RandomAccessFile`不同，它们是顺序读写的，曾经读过的数据无法再读(除非支持标记)，在读取数据前必须将所有前面的数据全部读取
- 按数据流向分类，分为输入流、输出流
- 按数据单位分类，分为字符流、字节流，字符流以两个字节为单位、字节流以单个字节为单位
  `java`对这两个分类依据有较易区分的命名风格，输入字符流为`XxxReader`、输出字符流为`XxxWriter`、输入字节流为`XxxInputStream`、输出字节流为`XxxOutputStream`
- 按功能分类，分为节点流、处理流，节点流对象直接和数据源绑定，仅提供读取字节或字符的功能，处理流则是节点流的封装，具体体现在构造方法上，节点流需要资源描述符对象，而处理流需要流对象

### 字节流

- 所有字节流类都继承`InputStream`或`OutputStream`
  - `InputStream`：仅提供读取单个或多个字节的方法，除此之外：
    `long skip(long n)`：跳过最多`n`个字节，返回实际跳过的字节数
    `boolean markSupported()`：测试该输入流是否支持`mark()`和`reset()`
    `mark(int lim)`：对现在指针作标记，该标记在此后的`lim`个字节内才有效
    `reset()`：复位到上一个有效的标记，若不存在则抛出异常
    通过`mark()`和`reset()`配合可以实现重复读取某一段连续字节
  - `OutputStream`：仅提供写入单个或多个字节的方法，除此之外有`flush()`方法能快速清空缓冲区
- 节点流最重要的是`FileInputStream`和`FileOutputStream`，可以提供文件路径或抽象文件对象构造文件字节流，`FileOutputStream`可以提供`append`布尔值参数表示是否以追加模式写入
  文件流本身不支持标记，`ByteArray`系列的流支持但并不常用(因为只能处理小文件)，更常用的是处理流`Buffered`系列的流
- 处理流则非常多样：
  - `BufferedInput/OutputStream`：十分常用的缓冲处理流，本身不提供解析之类的方法，而是用于管理缓冲区的流，支持`mark`
  - `DataInput/OutputStream`：能将字节数据解析为一系列基本数据类型的处理流，和`RandomAccessFile`提供的输入/输出方法一致
    `readLine()`方法已被弃用，建议使用`BufferedReader`提供的`readLine()`方法
  - `ObjectInput/OutputStream`：能将字节数据解析为`Object`对象的处理流，用于序列化和反序列化，要求类实现`Serializable`空接口；`java`原生的序列化有安全、性能问题
  - `PipedInput/OutputStream`：用于线程间通信，输入流或输出流通过`connect()`方法连接配套的输出流或输入流
  - `SequenceInputStream`：序列处理流，用于将一系列输入流串联起来，统一由`SequenceInputStream`对象管理
    需要用到传统的`Enumeration`枚举器接口，调用`Vector<InputStream>`对象的`elements()`方法即可，但实际上现在已经很少使用了
  - `PrintStream`：提供的最常用的方法是`print()`、`println()`的一系列重载方法，`System.out`是`PrintStream`类的`final`对象

### 字符流

- 所有字符流类都继承`Reader`或`Writer`
- 处理对象数据或二进制数据时，字节流通常更好用，但当数据是字符时，用字符流类来处理更简单(例如编码问题)，所以一般会用`InputStreamReader`将输入字节流转化为输入字符流，处理后通过`OutputStreamWriter`这个输出字符流将内容写入输出字节流

## `java.nio`

- `java.io`是阻塞型`IO`(`BIO`)，是面向流的，而`java.nio`是`java 1.4+`支持的`New IO`(或译为`Non-Block IO`)，是面向通道、缓冲区、选择器的非阻塞`IO`
  这两种`IO`都是同步的，需要程序主动检查`IO`任务是否完成，传统的`IO`优势在于小规模、低并发的操作(例如小文件操作)，而在网络`IO`中`nio`有较大优势
- `java.nio`的读写是全双工的

### `Buffer`

- 在`nio`中，`Buffer`用于**处理数据**，其子类包含`IntBuffer`、`ByteBuffer`、`CharBuffer`、`LongBuffer`，最常用的自然是`ByteBuffer`
- `Buffer`及上述几个子类都是抽象类，不能被实例化但其子类可以通过以下静态方法获取`Buffer`对象：
  - `allocate(int)`：获取一个自动分配静态数组的`Buffer`实例对象(`HeapXxxBuffer`)
    使用`JVM`堆内内存
  - `wrap(type[])`将提供的静态数组封装为`Buffer`对象
  - `allocateDirect(int)`：获取一个自动分配静态数组的`Buffer`实例对象(`DirectXxxBuffer`)
    使用堆外的本地内存，本地内存在分配与回收时消耗更大，但面对大量、频繁的读写更快
- `Buffer`有三个重要属性：`mark`(标记)、`position`(指向当前指针)、`limit`(现存数据大小)，`mark<=position<=limit`
- `Buffer`也提供随机读写的功能，允许对象在`position`属性小于`limit`属性时读写，并提供了以下和`position`与`mark`有关的方法：
  - `position()`和`position(int)`：获取或设置`position`属性
  - `limit()`和`limit(int)`：获取或设置`limit`属性
  - `mark()`：标记当前的`position`为`mark`
  - `reset()`：设置当前的`position`为当前的`mark`
  - **`flip()`**：设置`limit`为当前的`position`，再将`position`设置为`0`，相当于`buf.limit(buf.position()).position(0)`
  - `rewind()`：设置`mark`为当前的`position`
  - `remaining()`和`hasRemaining()`：返回`limit`与`position`的差值、判断该差值是否大于零
  它们均返回对象本身，因此可以级联操作，其中`flip()`很常用，因为通常需要向`Buffer`对象写入数据后紧接着读取这些数据

- `ByteBuffer`提供的读写方法在命名上和流有所区别：`read`变为`get`、`write`变为`put`，它们会自动更新`position`和`limit`
  除了相对存取，还允许提供`index`参数作为第一个参数，称为绝对存取，绝对存取不影响`position`和`limit`
- `Buffer`提供`duplicate()`方法创建一个共享同一个数据数组的缓冲区对象
  `XxxBuffer`提供`asXxxBuffer()`和`asReadOnlyBuffer()`创建一个其它类型的、或只读的本类型的`Buffer`对象，仍然共享同一个数据数组

### `FileChannel`

- `Channel`是一个接口，实现该接口的实例对象表示数据源到缓冲区的通道，它不能处理数据，而是负责**存取和传输数据**
- 有以下类实现了该接口：
  - `FileChannel`：文件通道，可以通过`RandomAccessFile`对象或流对象的`getChannel()`方法获取，也可以通过静态方法`open()`获取，其中`open()`方法实际上是`nio.2`提供的，将在下一节介绍
  - `SocketChannel`：客户端的`TCP`套接字通道，可以通过`Socket`对象的`getChannel()`方法获取
  - `ServerSocketChannel`：服务端`TCP`套接字连接的通道，可以通过`ServerSocket`对象的`getChannel()`方法获取
  - `DatagramChannel`：`UDP`套接字的通道
  在这里仅介绍`FileChannel`提供的方法，其余通道将在网络编程中详细介绍
- `read(Buffer dest)`和`write(Buffer src)`是通道类最重要的两个方法
- `force(false)`方法允许强制刷盘，立刻同步通道的更改到硬盘中，`force(true)`不仅会同步内容，还会同步文件元数据
- `truncate(long size)`：截断通道的内容，丢弃第`size`个字节及之后的数据

### `MappedByteBuffer`

- `MappedByteBuffer`：文件映射缓冲，是文件在内存中的直接映射，避免了传统`IO`那样需要切换内核态进行存取操作，效率更高，但和硬盘上的数据不是随时同步的，可以通过`force()`立刻刷新缓冲区
  通常通过通道对象调用`map()`方法创建，同样是本地内存，因此读写更快而分配与回收较慢，适用于大型文件
- 映射有三种模式：`FileChannel.MapMode`的`READ_ONLY`与`READ_WRITE`与`PRIVATE`
  其中`READ_WRITE`模式的更改会同步到文件但不会同步到其它缓冲区，`PRIVATE`模式的更改只会修改当前的缓冲区，必须通过`force()`刷盘同步更新
- `MappedByteBuffer`和`DirectByteBuffer`的区别是它和通道有直接关联，所以可以不通过`FileChannel`进行`read()`和`write()`，而是依靠缺页中断直接在内存映射文件中读写数据

### `java.nio.charset`

## `nio2`

### `AsynchronousChannel`

- `nio2`或称`AIO`，支持异步操作和更全面的跨文件系统支持，主体为`java.nio.file`包
- 除此之外提供异步的通道，名称均以`Asynchronous`开头，它们和`nio`的通道的方法一致，不再赘述

### `Path`和`Paths`

- `Paths`是工具类，仅提供静态方法用于创建`Path`对象，但在新版本中由于接口允许含有静态方法，`Paths.get()`内部被替换为`Path.of()`
  所以直接调用`Path.of()`创建`Path`对象即可，需要提供`URI`(网络资源)或`String`(本地资源)
- 在`Path`中，`.`表示`classpath`
- `Path`和`File`起的作用类似，均表示一个抽象路径，提供以下实例方法：
  - `getFileName()`：获取`String`表示的文件名
  - `getFileSystem()`：获取`FileSystem`对象
  - `getNameCount()`：获取该`Path`上级目录的个数
  - `getName(int idx)`：获取第`idx`层上级目录的`Path`对象，`idx`从`0`开始表示最上级目录
  - `iterator()`：获取用于遍历上级目录的迭代器
  - `getParent()`：获取绝对路径中上级目录的`Path`对象，若为相对路径则返回`null`
  - `getRoot()`：获取绝对路径中根目录的`Path`对象，若为相对路径则返回`null`
  - `isAbsolute()`：判断是否为绝对路径
  - `normalize()`：规范化路径，将所有`.`和`..`解析并去除
  - `relative(Path p)`：返回该对象和`p`之间的相对路径
  - `resolve(Path p)`：若`p`为绝对路径则返回`p`，否则拼接在本`Path`对象之后并返回
  - `resolveSibling(Path p)`：若`p`为绝对路径则返回`p`，否则拼接在本`Path`对象的`getParent()`对象之后并返回
  - `startsWith()`和`endsWith()`：判断该对象是否以某个`Path`或某个`String`起始、结尾
  - `toAbsolutePath()`：返回对应的绝对路径
  - `toUri()`：返回对应的`URI`

### `Files`

- `Files`是工具类，提供更丰富的文件操作
- `copy(Path, Path, CopyOption...)`：复制文件，默认行为为非原子地复制文件内容和最后修改时间到未存在文件，不保留硬链接、若源文件为软链接则取其指向文件的数据
  其中`CopyOption`是空接口，有枚举`StandardCopyOption`实现
  - `ATOMIC_MOVE`：原子地移动，仅支持`move()`
  - `REPLACE_EXISTING`：若目标已存在则覆盖内容
  - `COPY_ATTRIBUTES`：复制文件元数据，包括权限和所有者等
- `move(Path, Path, CopyOption...)`：移动文件，默认行为为非原子性地移动文件内容到未存在文件，若为软链接则复制软链接本身

### `java.nio.file.attribute`

### `PathMatcher`
