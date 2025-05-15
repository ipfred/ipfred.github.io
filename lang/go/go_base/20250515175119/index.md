# 6-1. 并发与通道

# 并发基础

## 1.并发,并行

```并发和并行都可以处理“多任务”，二者的主要区别在于是否是“同时进行”多个的任务。```

- **并发**:交替做不同事情的能力,**不同的代码块交替执行**

- **并行**:同时做不同事情的能力,**不同的代码块同时执行**

  ```python
  #帮助理解
  并发：老师甲先给学生A去讲思路，A听懂了自己书写过程并且检查，而甲老师在这期间直接去给B讲思路，讲完思路再去给C讲思路，让B自己整理步骤。这样老师就没有空着，一直在做事情，很快就完成了三个任务。与顺序执行不同的是，顺序执行，老师讲完思路之后学生在写步骤，这在这期间，老师是完全空着的，没做事的，所以效率低下。
  并行：直接让三个老师甲、乙、丙三个老师“同时”给三个学生辅导作业，也完成的很快。
  ```

## 2.同步,异步

```同步和异步关注的是--消息通信机制--同步与异步是针对应用程序与内核的交互而言的```

- 同步:就是在发出一个**调用**时，在没有得到结果之前，该**调用**就不返回。但是一旦调用返回，就得到返回值了。
  - 调用者主动等待这个调用的结果。
- 异步:在异步过程**调用**发出后，调用者不会立刻得到结果。而是在*调用*发出后，*被调用者*通过状态、通知来通知调用者.
  - 调用者不会等待这个调用的结果。

```python
#针对IO操作而言
同步过程中进程触发IO操作,并等待或者轮询的去查看IO操作是否完成。
异步过程中进程触发IO操作以后，直接返回，做自己的事情，IO交给内核来处理，完成后内核通知进程IO完成。
```

## 3.阻塞,非阻塞

```同步异步是个操作方式，阻塞非阻塞是线程的一种状态。涉及到CPU线程调度```

- 阻塞:调用结果返回前，线程挂起.
- 非阻塞:调用不会阻塞线程，而且立即返回.

```python
#涉及到CPU线程调度
- 阻塞，就是调用结果返回之前，该执行线程会被挂起，不释放CPU执行权，线程不能做其它事情，只能等待，只有等到调用结果返回了，才能接着往下执行；
- 非阻塞，就是在没有获取调用结果时，不是一直等待，线程可以往下执行，如果是同步的，通过轮询的方式检查有没有调用结果返回，如果是异步的，会通知回调。
```

## 4.**进程、线程、协程**

- 进程，计算机中资源分配的最小单元。 高计算的时候使用多进程
- 线程，计算机中被cpu调度的最小单元。 
- 协程，又称为“微线程”，与进程、线程不同，进程线程是计算机中真实存在，协程是程序员级别人为创造出来的，本质上通过一个线程实现并发的操作。 io多的时候使用线程

```python
进程，计算机中资源分配的最小单元。 高计算的时候使用多进程
线程，计算机中被cpu调度的最小单元。 
协程，又称为“微线程”，与进程、线程不同，进程线程是计算机中真实存在，协程是程序员级别人为创造出来的，本质上通过一个线程实现并发的操作。 io多的时候使用线程

一个进程中可以有多个线程、一个线程中有多个协程，他们都可以帮助我们完成并发操作，特殊协程只有遇到IO切换才有意义，否则效率反倒会降低。 
-----------------
# 进程(process):
	进程是计算机中最小的资源分配单位,创建和销毁都需要一定的开销,进程有自己的pid,进程有三个状态,分别是就绪,运行和阻塞;数据隔离,数据不安全,由操作系统进行控制,可以利用多核;
# 线程(threading):
	线程是cpu最小的调度单位,创建和销毁也需要一定的开销,但是相对进程来说较小,也是由操作系统控制,数据共享,数据不安全,在cpython解析器下不能利用多核,因为gil锁；
# 协程:
	创建和销毁的开销极小,数据共享但是数据安全,不同利用多核,协程是通过代码来实现的
一个进程中可以有多个线程、一个线程中有多个协程，他们都可以帮助我们完成并发操作，特殊协程只有遇到IO切换才有意义，否则效率反倒会降低。 --计算密集型用多进程、IO密集型用多线程。 
    -------
#应用场景:
# 在哪些地方用到了线程和协程
1.自己用线程、协程完成爬虫任务
2.但是后来有了比较丰富的爬虫框架
# 了解到 scrapy /beautyful soup/aiogttp爬虫框架 哪些用到了线程，哪些用到了协程？
3.web框架中的并发是如何实现的
# 传统框架 ： django 多线程
# flask 优先选用协程 其次使用线程
# socketserver ：多线程
# 异步框架 ：tornado，sanic底层都是协程
```

