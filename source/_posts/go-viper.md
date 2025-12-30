---
title: 'Go: Viper配置管理'
categories:
  - Programming
  - Go
  - 3P Framework
  - Viper
tags:
  - Go
  - config
  - webapp
mathjax: false
date: 2025-12-27T14:58:39.000Z
---
<!-- placeholder -->
<!-- more -->

# `Viper`配置管理

## 配置优先级

- `Viper.Set()`设置的值
- 命令行中传入的参数
- 环境变量
- 配置文件
- 键值存储
- `Viper.SetDefault()`设置的值

## 读取配置

### 从文件读取

- ```go
  viper.SetConfigName("config")
  viper.SetConfigType("yaml")
  viper.AddConfigPath("/etc/appname/")
  viper.AddConfigPath("$HOME/.appname")
  viper.AddConfigPath(".")
  err := viper.ReadInConfig()
  if err != nil {
    panic(fmt.Errorf("fatal error config file%w", err))
  }
  ```

- `SetConfigName()`：设置配置文件名，若不包含扩展名，需要设置`SetConfigType()`
- `SetConfigType()`：设置文件扩展名
- `AddConfigPath()`：设置配置文件搜索路径，先加入的优先级较低，支持使用环境变量
- `ReadInConfig() error`：搜索并读取配置

### 设置默认值

- `SetDefault(k, v)`

### 访问配置

- `Get(key string) any`
- `GetBool(key string) bool`
- `GetFloat64(key string) float64`
- `GetInt(key string) int`
- `GetIntSlice(key string) []int`
- `GetString(key string) string`
- `GetStringMap(key string) map[string]any`
- `GetStringMapString(key string) map[string]string`
- `GetStringSlice(key string) []string`
- `GetTime(key string) time.Time`
- `GetDuration(key string) time.Duration`
- `IsSet(key string) bool`
- `AllSettings() map[string]any`
- 键的构造：`viper`对大小写敏感，通常使用`.`来表示嵌套的子配置，如果存在环境变量，则会被隐式地替换为大写字母和`_`分隔

  例如`default.filepath`会先检查`DEFAULT_FILEPATH`，然后检查配置文件中`default`字段的子字段`filepath`

### 反序列化

- `Unmarshal(rawVal any) error`：传入一个指针，将配置反序列化给该指针
- `UnmarshalKey(key string, rawVal any) error`：仅反序列化某个键的配置
- 针对第一种反序列化，基本上都是传递一个匹配的结构体的指针，通常显式地指定`mapstruct`标签

  ```go
  type Conf struct {
  	FilePath string `mapstruct:"file_path"`
  }
  ```

## 监控配置变化与动态读取

- ```go
  viper.OnConfigChane(func(e fsnotify.Event) {
    // doSth
  })
  // 或
  viper.WatchConfig()
  ```

- `AutomaticEnv()`：
- 开发阶段常用

