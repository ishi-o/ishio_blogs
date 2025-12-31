---
title: 'Go: 日志管理'
categories:
  - Programming
  - Go
  - 3P Framework
  - Zap
tags:
  - Go
  - log
mathjax: false
date: 2025-12-30T15:43:40.000Z
---
<!-- placeholder -->
<!-- more -->

# `Go`日志管理

## `log`标准库

### 配置设置

- `log.SetPrefix(stirng)`：设置前缀
- `log.Prefix() string`：获取前缀

### 记录日志

- `log`的封装较为简单，包含以下三类常用方法
- `log.Print("INFO")`：记录信息
- `log.Panic("PANIC")`：记录信息后抛出`panic`
- `log.Fatal("FATAL)`：记录信息后调用`os.exit(1)`
- 它们都有对应的`ln`版本

## `zap`

### `zap`配置定义

- `zap.Config`定义：

  ```go
  import "go.uber.org/zap/zapcore"

  type Config struct {
  	// Level is the minimum enabled logging level.
  	// 运行时修改会修改全部 logger 的 Level
  	Level AtomicLevel `json:"level" yaml:"level"`
  	// 开发模式, zap 在提供默认配置时会参考该值
  	// 调用对应的 NewProductionXxxConfig() / NewDevelopmentXxxConfig()
  	Development bool `json:"development" yaml:"development"`
  	// (默认即可) 默认记录 调用者 所有场景
  	DisableCaller bool `json:"disableCaller" yaml:"disableCaller"`
  	// (默认即可) 默认记录 堆栈 WarnLevel-Development / ErrorLevel-Production
  	DisableStacktrace bool `json:"disableStacktrace" yaml:"disableStacktrace"`
  	// (默认即可) 生产环境下有采样, 用于限流高频日志
  	Sampling *SamplingConfig `json:"sampling" yaml:"sampling"`
  	// 编码 "json" (json风格 建议生产环境) / "console" (控制台风格 建议开发环境)
  	Encoding string `json:"encoding" yaml:"encoding"`
  	// 编码器配置
  	EncoderConfig zapcore.EncoderConfig `json:"encoderConfig" yaml:"encoderConfig"`
  	// 输出路径 "stdout" 表示标准输出 不会自动创建不存在的目录
  	OutputPaths []string `json:"outputPaths" yaml:"outputPaths"`
  	// 错误日志输出路径 "stderr" 表示标准错误流
  	ErrorOutputPaths []string `json:"errorOutputPaths" yaml:"errorOutputPaths"`
  	// 自定义的针对所有日志都默认追加的输出
  	InitialFields map[string]interface{} `json:"initialFields" yaml:"initialFields"`
  }
  ```

- `zapcore.EncoderConfig`定义：

  ```go
  import "io"

  type EncoderConfig struct {
  	// Set the keys used for each log entry. If any key is empty, that portion
  	// of the entry is omitted.
  	// 信息 键
  	MessageKey string `json:"messageKey" yaml:"messageKey"`
  	// 级别 键
  	LevelKey string `json:"levelKey" yaml:"levelKey"`
  	// 时间戳 键
  	TimeKey string `json:"timeKey" yaml:"timeKey"`
  	//
  	NameKey string `json:"nameKey" yaml:"nameKey"`
  	// 调用者 键
  	CallerKey string `json:"callerKey" yaml:"callerKey"`
  	//
  	FunctionKey string `json:"functionKey" yaml:"functionKey"`
  	// 调用栈 键
  	StacktraceKey string `json:"stacktraceKey" yaml:"stacktraceKey"`
  	//
  	SkipLineEnding bool `json:"skipLineEnding" yaml:"skipLineEnding"`
  	//
  	LineEnding string `json:"lineEnding" yaml:"lineEnding"`
  	// Configure the primitive representations of common complex types. For
  	// example, some users may want all time.Times serialized as floating-point
  	// seconds since epoch, while others may prefer ISO8601 strings.
  	// 日志级别 编码器
  	EncodeLevel LevelEncoder `json:"levelEncoder" yaml:"levelEncoder"`
  	// 时间戳 编码器
  	EncodeTime TimeEncoder `json:"timeEncoder" yaml:"timeEncoder"`
  	//
  	EncodeDuration DurationEncoder `json:"durationEncoder" yaml:"durationEncoder"`
  	// 调用者 编码器
  	EncodeCaller CallerEncoder `json:"callerEncoder" yaml:"callerEncoder"`
  	// Unlike the other primitive type encoders, EncodeName is optional. The
  	// zero value falls back to FullNameEncoder.
  	EncodeName NameEncoder `json:"nameEncoder" yaml:"nameEncoder"`
  	// Configure the encoder for interface{} type objects.
  	// If not provided, objects are encoded using json.Encoder
  	NewReflectedEncoder func(io.Writer) ReflectedEncoder `json:"-" yaml:"-"`
  	// Configures the field separator used by the console encoder. Defaults to tab.
  	// 控制台 不同字段的分隔符 默认为 tab
  	ConsoleSeparator string `json:"consoleSeparator" yaml:"consoleSeparator"`
  }
  ```

