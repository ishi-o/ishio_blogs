---
title: "Qt: 数据库连接"
date: 2024-06-30
categories: [Programming, C/Cpp, Qt]
tags: [Qt, databse]
---
<!-- placeholder -->
<!-- more -->
# 数据库

## 介绍

数据库分两种，关系型(`Relational Database`)和非关系型(`Not Only SQL`)，它们的区别：

- `RDBMS`：使用表格存储数据；具有强大的查询性能；但只能存储在硬盘，读写性能较差
- `NoSQL`：提供多种存储数据的结构；不支持`SQL`，查询方面差；但可使用`RAM`，数据量大时有优势

数据库必须符合`ACID`：

- `Atomicity`：事务若执行失败，数据库能恢复到事务开始前的状态
- `Consistency`：写入的数据必须符合预设规则，否则事务执行失败
- `Isolation`：允许并发事务的存在，是线程安全的
- `Durability`：数据及其修改是永久存在的(体现在保存于硬盘上)

`SQL`(`Structured Query Language`，结构化查询语言)是用于操作数据库的语言

不同数据库提供的`SQL`不一样，所以常出现在数据库的名字里

目前常用的开源数据库有：

- `SQLite`：轻量的数据库
- `MySQL`
- `PostegreSQL`

[`Navicat`](https://blog.csdn.net/qq_54621492/article/details/139901605)是一款能直观地操作数据库的应用

## `SQL`标准

### 杂碎知识

- 虽然不同数据库的`SQL`语法有差别(实现上也不同)，但它们都遵循`SQL`标准
- `SQL`是不区分大小写的
- 称表的一行为一条记录
- 通过`CREATE DATABASE dbName`创建数据库
- 文本数据需要添加单引号或双引号；当文本数据是一个数字时，需要仔细检查

### 常用数据类型

- 文本类型：
  - `CHAR(N)`：长度固定为`N`的字符串，文本长度不足则用空格补充，`N`最长一般为`65535`
  - `VARCHAR(N)`：长度可变的字符串，最长为`N`
  - `TEXT`：适用于长文本
- 数值类型：
  - `INT/INTEGER`：`4`字节整型
  - `SMALLINT`：`2`字节整型
  - `BIGINT`：`8`字节整型
  - `FLOAT(D)`：小数位最多为`D`的浮点数
  - `DECIMAL(M, D)`：总位数为`M`，小数位数为`D`的浮点数
- 时间类型：
  - `DATE`：存储日期`YYYY-MM-DD`
  - `TIME`：存储时间`HH:MM:SS`
  - `DATETIME`：存储日期时间`YYYY-MM-DD HH:MM:SS`
- 其它：
  - `BLOB`：存储大量二进制数据(如图片、音频等)
  - `ENUM`：枚举类型

### 查询语句

查询结果是类似于表的结构，对数据库的许多操作需要配合查询语句一起使用

**`SELECT`系列语句用于确定列，`WHERE`系列语句用于确定行**

```sql
# 查询列
SELECT * FROM tableName;    # 查询整个表
SELECT columnNameList FROM tableName; # 查询特定列, 列间用逗号隔开

# 查询时过滤记录
SELECT DISTINCT ... ;     # DISTINCT: 查询后,自动去除完全重复的记录
SELECT ... WHERE condition;    # WHERE condition: 按condition过滤记录

# condition的编写: 在查询时, 若表达式返回真, 则加入查询结果
# 格式          返回真的情况
columnName >/=/< value     # 和值比较
columnName IS NULL      # 空值
columnName BETWEEN val1 AND val2  # 介于val1, val2间的值
columnName IN (val1, ...)    # 等于括号内任意值
columnName LIKE val      # LIKE模糊查询
columnName REGEXP val     # 正则表达式匹配
columnName GLOB val      # SQLite支持GLOB匹配

# condition语句编写
# 括号优先级最高; 若不熟悉运算符的优先级, 可以直接打括号
exp1 AND exp2       # 逻辑与
exp1 OR exp2       # 逻辑或
NOT exp         # exp的否定

# 在condition后可添加ORDER BY语句, 表示按列排序
# 默认为ASC(上到下升序), DESC为上到下降序
# 若有复数个列, 则按顺序排序(可认为是先排最后列, 再排第一列)
SELECT ... FROM ... WHERE ... ORDER BY columnNames ASC / DESC;
# 例: 对下表查询SELECT * FROM table ORDER BY name, id; (先排id, 再排name)
id | name ---> id | name ---> id | name
3  a    1  b    2  a
1  b    2  a    3  a
2  a    3  a    1  b

# 起别名:
# 在查询语句中, 所有涉及表名、列名的语句, 可在名字后添加 AS 别名, 查询结果将使用别名
SELECT * FROM a AS newName;
```

`LIKE`通配符：

- `%`：匹配任意数目的字符
- `_`：匹配单个字符
- 若字段中含通配符，可通过`ESCAPE`转义字符：`LIKE 'a\_b' ESCAPE '\'`

### 对表内容的操作

对表内容的操作一定记得联合查询语句一起使用，否则可能会修改所有记录

```sql
# 更改记录内容
# WHERE指定要修改的行, SET指定要修改的列
UPDATE tableName SET column1=value1,...,columnN=valueN WHERE condition;

# 删除具体记录(配合查询语句)
DELETE FROM tableName WHERE condition;

# 插入新记录
INSERT tableName (columnList) VALUES (valueList);

# 以上为对列进行修改的语句, 而更新表的列需要用到ALTER系列语句
# 添加列
ALTER TABLE tableName ADD columnName valueType

# 删除列(SQLite不支持)
ALTER TABLE tableName DROP COLUMN columnName

# 更改列的数据类型及其约束
ALTER TABLE tableName MODIFY COLUMN columnName valueType

# 修改列名
ALTER TABLE tableName CHANGE columnName newName;

# 需要检查的情况(需要设置主键约束):
# 记录不存在则插入、存在则更新
INSERT OR REPLACE INTO ...  # SQLite
REPLACE INTO ...    # MySQL
INSERT IGNORE INTO ...   # 记录不存在则插入、存在则忽略操作
```

### 对表整体的操作

```sql
# 创建表
# 列名不是数据, 不用加引号
CREATE TABLE tableName (
 columnName dataType,   # 列名 该列的数据类型 逗号分隔开不同列
    ...
);

# 用oldTable的查询结果作为内容, 创建新表
CREATE TABLE newTable AS SELECT * FROM oldTable;

# 用oldTable的查询结果作为内容, 插入 已创建 的表里
INSERT INTO exTable (columnList) SELECT * FROM oldTable;

# 修改表名
ALTER TABLE tableName RENAME TO newName;

# 删除表
DROP TABLE tableName;

# 删除表的所有数据, 保留表
TRUNCATE TABLE tableName;   # 一次性删除所有记录
DELETE FROM tableName;    # 找出所有行, 逐个删除(DELETE还可用于删除特定行)
```

### 部分关键字

```sql
# 创建/删除表时, 判断表是否不存在/存在:
CREATE TABLE IF NOT EXISTS ...;   # 若表不存在, 才创建
DROP TABLE IF EXISTS ...;    # 若表存在, 才删除

# 数据的约束(创建表时, 在数据类型后指定; 或者使用ALTER ... MODIFY修改)
NOT NULL       # 该列数据一定非空
UNIQUE        # 该列数据各不相同
PRIMARY KEY       # 主键约束:该列数据各不相同, 且不为空
DEFAULT value      # 插入记录时, 若未指定该列的值, 则使用默认值value
AUTO_INCREMENT      # 用于整型的自动增长
COMMENT        # 注释(放在最后)

# 格式特殊的外键约束:
# 用于绑定两表的两列, 本表是从表, otherTable是主表
# 从表添加记录时, 外键约束列的数据必须存在于主表内
# 从表删除记录时, 主表对应的记录不删除
FOREIGN KEY columnName REFERENCES otherTable(columnName) dataType;
# 外键约束的关键字:
CASCADE  # 主表修改/删除记录时, 从表对应外键字段随之修改/删除, 常用
RESTRICT # 和CASCADE相反
SET NULL # 主表删除记录时, 从表对应外键字段设置为空
# 也可将修改和删除拆开:
ON UPDATE CASCADE / ON DELETE CASCADE / ON UPDATE RESTRICT / ON DELETE RESTRICT

# 可以在数据类型后指定, 也可以在后面批量设置约束:
CREATE ... (
 ...,
    CONSTRAINT 约束名 UNIQUE (要添加的列)
);
```

## `Qt Sql`

### 前言

`Qt`提供了两种通过`C++`修改数据库的方法：一是模型/视图套件；二是提供查询接口，直接输入`SQL`语句

它对`SQLite`的支持是最好的，可直接使用；而有些数据库的驱动需要自行编译，例如`MySQL`

**[`MySQL`驱动编译](https://www.cnblogs.com/zhuchunlin/p/16485929.html)**

### `QSqlDatabase`

该类的对象是将数据从硬盘加载到内存上的桥梁

连接数据库：

```c++
// 以连接SQLite为例
QSqlDatabase db = QSqlDatabase::addDatabase("QSQLITE"); // 创建数据库对象
db.setHostName("127.0.0.1"); // 设置主机名
db.setPort(3306);    // 设置端口
db.setUserName("root");   // 设置拥有者名
db.setPassword("1");   // 设置密码
db.setDatabase("mdb.db");  // 设置数据库路径; SQLite是一个.db文件
db.open();      // 打开数据库

// 含lastError()方法:返回该对象最近发生的错误, 若一切正常则返回空串
```

### `Qt Query`

在`Qt`中，可直接利用`QSqlQuery`对象操作数据库

```c++
QSqlQuery* query {new QSqlQuery(db)};  // 绑定QSqlDatabase对象即可
query->exec("SQL语句");

// 遍历查询结果
while (query->next()) {
    query->value(列名);  // 返回QVariant, 存储第i条记录中指定列的数据
}

// 也含lastError()方法
```

### `Qt Model/View/Delegate`

模型是为了操作数据库而提供的接口，视图能让用户直观地操作数据库，有两种常用模型和视图

所有模型都有`lastError()`方法

**委托**是模型和视图的中间层，用于**渲染视图**和**编辑模型**

#### `QSqlTableModel/QTableView`

以表为结构的模型/以表形式展现的视图

```c++
// QTableView控件一般在QD中绘制, 这里假设它是ui->tv

// 准备
QSqlTableModel* model {new QSqlTableModel(this, db); } // 指定父组件和数据库
model->setEditStrategy(QSqlTableModel::OnManualSubmit); // 一般要设置手动提交
model->setTable("tableName");  // 模型要设置数据库的表的名称
model->select();     // 将数据加载进模型
ui->tv->setModel(model);   // 视图要绑定模型:表->模型->视图

// 通过模型修改数据库
model->index(int, int)    // 返回QModelIndex对象(0开始)
    rowCount() / columnCount() // 返回当前行/列数
       data(QModelIndex)   // 返回索引指向的数据(类型为QVariant)
       removeRow(int)/removeColumn(int) // 删除某行/列
       submitAll()     // 提交所有修改(真正修改数据库)
       revertAll()     // 撤销所有修改(停止事务)
    
// 通过视图修改模型
ui->tv->currentIndex()    // 返回QModelIndex, 表示用户选中的格子
     setCurrentIndex(QModelIndex)// 设置选中格
// 模型调用方法后, 当前选中的信息可能丢失
    
// 关于QVariant和QModelIndex:
// QVariant含toString()和toInt()等方法, 能将字段转换成QString和int等数据类型
// QModelIndex含row()和column()方法, 返回对象包含的行/列信息
```

关于视图单元格的选中模式(枚举`QAbstractItemView::SelectionMode`)：

- `NoSelection`：无法选中
- `SingleSelection`：只能单选
- `MultiSelection`：允许多选
- `ExtendedSelection`：允许配合`Ctrl`或`Shift`键进行选择
- `ContiguousSelection`：允许配合`Ctrl+Shift`键进行批量选择

如果需要设置某列不可选中，可使用自定义委托：

```c++
class ReadOnlyDelegate: public QItemDelegate {
public:
    ReadOnlyDelegate(QWidget *parent = nullptr) {}

    QWidget *createEditor(QWidget *parent, const QStyleOptionViewItem &option,
                          const QModelIndex &index) const override {
        Q_UNUSED(parent)
        Q_UNUSED(option)
        Q_UNUSED(index)
        return nullptr;
    }
};

// 令委托无视掉指定列的选中
ReadOnlyDelegate* readOnlyDelegate = new ReadOnlyDelegate(this);
for (auto i = 1; i < 3; ++i) { // 无视
    ui->matrixTable->setItemDelegateForColumn(i, readOnlyDelegate);
}
```

#### `QStandardItemModel/QTreeView`

`QStandardItemModel`是以项为基础的模型，主从结构分明，很适合展示**树形结构**

它的编辑要比`QSqlTableView`复杂，不能直接通过`select()`(也没有`setTable()`方法)完成数据的加载，而要使用`QSqlQuery`，因此单纯的矩阵结构应该用后者绘制

```c++
// 准备
...   // 构造函数、设置手动提交、视图和模型绑定 与上述一致
ManageModel->setColumnCount(2);  // 设置模型共有两列
ManageModel->setHorizontalHeaderLabels({"用户组", "组权限"}); // 设置这两列的名称
    
// 例如要绘制二层的树结构, 表group和member如下:
group:
gid  grpname  power  memberCnt
    
member:
uid  grp_id[含group(gid)的外键约束] username pwd

// 加载, 假设有QStandardItemModel对象的指针model:
QSqlQuery group{db}, member{db};
group.exec("SELECT gid, grpname FROM group");
while (group.next()) {
    QList<QStandardItem*> rowItems; // 因设置了两列, 故插入的是链表
    auto tmpItem {new QStandardItem(qroup.value("grpname").toString())};
    // 树的第一层是 组名 和 组权限
    rowItems << tmpItem << new QStandardItem(group.value("power").toInt());
    // 查询当前组的组员
    member.exec("SELECT username FROM member WHERE grp_id=" +
                QString::number(group.value("gid").toInt()));
    while (member.next()) {
        // 增加子项(树的第二层 用户名)
        tmpItem->appendRow(new QStandardItem(member.value("username").toString()));
    }
    model->appendRow(rowItems);
}
// 项一旦加入模型, 所有权将移交, 模型内存释放时所有项一同释放

// 树形视图的方法:
tv->expandAll()    // 展开所有项
 collapseAII()   // 折叠所有项
```

[树形视图的更多方法](https://blog.csdn.net/qq_40597070/article/details/131542928)

[更多模型/视图](https://www.cnblogs.com/zhuchunlin/p/16485926.html)
