# 11. Context

# Context

- 在 Go http包的Server中，每一个请求在都有一个对应的 goroutine 去处理。请求处理函数通常会启动额外的 goroutine 用来访问后端服务，比如数据库和RPC服务。用来处理一个请求的 goroutine 通常需要访问一些与请求特定的数据，比如终端用户的身份认证信息、验证相关的token、请求的截止时间。 当一个请求被取消或超时时，所有用来处理该请求的 goroutine 都应该迅速退出，然后系统才能释放这些 goroutine 占用的资源。

## 为什么需要Context

- 下面以取消一个goroutine做示例说明为什么需要Context

### 1. 基本示例

```go
package main

import (
	&#34;fmt&#34;
	&#34;sync&#34;

	&#34;time&#34;
)

var wg sync.WaitGroup

// 初始的例子

func worker() {
	for {
		fmt.Println(&#34;worker&#34;)
		time.Sleep(time.Second)
	}
	// 如何接收外部命令实现退出
	wg.Done()
}

func main() {
	wg.Add(1)
	go worker()
	// 如何优雅的实现结束子goroutine
	wg.Wait()
	fmt.Println(&#34;over&#34;)
}
```

### 2. 全局变量方式

```go
package main

import (
	&#34;fmt&#34;
	&#34;sync&#34;

	&#34;time&#34;
)

var wg sync.WaitGroup
var exit bool

// 全局变量方式存在的问题：
// 1. 使用全局变量在跨包调用时不容易统一
// 2. 如果worker中再启动goroutine，就不太好控制了。

func worker() {
	for {
		fmt.Println(&#34;worker&#34;)
		time.Sleep(time.Second)
		if exit {
			break
		}
	}
	wg.Done()
}

func main() {
	wg.Add(1)
	go worker()
	time.Sleep(time.Second * 3) // sleep3秒以免程序过快退出
	exit = true                 // 修改全局变量实现子goroutine的退出
	wg.Wait()
	fmt.Println(&#34;over&#34;)
}
```

### 3. 通道方式

```go
package main

import (
	&#34;fmt&#34;
	&#34;sync&#34;

	&#34;time&#34;
)

var wg sync.WaitGroup

// 管道方式存在的问题：
// 1. 使用全局变量在跨包调用时不容易实现规范和统一，需要维护一个共用的channel

func worker(exitChan chan struct{}) {
LOOP:
	for {
		fmt.Println(&#34;worker&#34;)
		time.Sleep(time.Second)
		select {
		case &lt;-exitChan: // 等待接收上级通知
			break LOOP
		default:
		}
	}
	wg.Done()
}

func main() {
	var exitChan = make(chan struct{})
	wg.Add(1)
	go worker(exitChan)
	time.Sleep(time.Second * 3) // sleep3秒以免程序过快退出
	exitChan &lt;- struct{}{}      // 给子goroutine发送退出信号
	close(exitChan)
	wg.Wait()
	fmt.Println(&#34;over&#34;)
}
```

### 4. 官方版的方案

```go
package main

import (
	&#34;context&#34;
	&#34;fmt&#34;
	&#34;sync&#34;

	&#34;time&#34;
)

var wg sync.WaitGroup

func worker(ctx context.Context) {
LOOP:
	for {
		fmt.Println(&#34;worker&#34;)
		time.Sleep(time.Second)
		select {
		case &lt;-ctx.Done(): // 等待上级通知
			break LOOP
		default:
		}
	}
	wg.Done()
}

func main() {
	ctx, cancel := context.WithCancel(context.Background())
	wg.Add(1)
	go worker(ctx)
	time.Sleep(time.Second * 3)
	cancel() // 通知子goroutine结束
	wg.Wait()
	fmt.Println(&#34;over&#34;)
}
```

当子goroutine又开启另外一个goroutine时，只需要将ctx传入即可：

```go
package main

import (
	&#34;context&#34;
	&#34;fmt&#34;
	&#34;sync&#34;

	&#34;time&#34;
)

var wg sync.WaitGroup

func worker(ctx context.Context) {
	go worker2(ctx)
LOOP:
	for {
		fmt.Println(&#34;worker&#34;)
		time.Sleep(time.Second)
		select {
		case &lt;-ctx.Done(): // 等待上级通知
			break LOOP
		default:
		}
	}
	wg.Done()
}

func worker2(ctx context.Context) {
LOOP:
	for {
		fmt.Println(&#34;worker2&#34;)
		time.Sleep(time.Second)
		select {
		case &lt;-ctx.Done(): // 等待上级通知
			break LOOP
		default:
		}
	}
}
func main() {
	ctx, cancel := context.WithCancel(context.Background())
	wg.Add(1)
	go worker(ctx)
	time.Sleep(time.Second * 3)
	cancel() // 通知子goroutine结束
	wg.Wait()
	fmt.Println(&#34;over&#34;)
}
```