### `zap`默认配置

- `zap.NewProduction() (*zap.Logger, error)`：获取生产环境下的默认`Logger`实例
- `zap.NewDevelopment() (*zap.Logger, error)`：获取开发环境下的默认`Logger`实例
- `zap.NewProductionConfig() zap.Config`：获取生产环境的默认配置
- `zap.NewDevelopmentConfig() zap.Config`：获取开发环境的默认配置

### 自定义配置(`zapcore`)

- `zap`的默认配置很多时候十分死板，而`zap`提供的获取`Logger`的方法不支持自定义编码器配置与自定义写接口，因此需要使用`zapcore`

  甚至`zap`提供的默认配置无法通过`viper`管理

- `zapcore.NewProductionEncoderConfig() zapcore.EncoderConfig`：获取生产环境的编码器配置，可在此基础上作修改
- `zapcore.NewDevelopmentEncoderConfig() zapcore.EncoderConfig`：获取开发环境的编码器配置
- `zapcore.NewJSONEncoder(EncoderConfig) zapcore.Encoder`：获取输出`JSON`格式的编码器
- `zapcore.NewConsoleEncoder(EncoderConfig) zapcore.Encoder`：获取输出控制台、更易读日志的编码器
- `zapcore.Level`：日志级别，实现了`zapcore.LevelEnabler`接口

  `zap`和`zapcore`提供了日志级别的快捷枚举：
  - `DebugLevel`
  - `InfoLevel`
  - `WarnLevel`
  - `ErrorLevel`
  - `DPanicLevel`
  - `PanicLevel`
  - `FatalLevel`

- `zapcore.AddSync(w io.Writer) zapcore.WriteSyncer`：将一个`io.Writer`包装成`WriteSyncer`
- `zapcore.NewMultiWriteSyncer(ws ...zapcore.WriteSyncer) zapcore.WriteSyncer`：将多个`WriteSyncer`包装成一个`WriteSyncer`，在写入时会同步写入所有的`ws`，需要**多路径输出**时使用
- `zapcore.NewCore(enc zapcore.Encoder, ws zapcore.WriteSyncer, enab zapcore.LevelEnabler) zapcore.Core`：将编码器、写入器、级别限制包装成`zapcore.Core`，需要**分级别输出**时可以创建多个核心
- `zapcore.NewTee(...zapcore.Core) zapcore.Core`：将多个核心包装成一个核心，需要**分级别输出**时使用
- `zap.New(zapcore.Core, ...zap.Option) *zap.Logger`：使用给定的核心创建`Logger`对象，不会有错误抛出

### `lumberjack`

- `lumberjack`只有一个核心用法：以`zap`作为后端，提供日志轮转功能，包括日志过大分割、日志定时丢弃、日志压缩
- 创建`lumberjack.Logger`：

  ```go
  logRotator := &lumberjack.Logger{
    Filename:   "/var/log/myapp/app.log", // 日志文件路径
    MaxSize:    100,    // 单位: MB, 单个文件最大尺寸, 超过即分割
    MaxBackups: 30,     // 保留的旧日志文件最大数量
    MaxAge:     90,     // 单位: 天, 保留旧日志的最大天数
    Compress:   true,   // 是否压缩归档的旧日志文件
    LocalTime:  true,   // 使用本地时间命名旧日志文件 (默认使用 UTC)
  }
  ```

- 通过`zapcore.AddSync(&lumberjack.Logger{...})`将其转换为`zapcore.WriteSyncer`

### 自定义编码器格式

