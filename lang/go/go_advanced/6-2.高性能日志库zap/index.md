# 

## 高性能日志库zap

- doc: https://github.com/uber-go/zap

- 1. **安装和基本使用**

  ```go
  // go get -u go.uber.org/zap
  package main
  
  import (
      &#34;go.uber.org/zap&#34;
  )
  
  func main()  {
      logger, _ := zap.NewProduction()
      defer logger.Sync() // flushes buffer, if any
      url := &#34;https://imooc.com&#34;
      sugar := logger.Sugar()
      sugar.Infow(&#34;failed to fetch URL&#34;,
          // Structured context as loosely typed key-value pairs.
          &#34;url&#34;, url,
          &#34;attempt&#34;, 3,
      )
      sugar.Infof(&#34;Failed to fetch URL: %s&#34;, url)
  }
  ```

  &gt; Zap提供了两种类型的日志记录器—`Sugared Logger`和`Logger`。
  &gt;
  &gt; 在性能很好但不是很关键的上下文中，使用`SugaredLogger`。它比其他结构化日志记录包快4-10倍，并且支持结构化和printf风格的日志记录。
  &gt;
  &gt; 在每一微秒和每一次内存分配都很重要的上下文中，使用`Logger`。它甚至比`SugaredLogger`更快，内存分配次数也更少，但它只支持强类型的结构化日志记录

  

- 2. **写入日志文件**

  ```go
  package main
  
  import (
      &#34;go.uber.org/zap&#34;
      &#34;time&#34;
  )
  
  
  func NewLogger() (*zap.Logger, error) {
      cfg := zap.NewProductionConfig()
      cfg.OutputPaths = []string{
          &#34;./myproject.log&#34;, // 输出到文件
        &#34;stdout&#34;,// 输出到控制台
        &#34;stderr&#34;,//输出到控制台err
      }
      return cfg.Build()
  }
  
  func main()  {
      //logger, _ := zap.NewProduction()
      logger, err := NewLogger()
      if err != nil {
          panic(err)
          //panic(&#34;初始化logger失败&#34;)
      }
      su := logger.Sugar()
      defer su.Sync()
      url := &#34;https://imooc.com&#34;
      su.Info(&#34;failed to fetch URL&#34;,
          // Structured context as strongly typed Field values.
          zap.String(&#34;url&#34;, url),
          zap.Int(&#34;attempt&#34;, 3),
          zap.Duration(&#34;backoff&#34;, time.Second),
      )
  }
  ```

- **高级用法**

  ```go
  /*
  1.S()可以获取一个全局的 sugar,可以让我们已设置一个全局的1oger, 而且S()是安全的
  2.日志是分级别的, debug,info,Warn, error, fetal
  3.zap.S()函数和zap.L()函数很有用 -- S()= sugure --L() = logger()
  */
  
  logger, _ := zap.NewDevelopment()
  zap.ReplaceGlobals(logger)
  //	初始化router
  Router := initialize.Routers()
  zap.S().Info(&#34;启动服务端 %d&#34;, port)
  zap.S().Panic(&#34;启动失败:&#34;, err.Error())
  ```


## 案例1-gin