# CSP 并发模型

- CSP(Communication Sequential Process 通讯顺序进程 ) 模型
- CSP 并发模型不关注发送消息的实体,而只关注发送消息时使用的通信管道; 换句话说就是:**CSP模型提倡通过通信来实现内存共享, 而不是通过内存共享而实现通信**
- golang并发基于CSP并发模型, channal 类型的引入就是CSP模型的体现

# goroutine 并发

- golang并发优势:
  - **golang在代码底层实现了并发**, 开发者不用再担心并发的底层逻辑和内存管理, 只需要担心业务逻辑即可
  - golang通过goroutine 实现协程并发编程, 在底层实现了内存共享, 比线程更加易用高效.
- 每一个并发的执行单元都是一个goroutine; 通过使用go关键字实现并发; 一旦使用go关键字, 就不能使用函数的返回值来和主进程进行数据交换, 只能使用**channel**进行数据交换

- 当一个程序启动时, 其主函数就在一个单独的goroutine 中运行,**当main函数后面没有代码逻辑时main函数就会停止, 而所有goroutine在main函数结束时会一并结束!** 终止goroutine的最好方法是在goroutine内部结束goroutine

## 1. runtime包

- runtime包是一个小型的任务调度器, 可以高效的将CPU资源分配给每一个任务

### 1.1 Gosched

- runtime.GOsched() 方法会将当前任务单元放弃处理器, 让其他Go协程运行; 等到其他goroutine结束后就会重启改任务单元;

- 一版goroutine出现以下几种情况, goroutine就会发生调度

  - syscall

  - C函数调用(本质和sysycall类似)

  - 主动调用runtime.Gosched() 方法

  - 某个goroutine调用时间超过100ms, 并且这个goroutine调用了非内联函数

    `内联函数是指当编译器发现某段代码在调用一个内联函数时, 他不是去调用函数而是将该函数的代码整段插入到当前位置, 省去了调用过程, 加快程序运行速度.`

### 2.2 Goexit

- runtime.Goexit() 终止调用他的携程, 但是不会影响其他协程, 并在终止之前会调用defer的函数
- main函数调用此方法main函数结束,程序崩溃

### 3.3 GOMAXPROCS

- **runtime.GOMAXPROCS(n int)函数 可以设置程序在运行中所使用的CPU函数**

- go语言程序默认会使用最大CPU数进行计算:
  - runtime.GOMAXPROCS(n int)设置可同时执行的最大CPU个数,并返回先前的设置. 若n&lt;1, 不会改变当前设置, 本地机器的CPU个数可以通过NumCPU查询

## 2. Sync.WaitGroup

- 等待goroutine结束

  ```go
  package main
  
  import (
      &#34;sync&#34;
  )
  
  type httpPkg struct{}
  
  func (httpPkg) Get(url string) {}
  
  var http httpPkg
  
  func main() {
      var wg sync.WaitGroup
      var urls = []string{
          &#34;http://www.golang.org/&#34;,
          &#34;http://www.google.com/&#34;,
          &#34;http://www.somestupidname.com/&#34;,
      }
      for _, url := range urls {
          // waitgroup计数.
          wg.Add(1)
          // Launch a goroutine to fetch the URL.
          go func(url string) {
              // 函数完事之后告诉wg结束
              defer wg.Done()
              // Fetch the URL.
              http.Get(url)
          }(url)
      }
      // 等待所有的goroutine结束.
      wg.Wait()
  }
  
  ```

  

# channel 通道

&gt;特点: 空读写阻塞，写关闭异常，读关闭空零

- channel是一种特殊的类型, 与map类似; channel可以使用make创建的底层数据结构的引用
- 用于多个goroutine的通信, 内部实现了同步, 保证数据安全

## 1. channel类型

- 创建channal类型

  - ```go
    var 通道变量 chan 通道类型
    
    make(chan Type) //等价于  make(chan Type ,0)  无缓冲阻塞通道 
    make(chan Type, capacity)   // 有缓冲通道
    ```

  - channal的零值是nil

  - **当capacity 的值为0时,  channal 是无缓冲阻塞读写, 当capacity 的值大于0时 ,channal是有缓冲物阻塞读写**

  - channal使用`&lt;-`来接收和发送数据

    | k                  | v                                                            |
    | ------------------ | ------------------------------------------------------------ |
    | channal  &lt;- value  | 发送value值到channal                                         |
    | &lt;-channal          | 接收并将其丢弃                                               |
    | x:= &lt;- channal     | 从channal中接收数据赋值给x                                   |
    | x, ok:= &lt;- channal | 同上,并检查通道是否关闭, 将此状态赋值给ok, 开启是true, 关闭是false |

