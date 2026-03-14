### 基本使用
- 通过调用`zap.NewProduction()`或`zap.NewDevelopment()`或者`zap.NewExample()`创建一个Logger
- 上面的每一个函数都将创建一个logger。唯一的区别在于它将记录的信息不同。例如production logger默认记录调用函数信息、日期和时间等
- 通过Logger调用Info/Error等
- 默认情况下日志都会打印到应用程序的console界面

```Go
package main

import "go.uber.org/zap"

func dev() {
	logger, _ := zap.NewDevelopment()
	logger.Debug("this is dev debug log")
	logger.Info("this is dev info  log")
	logger.Warn("this is dev warn log")
	logger.Error("this is dev err  log")

	// 下面两条语句执行后会退出程序，即执行完Fatal或Panic之后不会再执行后面的代码
	/*
		logger.Fatal("this is dev fatal log")
		logger.Panic("this is dev panic  log")
	*/

	/*
		2025-03-24T19:34:05.875+0800    DEBUG   test1/main.go:7 this is dev debug log
		2025-03-24T19:34:05.882+0800    INFO    test1/main.go:8 this is dev info  log
		2025-03-24T19:34:05.882+0800    WARN    test1/main.go:9 this is dev warn log
		main.dev
		        C:/code_go/Zap_study/test1/main.go:9
		main.main
		        C:/code_go/Zap_study/test1/main.go:34
		runtime.main
		        C:/Program Files/go/src/runtime/proc.go:272
		2025-03-24T19:34:05.882+0800    ERROR   test1/main.go:10        this is dev err  log
		main.dev
		        C:/code_go/Zap_study/test1/main.go:10
		main.main
		        C:/code_go/Zap_study/test1/main.go:34
		runtime.main
		        C:/Program Files/go/src/runtime/proc.go:272
	*/
}

func example() {
	logger := zap.NewExample()
	logger.Debug("this is example debug log")
	logger.Info("this is example info  log")
	logger.Warn("this is example warn log")
	logger.Error("this is example err  log")

	// 下面两条语句执行后会退出程序，即执行完Fatal或Panic之后不会再执行后面的代码
	/*
		logger.Fatal("this is example fatal log")
		logger.Panic("this is example panic  log")
	*/

	/*
		{"level":"debug","msg":"this is example debug log"}
		{"level":"info","msg":"this is example info  log"}
		{"level":"warn","msg":"this is example warn log"}
		{"level":"error","msg":"this is example err  log"}
	*/
}

func production() {
	logger, _ := zap.NewProduction()
	logger.Debug("this is production debug log")
	logger.Info("this is production info  log")
	logger.Warn("this is production warn log")
	logger.Error("this is production err  log")

	// 下面两条语句执行后会退出程序，即执行完Fatal或Panic之后不会再执行后面的代码
	/*
		logger.Fatal("this is production fatal log")
		logger.Panic("this is production panic  log")
	*/

	/*
		{"level":"info","ts":1742816277.2769861,"caller":"test1/main.go:62","msg":"this is production info  log"}
		{"level":"warn","ts":1742816277.2769861,"caller":"test1/main.go:63","msg":"this is production warn log"}
		{"level":"error","ts":1742816277.2769861,"caller":"test1/main.go:64","msg":"this is production
		 err  log","stacktrace":"main.production\n\tC:/code_go/Zap_study/test1/main.go:64\nmain.main\n
		\tC:/code_go/Zap_study/test1/main.go:76\nruntime.main\n\tC:/Program Files/go/src/runtime/proc.go:272"}
	*/
}

func main() {
	//dev()
	//example()
	production()
}
```


日志记录器方法的语法：
```Go
func (log *Logger) MethodXXX(msg string, field ..Field)
```
其中`MethodXXX`是一个可变参数函数，可以是Info/Error/Debug/Panic等。每个方法都接受一个消息字符串和任意数量的`zapcore.Field`场参数。
每个`zapcore.Field`其实就是一组键值对参数。


### Sugared Logger
- 大部分实现基本都相同


### 定制Logger
#### 将日志写入文件而不是终端
将使用`zap.New(...)`方法来手动传递所有配置，而不是像`zap.NewProduction()`这样的预置方法来创建logger。
```Go
func New(core zapcore.Core, option ...Option) *Logger
```
其中`zapcore.Core`需要三个配置：
- Encoder
- WriterSyncer
- LogLevel

##### Encoder
编码器（如何写入日志）。我们将使用`NewJSONEncoder()`，并使用预先设置的`ProductionEncoderConfig()`.
```Go
zapcore.NewJSONEncoder(zap.NewProductionEncoderConfig())
```