## Context初识

- Go1.7加入了一个新的标准库`context`，它定义了`Context`类型，**专门用来简化 对于处理单个请求的多个 goroutine 之间与请求域的数据、取消信号、截止时间等相关操作**，这些操作可能涉及多个 API 调用。

- 对服务器传入的请求应该创建上下文，而对服务器的传出调用应该接受上下文。它们之间的函数调用链必须传递上下文，或者可以使用`WithCancel`、`WithDeadline`、`WithTimeout`或`WithValue`创建的派生上下文。当一个上下文被取消时，它派生的所有上下文也被取消。

## Context接口

`context.Context`是一个接口，该接口定义了四个需要实现的方法。具体签名如下：

```go
type Context interface {
    Deadline() (deadline time.Time, ok bool)
    Done() &lt;-chan struct{}
    Err() error
    Value(key interface{}) interface{}
}
```

- `Deadline`方法需要返回当前`Context`被取消的时间，也就是完成工作的截止时间（deadline）；
- `Done`方法需要返回一个`Channel`，这个Channel会在当前工作完成或者上下文被取消之后关闭，多次调用`Done`方法会返回同一个Channel；
- `Err`方法会返回当前`Context`结束的原因，它只会在`Done`返回的Channel被关闭时才会返回非空的值；
  - 如果当前`Context`被取消就会返回`Canceled`错误；
  - 如果当前`Context`超时就会返回`DeadlineExceeded`错误；
- `Value`方法会从`Context`中返回键对应的值，对于同一个上下文来说，多次调用`Value` 并传入相同的`Key`会返回相同的结果，该方法仅用于传递跨API和进程间跟请求域的数据；

### 1. Background()和TODO()

Go内置两个函数：`Background()`和`TODO()`，这两个函数分别返回一个实现了`Context`接口的`background`和`todo`。我们代码中最开始都是以这两个内置的上下文对象作为最顶层的`partent context`，衍生出更多的子上下文对象。

- `Background()`主要用于main函数、初始化以及测试代码中，作为Context这个树结构的最顶层的Context，也就是根Context。

- `TODO()`，它目前还不知道具体的使用场景，如果我们不知道该使用什么Context的时候，可以使用这个。

- `background`和`todo`本质上都是`emptyCtx`结构体类型，是一个不可取消，没有设置截止时间，没有携带任何值的Context。

## With系列函数

### 1. WithCancel

- `WithCancel`的函数签名如下：

```go
func WithCancel(parent Context) (ctx Context, cancel CancelFunc)
```

- `WithCancel`返回带有新Done通道的父节点的副本。当调用返回的cancel函数或当关闭父上下文的Done通道时，将关闭返回上下文的Done通道，无论先发生什么情况。

- 取消此上下文将释放与其关联的资源，因此代码应该在此上下文中运行的操作完成后立即调用cancel。

```go
func gen(ctx context.Context) &lt;-chan int {
		dst := make(chan int)
		n := 1
		go func() {
			for {
				select {
				case &lt;-ctx.Done():
					return // return结束该goroutine，防止泄露
				case dst &lt;- n:
					n&#43;&#43;
				}
			}
		}()
		return dst
	}
func main() {
	ctx, cancel := context.WithCancel(context.Background())
	defer cancel() // 当我们取完需要的整数后调用cancel

	for n := range gen(ctx) {
		fmt.Println(n)
		if n == 5 {
			break
		}
	}
}
```

- 上面的示例代码中，`gen`函数在单独的goroutine中生成整数并将它们发送到返回的通道。 gen的调用者在使用生成的整数之后需要取消上下文，以免`gen`启动的内部goroutine发生泄漏。

### 2. WithDeadline

- `WithDeadline`的函数签名如下：

```go
func WithDeadline(parent Context, deadline time.Time) (Context, CancelFunc)
```

- 返回父上下文的副本，并将deadline调整为不迟于d。如果父上下文的deadline已经早于d，则WithDeadline(parent, d)在语义上等同于父上下文。当截止日过期时，当调用返回的cancel函数时，或者当父上下文的Done通道关闭时，返回上下文的Done通道将被关闭，以最先发生的情况为准。

- 取消此上下文将释放与其关联的资源，因此代码应该在此上下文中运行的操作完成后立即调用cancel。

```go
func main() {
	d := time.Now().Add(50 * time.Millisecond)
	ctx, cancel := context.WithDeadline(context.Background(), d)

	// 尽管ctx会过期，但在任何情况下调用它的cancel函数都是很好的实践。
	// 如果不这样做，可能会使上下文及其父类存活的时间超过必要的时间。
	defer cancel()

	select {
	case &lt;-time.After(1 * time.Second):
		fmt.Println(&#34;overslept&#34;)
	case &lt;-ctx.Done():
		fmt.Println(ctx.Err())
	}
}
```