- 默认情况下, channal是阻塞的, 除非接收端和发送端同时准备好才能完成发送和接收操作

## 2. 缓冲机制

- 通道可以分为有缓冲通道和无缓冲通道;

- **无缓冲通道**在接收之前没有能力保存任何值的通道

  - 无缓冲阻塞, 要求接收方和发送发同时准备好 , 才能完成接收和发送的动作,否则会导致先接收或者发送的goroutine 阻塞等待

  - 接收和发送是同步的, 谁也离不开谁

  - ```go
    package main
    
    import &#34;fmt&#34;
    
    func main() {
    	var cha = make(chan int)
    	go func() {
    		for i := 0; i &lt; 3; i&#43;&#43; {
    			cha &lt;- i
    		}
    	}()
    
    	for  {
    		fmt.Println(&lt;-cha) // 阻塞
    	}
    }
    
    ```

    

- **有缓冲通道**在接收之前就能存储一个或多个值得通道

  - 有缓冲通道不会强制要求goroutine之间必须同时完成接收和发送;
  -  **只有在通道中没有要接收的值得时候接收端才会阻塞; 只有在通道中没有可用缓冲区容纳发送的值时, 发送端就会阻塞**

## 3. close和range

- 当发送者知道没有更多的数据发送时, 让接收者知道没有更多的数据可以接收,可以让接收者停止不必要的等待; 可以通过close和range实现
- **close**关闭通道时注意:
  - channal不能和文件一样去经常关闭,除非你确定没有数据需要传输; 或者想显示的关闭range()之类的才回去关闭channal;
  - **关闭channal之后, 无法再次向channal发送数据**
  - **关闭channal之后,可以继续从channal接收数据**
  - 对于nil的channal, 无论接收和发送都会被阻塞

- **range** 

  ```go
  package main
  
  import &#34;fmt&#34;
  
  func main() {
  	var cha = make(chan int)
  	go func() {
  		for i := 0; i &lt; 3; i&#43;&#43; {
  			cha &lt;- i
  		}
  		close(cha)
  	}()
  
  	for data:= range cha{
  		fmt.Println(data)
  	}
  }
  
  ```

  

## 4. 单向channal

- channal默认是双向的

- 定义单向channal

  ```go
  var cha1 chan int  //双向通道
  var cha1 chan&lt;- int  // 单向接收通道
  var &lt;-chan int  // 单向发送通道
  ```

- 可以将双向channal隐式转换为单向通道, 但是不能将单向channal转为双向channal

- 单向channal的应用: **定时器**

  ```go
  package main
  
  import (
  	&#34;fmt&#34;
  	&#34;time&#34;
  )
  
  func main() {
  	ticker := time.NewTicker(time.Second)
  	for {
  		&lt;- ticker.C
  		fmt.Println(&#34;loop&#34;)
  	}
  }
  ```

  

## 5. select 

- Select 关键字监听channal的数据流动,select 的用法和switch非常相似

- select有较多的限制;其中最大的限制就是每一个case语句中都要是一个I/O操作

  ```go
  select{
    case cha&lt;-:
    	// do
    case &lt;-cha:
    	// do
    default:
    	// do 
  }
  ```

- select 阻塞; 满足条件时会从满足条件中的可执行语句中随机选择一个执行,没有执行default;

- 为了避免长时间阻塞; 使用time.After()执行超时操作

  ```go
  package main
  
  import (
  	&#34;fmt&#34;
  	&#34;time&#34;
  )
  
  func main() {
  	ch  := make(chan int)
  	done := make(chan bool)
  	go func() {
  		for{
  			select {
  			case val:= &lt;-ch:
  				fmt.Println(val)
  			case &lt;- time.After(time.Second*3):
  				fmt.Println(&#34;已超时!&#34;)
  				done &lt;- true
  			}
  
  		}
  	}()
  	for i :=0; i&lt;10	;i&#43;&#43;{
  		ch &lt;-i
  	}
  	&lt;-done
  	fmt.Println(&#34;程序终止&#34;)
  }
  
  /**
  0
  1
  2
  3
  4
  5
  6
  7
  8
  9
  已超时!
  程序终止
  */
  ```

  



---

> 作者: [Fred](https://github.com/ipfred)  
> URL: https://ipfred.github.io/lang/go/go_base/20250515175119/  