##### WriteSyncer
指定日志将写到哪去。我们使用`zapcore.AddSync()`函数并且将打开的文件句柄传进去。
```Go
file, _ := os.Create("./test.log")
writeSyncer := zapcore.AddSync(file)
```

##### LogLevel
哪种级别的日志将被填入。


```Go
package main

import (
	"go.uber.org/zap"
	"go.uber.org/zap/zapcore"
	"net/http"
	"os"
)

var sugarLogger *zap.SugaredLogger

func InitLogger() {
	writeSyncer := getLogWriter()
	encoder := getEncoder()
	core := zapcore.NewCore(encoder, writeSyncer, zapcore.DebugLevel)

	logger := zap.New(core)
	sugarLogger = logger.Sugar()
}

func getEncoder() zapcore.Encoder {
	//zapcore.NewJSONEncoder(zap.NewDevelopmentEncoderConfig())
	return zapcore.NewConsoleEncoder(zap.NewProductionEncoderConfig())
}

func getLogWriter() zapcore.WriteSyncer {
	file, _ := os.Create("./test2/test.log")
	return zapcore.AddSync(file)
}

func SimpleHttpGet(url string) {
	resp, err := http.Get(url)
	if err != nil {
		sugarLogger.Error(
			"Error fetching url..",
			zap.String("url", url),
			zap.Error(err))
	} else {
		sugarLogger.Info(
			"success..",
			zap.String("url", url),
			zap.Error(err))
		resp.Body.Close()
	}
}

func main() {
	InitLogger()
	defer sugarLogger.Sync()
	SimpleHttpGet("www.baidu.com")
	SimpleHttpGet("http://www.baidu.com")
}
```
```
2025-03-26T17:47:59.100+0800	ERROR	Error fetching url..{url 15 0 www.baidu.com <nil>} {error 26 0  Get "www.baidu.com": unsupported protocol scheme ""}
2025-03-26T17:47:59.192+0800	INFO	success..{url 15 0 http://www.baidu.com <nil>} { 27 0  <nil>}

```


#### 更改时间编码并添加调用者详细信息
- 覆盖默认的`ProductionConfig()`
- 修改时间编码器
- 在日志文件中使用大写字母记录日志级别

```Go
func getEncoder() zapcore.Encoder {
	encoderConfig := zap.NewProductionEncoderConfig()
	encoderConfig.EncodeTime = zapcore.ISO8601TimeEncoder
	encoderConfig.EncodeLevel = zapcore.CapitalLevelEncoder
	return zapcore.NewConsoleEncoder(encoderConfig)
}
```

修改zap logger代码，添加将调用函数信息记录到日志中的功能。为此，可以在`zap.New()`函数中添加一个`Option`。
```Go
logger := zap.New(core, zap.AddCaller())
```
当使用这些修改过的logger配置调用上述部分的`main()`函数时，以下输出将打印在文件`test.log`中。
```
2025-03-26T17:52:17.460+0800	ERROR	test2/main.go:36	Error fetching url..{url 15 0 www.baidu.com <nil>} {error 26 0  Get "www.baidu.com": unsupported protocol scheme ""}
2025-03-26T17:52:17.551+0800	INFO	test2/main.go:41	success..{url 15 0 http://www.baidu.com <nil>} { 27 0  <nil>}

```


#### 使用Lumberjack进行日志切割归档
zap本身不支持切割归档日志文件
为了添加日志切割归档功能，我们将使用第三方库Lumberjack来实现。
##### 安装
`go get gopkg.in/natefinch/lumberjack.v2`
##### zap logger中加入Lumberjack
要在zap中加入Lumberjack支持，我们需要修改`WriterSyncer`代码。
```Go
func getLogWriter() zapcore.WriteSyncer {
	lumberJackLogger := &lumberjack.Logger{
		Filename:   "./test2/test.log",
		MaxSize:    1,     // 单位为M
		MaxBackups: 5,     // 最大备份数量
		MaxAge:     30,    // 最大备份天数
		Compress:   false, // 是否压缩
	}
	return zapcore.AddSync(lumberJackLogger)
}
```


