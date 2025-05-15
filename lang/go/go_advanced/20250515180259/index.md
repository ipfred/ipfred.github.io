# 16. Go runtime详解

&gt; runtime 包 提供了运行时与系统的交互，比如控制协程函数，触发垃圾立即回收等等底层操作;

## runtime.GOARCH 

&gt; 获取 GOARCH 信息

```go
fmt.Println(runtime.GOARCH)   // arm64
```

## runtime.GOOS

&gt; 获取 GOOS 信息

```go
fmt.Println(runtime.GOOS)     // darwin
```

## runtime.GOROOT()

&gt; 获取goroot环境变量  `func GOROOT() string`

```go
package main

import (
	&#34;fmt&#34;
	&#34;runtime&#34;
)

func main() {
	fmt.Println(runtime.GOROOT()) // /Users/liusaisai/.g/go
	fmt.Println(runtime.GOARCH)   // arm64
	fmt.Println(runtime.GOOS)     // darwin
}
```

## runtime.Version()

&gt; 获取go版本

```js
package main

import (
	&#34;fmt&#34;
	&#34;runtime&#34;
)

func main() {
	fmt.Println(runtime.Version())  //go1.18

}

```

## runtime.NumCPU()

&gt; 获取机器cpu数量

```go
package main

import (
	&#34;fmt&#34;
	&#34;runtime&#34;
)

func main() {
	fmt.Println(runtime.NumCPU())  // 16
}
```

## runtime.GOMAXPROCS(n int)

&gt; GOMAXPROCS设置可同时执行的最大CPU数，并返回先前的设置。 若 n &lt; 1，它就不会更改当前设置。

```go
package main

import (
	&#34;fmt&#34;
	&#34;runtime&#34;
)

func main() {
	fmt.Println(runtime.GOMAXPROCS(16))
}

```

## runtime.SetFinalizer()

&gt; 给变量绑定方法,当垃圾回收的时候进行监听 ` SetFinalizer(x, f interface{})`

```go
package main

import (
	&#34;runtime&#34;
	&#34;time&#34;
)

type Student struct {
	name string
}

func main() {
	var i *Student = new(Student)
	runtime.SetFinalizer(i, func(i interface{}) {
		println(&#34;垃圾回收了哦&#34;)
	})
	runtime.GC()
	time.Sleep(time.Second)
}

```



## runtime.GC()

&gt; 进行垃圾回收

```go
package main

import (
	&#34;runtime&#34;
	&#34;time&#34;
)

type Student struct {
	name string
}

func main() {
	var i *Student = new(Student)
	runtime.SetFinalizer(i, func(i interface{}) {
		println(&#34;垃圾回收了哦&#34;)
	})
	runtime.GC()
	time.Sleep(time.Second)
}

```

## runtime.ReadMemStats()

&gt; 查看内存申请和分配统计信息, 

```go
package main

import (
	&#34;fmt&#34;
	&#34;runtime&#34;
	&#34;time&#34;
)

type Student struct {
	name string
}

func main() {
	var list = make([]*Student, 0)
	for i := 0; i &lt; 100000; i&#43;&#43; {
		var s *Student = new(Student)
		list = append(list, s)
	}
	memStatus := runtime.MemStats{}
	runtime.ReadMemStats(&amp;memStatus)
	fmt.Printf(&#34;申请的内存:%d\n&#34;, memStatus.Mallocs)
	fmt.Printf(&#34;释放的内存次数:%d\n&#34;, memStatus.Frees)
	time.Sleep(time.Second)
}

```

- memStatus中可以查看到的信息

  ```go
  type MemStats struct {
      // 一般统计
      Alloc      uint64 // 已申请且仍在使用的字节数
      TotalAlloc uint64 // 已申请的总字节数（已释放的部分也算在内）
      Sys        uint64 // 从系统中获取的字节数（下面XxxSys之和）
      Lookups    uint64 // 指针查找的次数
      Mallocs    uint64 // 申请内存的次数
      Frees      uint64 // 释放内存的次数
      // 主分配堆统计
      HeapAlloc    uint64 // 已申请且仍在使用的字节数
      HeapSys      uint64 // 从系统中获取的字节数
      HeapIdle     uint64 // 闲置span中的字节数
      HeapInuse    uint64 // 非闲置span中的字节数
      HeapReleased uint64 // 释放到系统的字节数
      HeapObjects  uint64 // 已分配对象的总个数
      // L低层次、大小固定的结构体分配器统计，Inuse为正在使用的字节数，Sys为从系统获取的字节数
      StackInuse  uint64 // 引导程序的堆栈
      StackSys    uint64
      MSpanInuse  uint64 // mspan结构体
      MSpanSys    uint64
      MCacheInuse uint64 // mcache结构体
      MCacheSys   uint64
      BuckHashSys uint64 // profile桶散列表
      GCSys       uint64 // GC元数据
      OtherSys    uint64 // 其他系统申请
      // 垃圾收集器统计
      NextGC       uint64 // 会在HeapAlloc字段到达该值（字节数）时运行下次GC
      LastGC       uint64 // 上次运行的绝对时间（纳秒）
      PauseTotalNs uint64
      PauseNs      [256]uint64 // 近期GC暂停时间的循环缓冲，最近一次在[(NumGC&#43;255)%256]
      NumGC        uint32
      EnableGC     bool
      DebugGC      bool
      // 每次申请的字节数的统计，61是C代码中的尺寸分级数
      BySize [61]struct {
          Size    uint32
          Mallocs uint64
          Frees   uint64
      }
  }
  ```