```go
package main

import (
    &#34;net&#34;
    &#34;net/http&#34;
    &#34;net/http/httputil&#34;
    &#34;os&#34;
    &#34;runtime/debug&#34;
    &#34;strings&#34;
    &#34;time&#34;

    &#34;github.com/gin-gonic/gin&#34;
    &#34;github.com/natefinch/lumberjack&#34;
    &#34;go.uber.org/zap&#34;
    &#34;go.uber.org/zap/zapcore&#34;
)

var logger *zap.Logger
var sugarLogger *zap.SugaredLogger

//func main() {
//  InitLogger()
//  defer logger.Sync()
//
//  for i:=0;i&lt;10000;i&#43;&#43;{
//      logger.Info(&#34;test for log rotate...&#34;)
//  }
//  simpleHttpGet(&#34;www.sogo.com&#34;)
//  simpleHttpGet(&#34;http://www.sogo.com&#34;)
//}

func main() {
    InitLogger()
    //r := gin.Default()
    r := gin.New()
    // 这样每次请求都会先走日志函数，先记录路径和请求参数，再调用Next函数，等其他函数执行完再去记录全部的信息
    r.Use(GinLogger(logger), GinRecovery(logger, true))
    r.GET(&#34;/hello&#34;, func(c *gin.Context) {
        c.String(http.StatusOK, &#34;hello q1mi!&#34;)
        logger.Info(&#34;测试日志&#34;)
    })
    r.Run()
}

func InitLogger() {
    writeSyncer := getLogWriter()
    encoder := getEncoder()
    core := zapcore.NewCore(encoder, writeSyncer, zapcore.DebugLevel)

    logger = zap.New(core, zap.AddCaller())
    sugarLogger = logger.Sugar()
}

func getEncoder() zapcore.Encoder {
    //return zapcore.NewJSONEncoder(zap.NewProductionEncoderConfig())
    encoderConfig := zapcore.EncoderConfig{
        TimeKey:        &#34;ts&#34;,
        LevelKey:       &#34;level&#34;,
        NameKey:        &#34;logger&#34;,
        CallerKey:      &#34;caller&#34;,
        MessageKey:     &#34;msg&#34;,
        StacktraceKey:  &#34;stacktrace&#34;,
        LineEnding:     zapcore.DefaultLineEnding,
        EncodeLevel:    zapcore.LowercaseLevelEncoder,
        EncodeTime:     zapcore.ISO8601TimeEncoder,
        EncodeDuration: zapcore.SecondsDurationEncoder,
        EncodeCaller:   zapcore.ShortCallerEncoder,
    }

    return zapcore.NewConsoleEncoder(encoderConfig)
}

//func getLogWriter() zapcore.WriteSyncer {
//  file, _ := os.OpenFile(&#34;./test.log&#34;, os.O_CREATE|os.O_APPEND|os.O_RDWR, 0744)
//  return zapcore.AddSync(file)
//}

func getLogWriter() zapcore.WriteSyncer {
    lumberJackLogger := &amp;lumberjack.Logger{
        Filename:   &#34;./test.log&#34;, // 这里定义保存日志的位置
        MaxSize:    1,     // 在进行切割之前，日志文件的最大大小（以MB为单位）
        MaxBackups: 5,     // 保留旧文件的最大个数
        MaxAge:     30,    // 保留旧文件的最大天数
        Compress:   false, // 是否压缩/归档旧文件
    }
    return zapcore.AddSync(lumberJackLogger)
}

func simpleHttpGet(url string) {
    resp, err := http.Get(url)
    if err != nil {
        sugarLogger.Error(
            &#34;Error fetching url..&#34;,
            zap.String(&#34;url&#34;, url),
            zap.Error(err))
    } else {
        sugarLogger.Info(&#34;Success..&#34;,
            zap.String(&#34;statusCode&#34;, resp.Status),
            zap.String(&#34;url&#34;, url))
        resp.Body.Close()
    }
}

// GinLogger 接收gin框架默认的日志
func GinLogger(logger *zap.Logger) gin.HandlerFunc {
    return func(c *gin.Context) {
        start := time.Now()
        path := c.Request.URL.Path
        query := c.Request.URL.RawQuery
        c.Next()

        cost := time.Since(start)
        logger.Info(path,
            zap.Int(&#34;status&#34;, c.Writer.Status()),
            zap.String(&#34;method&#34;, c.Request.Method),
            zap.String(&#34;path&#34;, path),
            zap.String(&#34;query&#34;, query),
            zap.String(&#34;ip&#34;, c.ClientIP()),
            zap.String(&#34;user-agent&#34;, c.Request.UserAgent()),
            zap.String(&#34;errors&#34;, c.Errors.ByType(gin.ErrorTypePrivate).String()),
            zap.Duration(&#34;cost&#34;, cost),
        )
    }
}

// GinRecovery recover掉项目可能出现的panic
func GinRecovery(logger *zap.Logger, stack bool) gin.HandlerFunc {
    return func(c *gin.Context) {
        defer func() {
            if err := recover(); err != nil {
                // Check for a broken connection, as it is not really a
                // condition that warrants a panic stack trace.
                var brokenPipe bool
                if ne, ok := err.(*net.OpError); ok {
                    if se, ok := ne.Err.(*os.SyscallError); ok {
                        if strings.Contains(strings.ToLower(se.Error()), &#34;broken pipe&#34;) || strings.Contains(strings.ToLower(se.Error()), &#34;connection reset by peer&#34;) {
                            brokenPipe = true
                        }
                    }
                }

                httpRequest, _ := httputil.DumpRequest(c.Request, false)
                if brokenPipe {
                    logger.Error(c.Request.URL.Path,
                        zap.Any(&#34;error&#34;, err),
                        zap.String(&#34;request&#34;, string(httpRequest)),
                    )
                    // If the connection is dead, we can&#39;t write a status to it.
                    c.Error(err.(error)) // nolint: errcheck
                    c.Abort()
                    return
                }

                if stack {
                    logger.Error(&#34;[Recovery from panic]&#34;,
                        zap.Any(&#34;error&#34;, err),
                        zap.String(&#34;request&#34;, string(httpRequest)),
                        zap.String(&#34;stack&#34;, string(debug.Stack())),
                    )
                } else {
                    logger.Error(&#34;[Recovery from panic]&#34;,
                        zap.Any(&#34;error&#34;, err),
                        zap.String(&#34;request&#34;, string(httpRequest)),
                    )
                }
                c.AbortWithStatus(http.StatusInternalServerError)
            }
        }()
        c.Next()
    }
}

```