### 使用zap接收gin框架默认的日志并配置日志归档
```Go
package main

import (
	"github.com/gin-gonic/gin"
	"go.uber.org/zap"
	"go.uber.org/zap/zapcore"
	"gopkg.in/natefinch/lumberjack.v2"
	"net"
	"net/http"
	"net/http/httputil"
	"os"
	"runtime/debug"
	"strings"
	"time"
)

var logger *zap.Logger

func InitLogger() {
	writeSyncer := getLogWriter()
	encoder := getEncoder()
	core := zapcore.NewCore(encoder, writeSyncer, zapcore.DebugLevel)

	logger = zap.New(core, zap.AddCaller())
}

func getEncoder() zapcore.Encoder {
	encoderConfig := zap.NewProductionEncoderConfig()
	encoderConfig.EncodeTime = zapcore.ISO8601TimeEncoder
	encoderConfig.EncodeLevel = zapcore.CapitalLevelEncoder
	return zapcore.NewConsoleEncoder(encoderConfig)
}

/*func getLogWriter() zapcore.WriteSyncer {
	file, _ := os.Create("./test2/test.log")
	return zapcore.AddSync(file)
}*/

func getLogWriter() zapcore.WriteSyncer {
	lumberJackLogger := &lumberjack.Logger{
		Filename:   "./test3/test.log",
		MaxSize:    1,     // 单位为M
		MaxBackups: 5,     // 最大备份数量
		MaxAge:     30,    // 最大备份天数
		Compress:   false, // 是否压缩
	}
	return zapcore.AddSync(lumberJackLogger)
}

func SimpleHttpGet(url string) {
	resp, err := http.Get(url)
	if err != nil {
		logger.Error(
			"Error fetching url..",
			zap.String("url", url),
			zap.Error(err))
	} else {
		logger.Info(
			"success..",
			zap.String("url", url),
			zap.Error(err))
		resp.Body.Close()
	}
}

// GinLogger接收gin框架默认的日志
func GinLogger(logger *zap.Logger) gin.HandlerFunc {
	return func(c *gin.Context) {
		start := time.Now()
		path := c.Request.URL.Path
		query := c.Request.URL.RawQuery
		c.Next()

		cost := time.Since(start)
		logger.Info(path,
			zap.Int("status", c.Writer.Status()),
			zap.String("method", c.Request.Method),
			zap.String("path", path),
			zap.String("query", query),
			zap.String("ip", c.ClientIP()),
			zap.String("user-agent", c.Request.UserAgent()),
			zap.String("errors", c.Errors.ByType(gin.ErrorTypePrivate).String()),
			zap.Duration("cost", cost))
	}
}

// GinRecovery recover掉项目可能出现的panic
func GinRecovery(logger *zap.Logger, stack bool) gin.HandlerFunc {
	return func(c *gin.Context) {
		defer func() {
			if err := recover(); err != nil {
				var brokenPipe bool
				if ne, ok := err.(*net.OpError); ok {
					if se, ok := ne.Err.(*os.SyscallError); ok {
						if strings.Contains(strings.ToLower(se.Error()), "broken pipe") || strings.Contains(strings.ToLower(se.Error()), "connection reset by peer") {
							brokenPipe = true
						}
					}
				}

				httpRequest, _ := httputil.DumpRequest(c.Request, false)
				if brokenPipe {
					logger.Error(c.Request.URL.Path,
						zap.Any("error", err),
						zap.String("request", string(httpRequest)),
					)
					c.Error(err.(error))
					c.Abort()
					return
				}

				if stack {
					logger.Error("[Recovery from panic]",
						zap.Any("error", err),
						zap.String("request", string(httpRequest)),
						zap.String("stack", string(debug.Stack())),
					)
				} else {
					logger.Error("[Recovery from panic]",
						zap.Any("error", err),
						zap.String("request", string(httpRequest)),
					)
				}
				c.AbortWithStatus(http.StatusInternalServerError)
			}
		}()
		c.Next()
	}
}

func main() {
	//r := gin.Default()
	InitLogger()
	r := gin.New()
	r.Use(GinLogger(logger), GinRecovery(logger, true))
	r.GET("/hello", func(c *gin.Context) {
		c.String(http.StatusOK, "hello world!")
	})
	r.Run(":8080")
}
```
```
2025-03-26T19:10:44.866+0800	INFO	test3/main.go:75	/hello	{"status": 200, "method": "GET", "path": "/hello", "query": "", "ip": "127.0.0.1", "user-agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/134.0.0.0 Safari/537.36 Edg/134.0.0.0", "errors": "", "cost": 0}
2025-03-26T19:10:45.103+0800	INFO	test3/main.go:75	/favicon.ico	{"status": 404, "method": "GET", "path": "/favicon.ico", "query": "", "ip": "127.0.0.1", "user-agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/134.0.0.0 Safari/537.36 Edg/134.0.0.0", "errors": "", "cost": 0}
```