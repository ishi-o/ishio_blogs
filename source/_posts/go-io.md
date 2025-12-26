---
title: 'Go: I/O'
categories:
  - Programming
  - Go
  - Spec & Stdlib
tags:
  - Go
  - I/O API
mathjax: false
date: 2025-12-25T11:37:22.000Z
---
<!-- placeholder -->
<!-- more -->

# `Go I/O`

## `os`

### `os.File`

- `os.File`是`Go`对文件描述符的抽象
- `os.OpenFile(name string, flag int, perm os.FileMode) (*os.File, error)`：用指定`flag`打开文件并返回文件指针，若发生错误则`error`不为`nil`
  - `flag`包括

    ```go
    import "syscall"

    const (
    	// Exactly one of O_RDONLY, O_WRONLY, or O_RDWR must be specified.
    	O_RDONLY int = syscall.O_RDONLY // open the file read-only.
    	O_WRONLY int = syscall.O_WRONLY // open the file write-only.
    	O_RDWR   int = syscall.O_RDWR   // open the file read-write.
    	// The remaining values may be or'ed in to control behavior.
    	O_APPEND int = syscall.O_APPEND // append data to the file when writing.
    	O_CREATE int = syscall.O_CREAT  // create a new file if none exists.
    	O_EXCL   int = syscall.O_EXCL   // used with O_CREATE, file must not exist.
    	O_SYNC   int = syscall.O_SYNC   // open for synchronous I/O.
    	O_TRUNC  int = syscall.O_TRUNC  // truncate regular writable file when opened.
    )
    ```

    只读、只写、读写、追加、创建、不存在才创建、同步写、存在则清空，它们可以使用按位或运算同时存在

    必须三选一：指定只读、只写、读写

    其它标志可以使用按位或运算配合使用

    其中`O_EXCL`必须配合`O_CREATE`一起使用，此时如果文件已存在则会报错

  - `FileMode`：表示文件类型和文件权限，`Linux`下用`ls -l`查询出的权限的分段二进制表示，比如`0444`

    `FileMode`只有在**创建文件时**有效，在其它情况下使用`0`即可

    ```go
    import "io/fs"

    // A FileMode represents a file's mode and permission bits.
    // The bits have the same definition on all systems, so that
    // information about files can be moved from one system
    // to another portably. Not all bits apply to all systems.
    // The only required bit is [ModeDir] for directories.
    // os
    type FileMode = fs.FileMode

    // fs
    type FileMode uint32
    ```

- `Open(name string) (*File, error)`：只读打开文件
- `Create(name string) (*File, error)`：创建文件，若文件存在则截断，否则以`0o666`模式创建
- `NewFile(fd uintptr, name string) *File`：将`fd`封装成`*File`返回，`*File`可以使用`Fd()`获取底层的`uintptr`
- `*File`一定要记得`Close()`，例如在获取后立刻`defer f.Close()`

### 文件`I/O`错误

- `os`提供一系列方法帮助判断`err`：
  - `IsExist(error)/IsNotExist(error)`：判断错误是否为文件存在/文件不存在
  - `IsPermission(error)`：判断错误是否为文件权限不足
  - `IsTimeout(error)`：判断错误是否为超时

### 读写文件

- `*File`提供的方法：
  - `Read(b []byte) (n int, error)`：读取文件内容存储到`b`，只会覆盖到`len(b)`
  - `ReadAt()`：可额外传递偏移量（开始读的偏移量而不是字节切片的偏移量）
  - `Write(b []byte) (n int, error)`：将`b`的内容写入文件，只会写入直到`len(b)`的内容
  - `WriteAt()`：可额外传递偏移量（开始写的偏移量而不是字节切片的偏移量），和`os.O_APPEND`互斥
  - `WriteString(s string) (n int, error)`：写入字符串
- `os.ReadFile(name string) ([]byte, error)`：读取一个文件的全部内容并返回字节切片
- `os.WriteFile(name string, data []byte, FileMode) error`：将字节切片的全部内容写入文件

### 文件元属性操作

- `os.Rename(oldpath, newpath string) error`：重命名文件
- `os.Delete(name string) error`：删除文件
- `os.DeleteAll(dir string) error`：递归删除目录下所有文件和目录
- `(*File) Sync()`：刷盘

### 目录操作

- `os.ReadDir(name string) ([]DirEntry, error)`：读取一个目录，返回切片

  ```go
  import "os"

  func A() {
  	dirs, _ := os.ReadDir(".")
  	for _, dir := range dirs {
  		dir.Name()  // string
  		dir.IsDir() // bool
  		dir.Info()  // (os.FileInfo, error)
  		dir.Type()  // os.FileMode
  	}
  }
  ```

- 或者使用`*File`的`ReadDir(n int)`，最多读取`n`个`DirEntry`，`n`为负数时读取全部

  `os.ReadDir()`是对它的简单封装

- `os.Mkdir(name string, perm FileMode) error`：创建目录
- `os.MkdirAll(path string, perm FileMode) error`：类似`mkdir -p`地创建目录
- `os.Getwd() (string, error)`：获取工作目录
- `os.Chdir(dir string) error`：修改工作目录

### 环境变量相关

- `os.Getenv(key string) string`
- `os.Setenv(key, value string) error`
- `os.LookupEnv(key string) (string bool)`
- `filepath`标准库提供一系列处理文件路径的函数

## `io`与`io/fs`

### `I/O`接口

