# 4-2控制goroutine的数量

# 一. 不控制goroutine数量

```go
package main

import (
    &#34;fmt&#34;
    &#34;math&#34;
    &#34;runtime&#34;
)

func main() {
    //模拟用户需求业务的数量
    task_cnt := math.MaxInt64

    for i := 0; i &lt; task_cnt; i&#43;&#43; {
        go func(i int) {
            //... do some busi...

            fmt.Println(&#34;go func &#34;, i, &#34; goroutine count = &#34;, runtime.NumGoroutine())
        }(i)
    }
}

// panic: too many concurrent operations on a single file or socket (max 1048575)
```

- 我们迅速的开辟goroutine(**不控制并发的 goroutine 数量** )会在短时间内占据操作系统的资源(CPU、内存、文件描述符等)。
  - CPU 使用率浮动上涨
  - Memory 占用不断上涨。
  - 主进程崩溃（被杀掉了）
- 这些资源实际上是所有用户态程序共享的资源，所以大批的goroutine最终引发的灾难不仅仅是自身，还会关联其他运行的程序。



# 二. 控制goroutines数量

## 1. 有缓冲channel与sync同步组合方式

```go
package main

import (
    &#34;fmt&#34;
    &#34;math&#34;
    &#34;sync&#34;
    &#34;runtime&#34;
)

var wg = sync.WaitGroup{}

func busi(ch chan bool, i int) {

    fmt.Println(&#34;go func &#34;, i, &#34; goroutine count = &#34;, runtime.NumGoroutine())

    &lt;-ch

    wg.Done()
}

func main() {
    //模拟用户需求go业务的数量
    task_cnt := math.MaxInt64

    ch := make(chan bool, 3)

    for i := 0; i &lt; task_cnt; i&#43;&#43; {
		wg.Add(1)

        ch &lt;- true

        go busi(ch, i)
    }

	  wg.Wait()
}
```

## 2. 利用无缓冲channel与任务发送/执行分离方式

&gt;生产者消费者模型

```go
package main

import (
    &#34;fmt&#34;
    &#34;math&#34;
    &#34;sync&#34;
    &#34;runtime&#34;
)

var wg = sync.WaitGroup{}

func busi(ch chan int) {

    for t := range ch {
        fmt.Println(&#34;go task = &#34;, t, &#34;, goroutine count = &#34;, runtime.NumGoroutine())
        wg.Done()
    }
}

func sendTask(task int, ch chan int) {
    wg.Add(1)
    ch &lt;- task
}

func main() {

    ch := make(chan int)   //无buffer channel

    goCnt := 3              //启动goroutine的数量
    for i := 0; i &lt; goCnt; i&#43;&#43; {
        //启动go
        go busi(ch)
    }

    taskCnt := math.MaxInt64 //模拟用户需求业务的数量
    for t := 0; t &lt; taskCnt; t&#43;&#43; {
        //发送任务
        sendTask(t, ch)
    }

	  wg.Wait()
}
```

![img](https://gitee.com/big_ox/my_pictrue/raw/master/Typora/6a77f3dfeee80f074b120fd34c96137e_1920x1080.jpeg)

---

> 作者: [Fred](https://github.com/ipfred)  
> URL: https://ipfred.github.io/lang/go/go_advanced/20250515180140/  

