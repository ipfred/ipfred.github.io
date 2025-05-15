# 6-2. channel 原理和坑

## 1. 前言

- channel是Golang在语言层面提供的goroutine间的通信方式，比Unix管道更易用也更轻便。channel主要用于进程内各goroutine间通信，如果需要跨进程通信，建议使用分布式系统的方法来解决。
- channel存在`3种状态`：
  1. **nil，未初始化的状态，只进行了声明，或者手动赋值为`nil`**
  2. **active，正常的channel，可读或者可写**
  3. **closed，已关闭，千万不要误认为关闭channel后，channel的值是nil**

## 2. chan数据结构

`src/runtime/chan.go:hchan`定义了channel的数据结构：

```go
type hchan struct {
    qcount   uint           // 当前队列中剩余元素个数
    dataqsiz uint           // 环形队列长度，即可以存放的元素个数
    buf      unsafe.Pointer // 环形队列指针
    elemsize uint16         // 每个元素的大小
    closed   uint32            // 标识关闭状态
    elemtype *_type         // 元素类型
    sendx    uint           // 队列下标，指示元素写入时存放到队列中的位置
    recvx    uint           // 队列下标，指示元素从队列的该位置读出
    recvq    waitq          // 等待读消息的goroutine队列
    sendq    waitq          // 等待写消息的goroutine队列
    lock mutex              // 互斥锁，chan不允许并发读写
}
```

&gt; 从数据结构可以看出channel由队列、类型信息、goroutine等待队列组成，下面分别说明其原理。

​	

### 2.1 环形队列

chan内部实现了一个环形队列作为其缓冲区，队列的长度是创建chan时指定的。

下图展示了一个可缓存6个元素的channel示意图：

![null](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/m_f48c37e012c38de53aeb532c993b6d2d_r.png)

- dataqsiz指示了队列长度为6，即可缓存6个元素；
- buf指向队列的内存，队列中还剩余两个元素；
- qcount表示队列中还有两个元素；
- sendx指示后续写入的数据存储的位置，取值[0, 6)；
- recvx指示从该位置读取数据, 取值[0, 6)；

### 2.2 等待队列

**从channel读数据，如果channel缓冲区为空或者没有缓冲区，当前goroutine会被阻塞。**
**向channel写数据，如果channel缓冲区已满或者没有缓冲区，当前goroutine会被阻塞。**

被阻塞的goroutine将会挂在channel的等待队列中：

- **因读阻塞的goroutine会被向channel写入数据的goroutine唤醒；**
- **因写阻塞的goroutine会被从channel读数据的goroutine唤醒；**

下图展示了一个没有缓冲区的channel，有几个goroutine阻塞等待读数据：

![null](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/m_f48c37e012c38de53aeb532c993b6d2d_r.png)

注意，一般情况下recvq和sendq至少有一个为空。只有一个例外，那就是同一个goroutine使用select语句向channel一边写数据，一边读数据。

### 2.3 类型信息

一个channel只能传递一种类型的值，类型信息存储在hchan数据结构中。

- elemtype代表类型，用于数据传递过程中的赋值；
- elemsize代表类型大小，用于在buf中定位元素位置。

### 2.4 锁

**一个channel同时仅允许被一个goroutine读写。**

## 3. channel读写和关闭

1. **关闭值为nil的channel**   panic
2. **关闭已经被关闭的channel**  panic
3. **向已经关闭的 channel写数据**  panic
4. 读已经关闭的channel ,如果channel有值则读出,如果没有读出的数据是 channel中可插入类型的零值,第二个返回值一直为false.
5. 如果使用select case 去hold一个已经关闭的channel, 永远都会取到一个channel中可插入类型的零值.
6. for range 如果channel有值则读出, 如果每有值读取已关闭channel 会结束遍历

## 4. select

使用select可以监控多channel，比如监控多个channel，当其中某一个channel有数据时，就从其读出数据。

一个简单的示例程序如下：

```go
package main

import (
    &#34;fmt&#34;
    &#34;time&#34;
)

func addNumberToChan(chanName chan int) {
    for {
        chanName &lt;- 1
        time.Sleep(1 * time.Second)
    }
}

func main() {
    var chan1 = make(chan int, 10)
    var chan2 = make(chan int, 10)

    go addNumberToChan(chan1)
    go addNumberToChan(chan2)

    for {
        select {
        case e := &lt;- chan1 :
            fmt.Printf(&#34;Get element from chan1: %d\n&#34;, e)
        case e := &lt;- chan2 :
            fmt.Printf(&#34;Get element from chan2: %d\n&#34;, e)
        default:
            fmt.Printf(&#34;No element in chan1 and chan2.\n&#34;)
            time.Sleep(1 * time.Second)
        }
    }
}
```

程序中创建两个channel： chan1和chan2。函数addNumberToChan()函数会向两个channel中周期性写入数据。通过select可以监控两个channel，任意一个可读时就从其中读出数据。

**如果使用select case 去hold一个已经关闭的channel, 永远都会取到一个channel中可插入类型的零值.**

---

> 作者: [Fred](https://github.com/ipfred)  
> URL: https://ipfred.github.io/lang/go/go_base/20250515175124/  