- 上面的代码中，定义了一个50毫秒之后过期的deadline，然后我们调用`context.WithDeadline(context.Background(), d)`得到一个上下文（ctx）和一个取消函数（cancel），然后使用一个select让主程序陷入等待：等待1秒后打印`overslept`退出或者等待ctx过期后退出。 因为ctx50秒后就过期，所以`ctx.Done()`会先接收到值，上面的代码会打印ctx.Err()取消原因。

### 3. WithTimeout

- `WithTimeout`的函数签名如下：

```go
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc)
```

`WithTimeout`返回`WithDeadline(parent, time.Now().Add(timeout))`。

- 取消此上下文将释放与其相关的资源，因此代码应该在此上下文中运行的操作完成后立即调用cancel，通常用于数据库或者网络连接的超时控制。具体示例如下：

```go
package main

import (
	&#34;context&#34;
	&#34;fmt&#34;
	&#34;sync&#34;

	&#34;time&#34;
)

// context.WithTimeout

var wg sync.WaitGroup

func worker(ctx context.Context) {
LOOP:
	for {
		fmt.Println(&#34;db connecting ...&#34;)
		time.Sleep(time.Millisecond * 10) // 假设正常连接数据库耗时10毫秒
		select {
		case &lt;-ctx.Done(): // 50毫秒后自动调用
			break LOOP
		default:
		}
	}
	fmt.Println(&#34;worker done!&#34;)
	wg.Done()
}

func main() {
	// 设置一个50毫秒的超时
	ctx, cancel := context.WithTimeout(context.Background(), time.Millisecond*50)
	wg.Add(1)
	go worker(ctx)
	time.Sleep(time.Second * 5)
	cancel() // 通知子goroutine结束
	wg.Wait()
	fmt.Println(&#34;over&#34;)
}
```

### 4. WithValue

- `WithValue`函数能够将请求作用域的数据与 Context 对象建立关系。声明如下：

```go
func WithValue(parent Context, key, val interface{}) Context
```

`WithValue`返回父节点的副本，其中与key关联的值为val。

- 仅对API和进程间传递请求域的数据使用上下文值，而不是使用它来传递可选参数给函数。

- 所提供的键必须是可比较的，并且不应该是`string`类型或任何其他内置类型，以避免使用上下文在包之间发生冲突。`WithValue`的用户应该为键定义自己的类型。为了避免在分配给interface{}时进行分配，上下文键通常具有具体类型`struct{}`。或者，导出的上下文关键变量的静态类型应该是指针或接口。

```go
package main

import (
	&#34;context&#34;
	&#34;fmt&#34;
	&#34;sync&#34;

	&#34;time&#34;
)

// context.WithValue

type TraceCode string

var wg sync.WaitGroup

func worker(ctx context.Context) {
	key := TraceCode(&#34;TRACE_CODE&#34;)
	traceCode, ok := ctx.Value(key).(string) // 在子goroutine中获取trace code
	if !ok {
		fmt.Println(&#34;invalid trace code&#34;)
	}
LOOP:
	for {
		fmt.Printf(&#34;worker, trace code:%s\n&#34;, traceCode)
		time.Sleep(time.Millisecond * 10) // 假设正常连接数据库耗时10毫秒
		select {
		case &lt;-ctx.Done(): // 50毫秒后自动调用
			break LOOP
		default:
		}
	}
	fmt.Println(&#34;worker done!&#34;)
	wg.Done()
}

func main() {
	// 设置一个50毫秒的超时
	ctx, cancel := context.WithTimeout(context.Background(), time.Millisecond*50)
	// 在系统的入口中设置trace code传递给后续启动的goroutine实现日志数据聚合
	ctx = context.WithValue(ctx, TraceCode(&#34;TRACE_CODE&#34;), &#34;12512312234&#34;)
	wg.Add(1)
	go worker(ctx)
	time.Sleep(time.Second * 5)
	cancel() // 通知子goroutine结束
	wg.Wait()
	fmt.Println(&#34;over&#34;)
}
```

## 使用Context的注意事项

- 推荐以参数的方式显示传递Context
- 以Context作为参数的函数方法，应该把Context作为第一个参数。
- 给一个函数方法传递Context的时候，不要传递nil，如果不知道传递什么，就使用context.TODO()
- Context的Value相关方法应该传递请求域的必要数据，不应该用于传递可选参数
- Context是线程安全的，可以放心的在多个goroutine中传递

