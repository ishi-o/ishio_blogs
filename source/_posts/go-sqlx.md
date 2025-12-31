---
title: 'Go: sqlx'
categories:
  - Programming
  - Go
  - 3P Framework
  - Database
tags:
  - Go
  - SQL
  - database
mathjax: false
date: 2025-12-31T14:22:06.000Z
---
<!-- placeholder -->
<!-- more -->

# `sqlx`

## 连接数据库

- `sqlx`是标准库`sql`的扩展库，只要是`sql`支持的数据库，或者说只要是实现了`driver.Driver`接口的数据库都支持
- 注册驱动：需要匿名导入对应的驱动（它们的`init()`里会调用`sql.Register()`），例如：

  ```go
  import _ "github.com/go-sql-driver/mysql"
  ```

- `sqlx.Open(driverName, DSN string) (*DB, error)`：打开一个数据库连接，其中`driverName`为注册时指定的名称，`DSN`为数据源，得到一个`DB`对象：

  ```go
  sqlx.Open("mysql", "root:pwd@tcp(127.0.0.1:3306)/test")
  ```

- `sqlx.Connect(dn, dsn string) (*DB, error)`：与`Open()`类似，但是会主动通过`ping`来验证连接是否有效
- 也可以通过`*DB.Ping()`手动验证
- `*DB.Close()`：关闭数据库连接

## `DB API`

- `*DB.Query(query string, args ...any) (*sql.Rows, error)`：基础查询
- `*DB.Queryx(query string, args ...any) (*sqlx.Rows, error)`：增强查询，返回`sqlx.Rows`
- `*DB.NamedQuery(query string, arg any) (*sqlx.Rows, error)`：增强命名查询，返回`sqlx.Rows`
- `*DB.Get(dest any, query string, args ...any) error`：查询单行记录赋值给结构体，查询结果必须只有一行
- `*DB.Select(dest any, query string, args ...any) error`：查询多行记录赋值给结构体切片
- `*DB.Exec(query string, args ...any) (sql.Result, error)`：执行`SQL`语句，通常是非查询语句如`INSERT`等
- `*DB.NamedExec(query string, arg any) (sql.Result, error)`：命名参数执行，即支持通过`:xxx`而不是`?`传递参数，`arg`可以传递`map`或是结构体
- `*DB.Preparex(query string) (*Stmt, error)`：预编译语句，得到的`Stmt`可重复执行，需要`Close()`
- `*DB.PrepareNamed(query string) (*NamedStmt, error)`：预编译命名语句
- `sql.Result`提供`LastInsertId()`与`RowsAffected()`方法获取主键`id`与影响的行数
- `*DB.XXXContext(...)`：带上下文版本的方法，在开发中建议使用带上下文的查询以实现

## `Stmt`与`NamedStmt`

- 它们需要`Close()`
- 它们是预编译的语句，可以重复使用同时效率比执行相同的`Select()`高效
- 它们的大多数方法的名字和`*DB`的相同，不过不需要`query`参数了

## 查询帮助函数

- `sqlx.In(query string, args ...any) (string, any, error)`：将一个切片展开

  ```go
  ids := []int{1, 2, 3}
  sqlx.In("SELECT * FROM x WHERE id IN (?)", ids)
  // 返回 "SELECT * FROM x WHERE id IN (?, ?, ?)", 1, 2, 3
  ```

- `*DB.Rebind(query string) string`：将一个查询语句翻译成`DB`方言适配的查询

## 事务管理

- `Begin()`：返回`*sql.Tx`
- `Beginx()`：返回`*sqlx.Tx`
- `BeginTx(ctx context.Context, opts *sql.TxOptions)`：带上下文地开启事务，返回`*sql.Tx`
- `BeginTxx(ctx context.Context, opts *sql.TxOptions)`：带上下文地开启事务，返回`*sqlx.Tx`
- `Tx.Rollback()`：回滚

  ```go
  defer func() {
   // 发现有 panic
   if p := recover(); p != nil {
    tx.Rollback()
    panic(p)
   }
  }
  ```

- `Tx.Commit() error`：提交事务
- 在事务内操作使用`Tx`而不是`DB`调用方法即可，其`API`与`DB`类似