- `io.Reader`：只有`Read([]byte) (n int, error)`方法
- `io.Writer`：只有`Write([]byte) (n int, error)`方法
- `io.Closer`：只有`Close() error`方法
- `io.Seeker`：只有`Seek(offset int64, whence int) (int64, error)`方法
- `io.ReadWriter`：`Reader`与`Writer`并集
- `io.ReadCloser`：`Reader | Closer`
- `io.WriteCloser`：`Writer | Closer`
- `io.ReadWriteCloser`：`Writer | Reader | Closer`
- `io.ReadSeeker`：`Reader | Seeker`
- `io.ReadSeekCloser`：`Reader | Seeker | Closer`
- 更多接口可查阅文档
- `*os.File`实现了上述所有接口

### `io`工具函数

- `io.Copy(dst Writer, src Reader) (written int64, err error)`：复制文件，默认使用`32K`的缓冲区
- `io.CopyBuffer(Writer, Reader, []byte) (int64, error)`：可以自己指定缓冲区
- `io.CopyN(Writer, Reader, int64)`：指定最多读取的字节数
- `io.ReadAll(Reader) ([]byte, error)`：读取所有字节并返回字节切片
- `io.ReadFull(Reader, []byte) (int, error)`：调用`Reader`的`Read`，若长度不足则报错
- `io.MultiReader(...Reader) Reader`：串联多个`Reader`顺序读取
- `io.MultiWriter(...Writer) Writer`：并联多个`Writer`同时写入

### `fs.FS`

- `fs.FS`将一系列文件资源包括网络资源都抽象成`fs.FS`，`FS`接口只有一个`Open(name string) (File, error)`方法
- 可以通过`os.DirFS(dir string) fs.FS`将目录转化为`FS`对象
- `fs.Sub(fsys FS, dir string) (FS, erorr)`：将文件系统对象的一个子目录再转换为文件系统对象
- `fs`提供了很多查找和读取文件的方法：
  - `fs.ReadFile(FS, name string) ([]byte, error)`：读取整个文件
  - `fs.Open(name string) (File, error)`：打开一个文件，根目录为文件系统对象的目录
  - `fs.File.Read([]byte) (int, error)`：读取一个文件读取到字节切片里
  - `fs.File.Close() error`：关闭文件
  - `fs.Glob(FS, pattern string) ([]string, error)`：模式匹配地查找文件名
  - `fs.ValidPath(name string) bool`：检查路径是否生效
- 以及很多关于目录的方法：
  - `fs.WalkDir(fsys FS, root string, WalkDirFunc) error`：递归遍历目录，`root`相对于`fsys`的路径，`WalkDirFunc`回调用于自定义处理函数
  - `fs.ReadDir(FS, string) ([]DirEntry, error)`：读取目录内容
  - `WalkDirFunc`：

    ```go
    func(path string, d DirEntry, err error) error
    ```

    在回调中需要处理`err`，对`d`进行自定义的处理

    返回`nil`表示继续遍历，`fs.SkipDir`表示跳过当前的`d`，`fs.SkipAll`表示跳过所有的`d`

- 在仅需要读取和遍历文件或其它资源的情况下，使用`fs`比使用`os`更好

## 标准输入输出

- `os`包内置声明了三个标准输入输出描述符：`Stdin`、`Stdout`、`Stdout`
- `os`包的这三个描述符本身就是文件，可以使用上述的`os`、`io`、`fs`提供的方法输出和输入
- 内置的`print()`和`println()`可以打印，但通常用于测试
- `fmt`包提供许多用于打印的方法：
  - `Print()/Println()/Printf()`
- `fmt`格式化：
  - `%%`：百分号
  - `%s/%d/%f/%b/%o/%x/%p`：与其它语言类似
  - `%v`与`%+v`：输出值的原始形式，后者会附加值的字段
  - `%q`：输出带有双引号的字符串

## `bufio`

### `bufio`接口

- `bufio`标准库用于缓冲读写，适用于大文件，但是`bufio`是顺序访问，不像`os.File`那样是随机访问
- `bufio.NewReader(io.Reader) *bufio.Reader`：将`io.Reader`转化为带缓冲的`Reader`
- `bufio.NewWriter(...) ...`：将`io.Writer`转化为带缓冲的`Writer`
- `bufio.NewReadWriter(...) ...`：将`io.ReadWriter`转化为带缓冲的`ReadWriter`
- `bufio.NewScanner(io.Reader) *bufio.Scanner`：将`io.Reader`转化为`bufio.Scanner`

### `bufio.Reader`

- `Read(...)...`：与`*os.File.Read()`类似，提供缓冲
- `ReadByte() (byte, error)`：读取一个字节
- `ReadBytes(delim byte) ([]byte, error)`：读取到分隔符`delim`
- `ReadString(delim byte) (string, error)`：读取到分隔符`delim`，返回`string`
- `ReadLine() (line []byte, isPrefix bool, error)`：读取一行
- `Peek(n int) ([]byte, error)`：预读取`n`个字节但是不移动指针
- `Reset(io.Reader)`：复用`*bufio.Reader`对象，绑定到新的`io.Reader`

### `bufio.Writer`

- 与`bufio.Reader`的方法相对应
- `Flush() error`：刷盘
- `Reset(io.Writer)`：复用`*bufio.Writer`对象，绑定到新的`io.Writer`

### `bufio.Scanner`

- `Scan() bool`：按行扫描，若指针没有到`EOF`则返回`true`
- `Text() string`：获取当行的文本
- `Err() error`：检查扫描中的错误
- `Split(SplitFunc)`：自定义分割函数，内置的分割函数如下
  - `bufio.ScanLines`(默认)
  - `bufio.ScanWords`
  - `bufio.ScanRunes`
  - `bufio.ScanBytes`