## 案例2

- logcore

  ```go
  /**
   * 获取日志
   * filePath 日志文件路径
   * level 日志级别
   * maxSize 每个日志文件保存的最大尺寸 单位：M
   * maxBackups 日志文件最多保存多少个备份
   * maxAge 文件最多保存多少天
   * compress 是否压缩
   * serviceName 服务名
   */
  func NewLogger(filePath string, level zapcore.Level, maxSize int, maxBackups int, maxAge int, compress bool, serviceName string) *zap.Logger {
      core := newCore(filePath, level, maxSize, maxBackups, maxAge, compress)
      return zap.New(core, zap.AddCaller(), zap.Development(), zap.Fields(zap.String(&#34;serviceName&#34;, serviceName)))
  }
  
  /**
   * zapcore构造
   */
  func newCore(filePath string, level zapcore.Level, maxSize int, maxBackups int, maxAge int, compress bool) zapcore.Core {
      //日志文件路径配置2
      hook := lumberjack.Logger{
          Filename:   filePath,   // 日志文件路径
          MaxSize:    maxSize,    // 每个日志文件保存的最大尺寸 单位：M
          MaxBackups: maxBackups, // 日志文件最多保存多少个备份
          MaxAge:     maxAge,     // 文件最多保存多少天
          Compress:   compress,   // 是否压缩
      }
      // 设置日志级别
      atomicLevel := zap.NewAtomicLevel()
      atomicLevel.SetLevel(level)
      //公用编码器
      encoderConfig := zapcore.EncoderConfig{
          TimeKey:        &#34;time&#34;,
          LevelKey:       &#34;level&#34;,
          NameKey:        &#34;logger&#34;,
          CallerKey:      &#34;linenum&#34;,
          MessageKey:     &#34;msg&#34;,
          StacktraceKey:  &#34;stacktrace&#34;,
          LineEnding:     zapcore.DefaultLineEnding,
          EncodeLevel:    zapcore.LowercaseLevelEncoder,  // 小写编码器
          EncodeTime:     zapcore.ISO8601TimeEncoder,     // ISO8601 UTC 时间格式
          EncodeDuration: zapcore.SecondsDurationEncoder, //
          EncodeCaller:   zapcore.FullCallerEncoder,      // 全路径编码器
          EncodeName:     zapcore.FullNameEncoder,
      }
      return zapcore.NewCore(
          zapcore.NewJSONEncoder(encoderConfig),                                           // 编码器配置
          zapcore.NewMultiWriteSyncer(zapcore.AddSync(os.Stdout), zapcore.AddSync(&amp;hook)), // 打印到控制台和文件
          atomicLevel,                                                                     // 日志级别
      )
  }
  ```

- log

  ```go
  var MainLogger *zap.Logger
  var GatewayLogger *zap.Logger
  
  func init() {
  
      MainLogger = NewLogger(&#34;./logs/main.log&#34;, zapcore.InfoLevel, 128, 30, 7, true, &#34;Main&#34;)
      GatewayLogger = NewLogger(&#34;./logs/gateway.log&#34;, zapcore.DebugLevel, 128, 30, 7, true, &#34;Gateway&#34;)
  }
  ```

- app

  ```go
  func main() {
      fmt.Println(&#34;init main&#34;)
      log.MainLogger.Debug(&#34;hello main Debug&#34;)
      log.MainLogger.Info(&#34;hello main Info&#34;)
      log.GatewayLogger.Debug(&#34;Hi Gateway Im Debug&#34;)
      log.GatewayLogger.Info(&#34;Hi Gateway  Im Info&#34;)
  }
  ```

  



---

> 作者:   
> URL: http://localhost:1313/lang/go/go_advanced/6-2.%E9%AB%98%E6%80%A7%E8%83%BD%E6%97%A5%E5%BF%97%E5%BA%93zap/  