## 客户端超时取消示例(小爬虫)

调用服务端API时如何在客户端实现超时控制？

### server

```go
// context_timeout/server/main.go
package main

import (
	&#34;fmt&#34;
	&#34;math/rand&#34;
	&#34;net/http&#34;

	&#34;time&#34;
)

// server端，随机出现慢响应

func indexHandler(w http.ResponseWriter, r *http.Request) {
	number := rand.Intn(2)
	if number == 0 {
		time.Sleep(time.Second * 10) // 耗时10秒的慢响应
		fmt.Fprintf(w, &#34;slow response&#34;)
		return
	}
	fmt.Fprint(w, &#34;quick response&#34;)
}

func main() {
	http.HandleFunc(&#34;/&#34;, indexHandler)
	err := http.ListenAndServe(&#34;:8000&#34;, nil)
	if err != nil {
		panic(err)
	}
}
```

### client

```go
// context_timeout/client/main.go
package main

import (
	&#34;context&#34;
	&#34;fmt&#34;
	&#34;io/ioutil&#34;
	&#34;net/http&#34;
	&#34;sync&#34;
	&#34;time&#34;
)

// 客户端

type respData struct {
	resp *http.Response
	err  error
}

func doCall(ctx context.Context) {
	transport := http.Transport{
	   // 请求频繁可定义全局的client对象并启用长链接
	   // 请求不频繁使用短链接
	   DisableKeepAlives: true, 	}
	client := http.Client{
		Transport: &amp;transport,
	}

	respChan := make(chan *respData, 1)
	req, err := http.NewRequest(&#34;GET&#34;, &#34;http://127.0.0.1:8000/&#34;, nil)
	if err != nil {
		fmt.Printf(&#34;new requestg failed, err:%v\n&#34;, err)
		return
	}
	req = req.WithContext(ctx) // 使用带超时的ctx创建一个新的client request
	var wg sync.WaitGroup
	wg.Add(1)
	defer wg.Wait()
	go func() {
		resp, err := client.Do(req)
		fmt.Printf(&#34;client.do resp:%v, err:%v\n&#34;, resp, err)
		rd := &amp;respData{
			resp: resp,
			err:  err,
		}
		respChan &lt;- rd
		wg.Done()
	}()

	select {
	case &lt;-ctx.Done():
		//transport.CancelRequest(req)
		fmt.Println(&#34;call api timeout&#34;)
	case result := &lt;-respChan:
		fmt.Println(&#34;call server api success&#34;)
		if result.err != nil {
			fmt.Printf(&#34;call server api failed, err:%v\n&#34;, result.err)
			return
		}
		defer result.resp.Body.Close()
		data, _ := ioutil.ReadAll(result.resp.Body)
		fmt.Printf(&#34;resp:%v\n&#34;, string(data))
	}
}

func main() {
	// 定义一个100毫秒的超时
	ctx, cancel := context.WithTimeout(context.Background(), time.Millisecond*100)
	defer cancel() // 调用cancel释放子goroutine资源
	doCall(ctx)
}
```

## 坑

### 1. WithCancel

- withCancel的使用要非常明确程序什么时候被取消
- 由于go大量的官方库、第三方库使用了context，所以调用`接收context的函数`时要小心，要清楚context在什么时候cancel，什么行为会触发cancel

### 2. WithValue

- 所提供的键必须是可比较的，并且不应该是`string`类型或任何其他内置类型，以避免使用上下文在包之间发生冲突。`WithValue`的用户应该为键定义自己的类型。为了避免在分配给interface{}时进行分配，上下文键通常具有具体类型`struct{}`。或者，导出的上下文关键变量的静态类型应该是指针或接口。

  ```go
  type TraceCode string
  func main() {
  	ctx := context.Background()
  	ctx = context.WithValue(ctx, TraceCode(&#34;TRACE_CODE&#34;), &#34;1&#34;)
  	fmt.Println(ctx.Value(&#34;TRACE_CODE&#34;)) // nil
  	fmt.Println(ctx.Value(TraceCode(&#34;TRACE_CODE&#34;))) // 1
  }
  ```

### 3. context 函数传递是值传递

- error demo

  ```go
  type ctxType string
  
  func main() {
  	ctx := context.WithValue(context.Background(), ctxType(&#34;key&#34;), 1)
  	valueChange(ctx)
  	log.Println(ctx.Value(ctxType(&#34;key&#34;))) // 1; 并不会更改值
  }
  
  func valueChange(ctx context.Context) {
  	context.WithValue(ctx, ctxType(&#34;key&#34;), 2)
  }
  ```

  


---

> 作者: [Fred](https://github.com/ipfred)  
> URL: https://ipfred.github.io/lang/go/go_base/20250515175139/  