- `EncodeTime`：`type TimeEncoder func(time.Time, zapcore.PrimitiveArrayEncoder)`
- `EncodeLevel`：`type LevelEncoder func(zapcore.Level, zapcore.PrimitiveArrayEncoder)`
- `EncodeDuration`：`type DurationEncoder func(time.Duration, zapcore.PrimitiveArrayEncoder)`
- `EncodeCaller`：`type CallerEncoder func(zapcore.EntryCaller, zapcore.PrimitiveArrayEncoder)`
- `EncodeName`：`type NameEncoder func(string, zapcore.PrimitiveArrayEncoder)`
- 在自定义时，直接调用`zapcore.PrimitiveArrayEncoder`接口的方法，包含一系列`AppendXxx()`方法，其中`Xxx`可以是数值类型和字符串类型

### `zap.Option`

- `zap.AddCaller()`：在日志中添加调用者的文件名和行号
- `zap.AddCallerSkip(skip int)`：设置调用栈信息跳过的层数
- `zap.AddStacktrace(lvl zapcore.Level)`：为指定级别及以上的日志记录堆栈跟踪
- `zap.Fields(fields ...Field)`：向Logger添加固定的上下文字段
- `zap.Development()`：启用开发模式（更易读的输出、禁用采样等）
- `zap.ErrorOutput(w zapcore.WriteSyncer)`：设置内部错误（如编码失败）的输出位置
- `zap.WrapCore(f func(zapcore.Core) zapcore.Core)`：用于包装或替换底层的Core
- `zap.Hooks(hooks ...func(zapcore.Entry) error)`：为每条日志添加钩子函数
- `zap.IncreaseLevel(lvl zapcore.LevelEnabler)`：提高日志记录的最低级别
- `zap.WithClock(clock zapcore.Clock)`：自定义日志时间来源
- `zap.WithFatalHook(hook zapcore.CheckWriteHook)`：为Fatal级别的日志设置钩子
- `zap.WithPanicHook(hook func(entry zapcore.Entry) error)`：为Panic级别的日志设置钩子

### `Logger`使用

- `Logger`本身很简单，对应每种级别有对应的方法（七种级别），`Logger`的性能较好，方法签名如下：

  `Xxx(msg string, ...zap.Field)`：本条日志的信息是`msg`，后面附带一系列字段（强类型）

- `zap.Field`：

  ```go
  const (
  	// UnknownType is the default field type. Attempting to add it to an encoder will panic.
  	UnknownType FieldType = iota
  	// ArrayMarshalerType indicates that the field carries an ArrayMarshaler.
  	ArrayMarshalerType
  	// ObjectMarshalerType indicates that the field carries an ObjectMarshaler.
  	ObjectMarshalerType
  	// BinaryType indicates that the field carries an opaque binary blob.
  	BinaryType
  	// BoolType indicates that the field carries a bool.
  	BoolType
  	// ByteStringType indicates that the field carries UTF-8 encoded bytes.
  	ByteStringType
  	// Complex128Type indicates that the field carries a complex128.
  	Complex128Type
  	// Complex64Type indicates that the field carries a complex64.
  	Complex64Type
  	// DurationType indicates that the field carries a time.Duration.
  	DurationType
  	// Float64Type indicates that the field carries a float64.
  	Float64Type
  	// Float32Type indicates that the field carries a float32.
  	Float32Type
  	// Int64Type indicates that the field carries an int64.
  	Int64Type
  	// Int32Type indicates that the field carries an int32.
  	Int32Type
  	// Int16Type indicates that the field carries an int16.
  	Int16Type
  	// Int8Type indicates that the field carries an int8.
  	Int8Type
  	// StringType indicates that the field carries a string.
  	StringType
  	// TimeType indicates that the field carries a time.Time that is
  	// representable by a UnixNano() stored as an int64.
  	TimeType
  	// TimeFullType indicates that the field carries a time.Time stored as-is.
  	TimeFullType
  	// Uint64Type indicates that the field carries a uint64.
  	Uint64Type
  	// Uint32Type indicates that the field carries a uint32.
  	Uint32Type
  	// Uint16Type indicates that the field carries a uint16.
  	Uint16Type
  	// Uint8Type indicates that the field carries a uint8.
  	Uint8Type
  	// UintptrType indicates that the field carries a uintptr.
  	UintptrType
  	// ReflectType indicates that the field carries an interface{}, which should
  	// be serialized using reflection.
  	ReflectType
  	// NamespaceType signals the beginning of an isolated namespace. All
  	// subsequent fields should be added to the new namespace.
  	NamespaceType
  	// StringerType indicates that the field carries a fmt.Stringer.
  	StringerType
  	// ErrorType indicates that the field carries an error.
  	ErrorType
  	// SkipType indicates that the field is a no-op.
  	SkipType

  	// InlineMarshalerType indicates that the field carries an ObjectMarshaler
  	// that should be inlined.
  	InlineMarshalerType
  )
  ```

  每种类型都对应一个方法，`Xxx(key string, value Type) zap.Field`，需要提供键、对应的强类型的值