## runtime.InUseBytes()

&gt; InUseBytes返回正在使用的字节数（AllocBytes – FreeBytes）

## runtime.InUseObjects()

&gt; InUseObjects返回正在使用的对象数（AllocObjects - FreeObjects）

## runtime.Stack()

&gt; Stack返回关联至此记录的调用栈踪迹，即r.Stack0的前缀。

## runtime.MemProfile()

&gt; MemProfile返回当前内存profile中的记录数n。若len(p)&gt;=n，MemProfile会将此分析报告复制到p中并返回(n, true)；如果len(p)&lt;n，MemProfile则不会更改p，而只返回(n, false)。
&gt;
&gt; 如果inuseZero为真，该profile就会包含无效分配记录（其中r.AllocBytes&gt;0，而r.AllocBytes==r.FreeBytes。这些内存都是被申请后又释放回运行时环境的）。
&gt;
&gt; 大多数调用者应当使用runtime/pprof包或testing包的-test.memprofile标记，而非直接调用MemProfile

## runtime.Breakpoint()

&gt; 执行一个断点

## runtime.Stack()

&gt; Stack将调用其的go程的调用栈踪迹格式化后写入到buf中并返回写入的字节数。若all为true，函数会在写入当前go程的踪迹信息后，将其它所有go程的调用栈踪迹都格式化写入到buf中。
&gt;
&gt; 在调用`Stack`方法后,首先格式化当前go协程的信息，然后把其他正在运行的go协程也格式化后写入buf中

```go
package main

import (
	&#34;fmt&#34;
	&#34;runtime&#34;
	&#34;time&#34;
)

func main() {
	go showRecord()
	time.Sleep(time.Second)
	buf := make([]byte, 10000)
	runtime.Stack(buf, true)
	fmt.Println(string(buf))
}

func showRecord() {
	tiker := time.Tick(time.Second)
	for t := range tiker {
		fmt.Println(t)
	}
}

```

![image-20221229160859663](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/20221229160859.png)

## runtime.Caller()

&gt; 获取当前函数或者上层函数的标识号、文件名、调用方法在当前文件中的行号

```go
package main

import (
	&#34;fmt&#34;
	&#34;runtime&#34;
)

func main() {
	pc, file, line, ok := runtime.Caller(0)
	fmt.Println(pc)   // 4308282211
	fmt.Println(file) // /Users/liusaisai/workspace/goProject/src/picturePro/main.go
	fmt.Println(line) // 9
	fmt.Println(ok)   // ok
}

```

## runtime.Callers()

&gt; `func Callers(skip int, pc []uintptr) int`
&gt;
&gt; 函数把当前go程序调用栈上的调用栈标识符填入切片pc中，返回写入到pc中的项数。
&gt;
&gt; 实参skip为开始在pc中记录之前所要跳过的栈帧数，0表示Callers自身的调用栈，1表示Callers所在的调用栈。返回写入p的项数

```go
package main

import (
	&#34;fmt&#34;
	&#34;runtime&#34;
)

func main() {
	pcs := make([]uintptr, 10)
	i := runtime.Callers(1, pcs)
	fmt.Println(pcs[:i])   // [4371884917 4371531008 4371698004]   三个pc 其中有一个是main方法自身的
}

```

## runtime.FuncForPC()

&gt; `func FuncForPC(pc uintptr) *Func`
&gt;
&gt; 获取一个标识调用栈标识符pc对应的调用栈

````go
package main

import (
	&#34;fmt&#34;
	&#34;runtime&#34;
)