- `Logger.Sugared() *zap.SugaredLogger`：获取一个**性能较差**，但是支持`log`标准库风格的日志器，即`SugaredLogger`提供`Xxx(string)`和`Xxxf(string, ...any)`

### 例子配置

- `viper`配置映射及级别配置：

  ```go
  import "go.uber.org/zap/zapcore"

  const (
  	DebugLevel   = "debug"
  	InfoLevel    = "info"
  	FatalLevel   = "fatal"
  	WarnLevel    = "warn"
  	ErrorLevel   = "error"
  	PanicLevel   = "panic"
  	DefaultLevel = "debug"
  )

  type ZapConfig struct {
  	Level            string         `mapstruct:"level"`
  	Development      bool           `mapstruct:"development"`
  	Encoding         string         `mapstruct:"encoding"`
  	OutputPaths      []string       `mapstruct:"outputPaths"`
  	ErrorOutputPaths []string       `mapstruct:"errorOutputPaths"`
  	InitialFields    map[string]any `mapstruct:"initialFields"`
  }

  func zapLevelEnabler(cfg *config.ZapConfig) zapcore.LevelEnabler {
  	level, ok := levelMap[cfg.Level]
  	if ok {
  		return level
  	} else {
  		return levelMap[config.DefaultLevel]
  	}
  }
  ```

- 配置编码器，这里使用的就是默认的

  ```go
  import (
  	"go.uber.org/zap"
  	"go.uber.org/zap/zapcore"
  )

  func zapEncoderConfig(cfg *config.ZapConfig) zapcore.EncoderConfig {
  	var encoderCfg zapcore.EncoderConfig
  	if cfg.Development {
  		encoderCfg = zap.NewDevelopmentEncoderConfig()
  	} else {
  		encoderCfg = zap.NewProductionEncoderConfig()
  	}
  	return encoderCfg
  }

  func zapEncoder(cfg *config.ZapConfig) zapcore.Encoder {
  	encoderCfg := zapEncoderConfig(cfg)
  	switch cfg.Encoding {
  	case "json":
  		return zapcore.NewJSONEncoder(encoderCfg)
  	case "console":
  		return zapcore.NewConsoleEncoder(encoderCfg)
  	default:
  		return zapcore.NewConsoleEncoder(encoderCfg)
  	}
  }
  ```

- `WriteSyncer`配置：

  ```go
  import (
  	"os"
  	"strings"

  	"github.com/natefinch/lumberjack"
  	"go.uber.org/zap/zapcore"
  )

  func newWriterSyncer(paths []string) zapcore.WriteSyncer {
  	syncers := make([]zapcore.WriteSyncer, 0, len(paths))
  	for _, path := range paths {
  		var ws zapcore.WriteSyncer
  		switch {
  		case strings.ToLower(path) == "stdout":
  			ws = zapcore.AddSync(os.Stdout)
  		case strings.ToLower(path) == "stderr":
  			ws = zapcore.AddSync(os.Stderr)
  		default:
  			lumberjackLogger := &lumberjack.Logger{
  				Filename:   path,
  				MaxSize:    100,
  				MaxBackups: 30,
  				MaxAge:     90,
  				Compress:   true,
  			}
  			ws = zapcore.AddSync(lumberjackLogger)
  		}
  		syncers = append(syncers, ws)
  	}
  	return zapcore.NewMultiWriteSyncer(syncers...)
  }

  func zapWriterSyncer(cfg *config.ZapConfig) (zapcore.WriteSyncer, zapcore.WriteSyncer) {
  	return newWriterSyncer(cfg.OutputPaths), newWriterSyncer(cfg.ErrorOutputPaths)
  }
  ```