func main() {
	pcs := make([]uintptr, 10)
	i := runtime.Callers(1, pcs)
	fmt.Println(pcs[:i])
	for _, pc := range pcs[:i] {
		p := runtime.FuncForPC(pc)
		file, line := p.FileLine(pc)
		println(
			fmt.Sprintf(&#34;调用栈:%v &#34;, p),
			fmt.Sprintf(&#34;调用函数名称:%v &#34;, p.Name()),
			fmt.Sprintf(&#34;文件名:%v 行号:%v &#34;, file, line),
			fmt.Sprintf(&#34;调用栈标识符:%v &#34;, p.Entry()),
		)
	}
}
```
[4305863597 4305503488 4305670692]
调用栈:&amp;{{}}  调用函数名称:main.main  文件名:/Users/liusaisai/workspace/goProject/src/picturePro/main.go 行号:10  调用栈标识符:4305863536 
调用栈:&amp;{{}}  调用函数名称:runtime.main  文件名:/Users/liusaisai/.g/go/src/runtime/proc.go 行号:259  调用栈标识符:4305502896 
调用栈:&amp;{{}}  调用函数名称:runtime.goexit  文件名:/Users/liusaisai/.g/go/src/runtime/asm_arm64.s 行号:1260  调用栈标识符:4305670688 
```
````

## runtime.NumCgoCall()

&gt; 获取当前进程调用c方法的次数

```go
package main

import (
	&#34;runtime&#34;
)

/*
#include &lt;stdio.h&gt;
*/
import &#34;C&#34;   // import c   调用了c包中的init方法

func main() {
	println(runtime.NumCgoCall())  // 1
}

```

## runtime.NumGoroutine()

&gt; 获取当前存在的go协程数

```go
package main

import &#34;runtime&#34;

func main() {
	go print()
	print()
	println(runtime.NumGoroutine())
}
func print() {

}

```



## runtime.Goexit()

&gt; 终止掉当前的go协程

```go
package main
import (
  &#34;runtime&#34;
    &#34;fmt&#34;
)

func main() {
 print()  // 1
 fmt.Println(&#34;继续执行&#34;)
}
func print(){
  fmt.Println(&#34;准备结束go协程&#34;)
  runtime.Goexit()
  defer fmt.Println(&#34;结束了&#34;)
}
```

- `Goexit`终止调用它的go协程,其他协程不受影响,`Goexit`会在终止该go协程前执行所有的defer函数，前提是defer必须在它前面定义,如果在main go协程调用本方法,会终止该go协程,但不会让main返回,因为main函数没有返回,程序会继续执行其他go协程,当其他go协程执行完毕后,程序就会崩溃

## runtime.Gosched()

&gt; 让出调度执行权限，让其他go协程优先执行, 等其他协程执行完后,在执行当前的协程

## runtime.GoroutineProfile()

&gt; 获取活跃的go协程的堆栈profile以及记录个数

## runtime.LockOSThread()

&gt; - 将调用的go程绑定到它当前所在的操作系统线程。除非调用的go程退出或调用UnlockOSThread，否则它将总是在该线程中执行，而其它go程则不能进入该线程
&gt; - 如果有需要协程,但是有一项重要的功能需要占一个线程，就需要它

```go
package main
import (
  &#34;fmt&#34;
  &#34;runtime&#34;
  &#34;time&#34;
)

func main() {
  go calcSum1()
  go calcSum2()
  time.Sleep(time.Second*100)
}

func calcSum1(){
  runtime.LockOSThread()
  start := time.Now()
  count := 0
  for i := 0; i &lt; 10000000000 ; i&#43;&#43;  {
    count &#43;= i
  }
  end := time.Now()
  fmt.Println(&#34;calcSum1耗时&#34;)
  fmt.Println(end.Sub(start))
  defer runtime.UnlockOSThread()
}

func calcSum2(){
  start := time.Now()
  count := 0
  for i := 0; i &lt; 10000000000 ; i&#43;&#43;  {
    count &#43;= i
  }
  end := time.Now()
  fmt.Println(&#34;calcSum2耗时&#34;)
  fmt.Println(end.Sub(start))
}
```

## runitme.UnlockOSThread()

&gt; 解除go协程与操作系统线程的绑定关系，将调用的go程解除和它绑定的操作系统线程。若调用的go程未调用LockOSThread，UnlockOSThread不做操作

## runtime.ThreadCreateProfile()

&gt; 获取线程创建profile中的记录个数， 返回线程创建profile中的记录个数。如果len(p)&gt;=n，本函数就会将profile中的记录复制到p中并返回(n, true)。若len(p)&lt;n，则不会更改p，而只返回(n, false)。
&gt;
&gt; 绝大多数使用者应当使用runtime/pprof包，而非直接调用ThreadCreateProfile。

## runtime.SetCPUProfileRate(hz int)

&gt; 官方注释：
&gt;
&gt; - SetCPUProfileRate设置CPU profile记录的速率为平均每秒hz次。如果hz&lt;=0，SetCPUProfileRate会关闭profile的记录。如果记录器在执行，该速率必须在关闭之后才能修改。
&gt; - 绝大多数使用者应使用runtime/pprof包或testing包的-test.cpuprofile选项而非直接使用SetCPUProfileRate

## runtime.SetBlockProfileRate(rate int)

&gt; 官方解释：
&gt;
&gt; - SetBlockProfileRate控制阻塞profile记录go程阻塞事件的采样频率。对于一个阻塞事件，平均每阻塞rate纳秒，阻塞profile记录器就采集一份样本。
&gt; - 要在profile中包括每一个阻塞事件，需传入rate=1；要完全关闭阻塞profile的记录，需传入rate&lt;=0。

## runtime.BlockProfile()

&gt; 返回当前阻塞profile中的记录个数
&gt;
&gt; BlockProfile返回当前阻塞profile中的记录个数。如果len(p)&gt;=n，本函数就会将此profile中的记录复制到p中并返回(n, true)。如果len(p)&lt;n，本函数则不会修改p，而只返回(n, false)。
&gt;
&gt; 绝大多数使用者应当使用runtime/pprof包或testing包的-test.blockprofile标记， 而非直接调用 BlockProfile





---

> 作者: [Fred](https://github.com/ipfred)  
> URL: https://ipfred.github.io/lang/go/go_advanced/20250515180259/  

