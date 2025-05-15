# 

## 使用场景

- `sync.Once` 是 Go 标准库提供的使函数只执行一次的实现，常应用于单例模式，例如初始化配置、保持数据库连接等。作用与 `init` 函数类似，但有区别。

  - init 函数是当所在的 package 首次被加载时执行，若迟迟未被使用，则既浪费了内存，又延长了程序加载时间。

  - sync.Once 可以在代码的任意位置初始化和调用，因此可以延迟到使用时再执行，并发场景下是线程安全的。

- 在多数情况下，`sync.Once` 被用于控制变量的初始化，这个变量的读写满足如下三个条件：

  - 当且仅当第一次访问某个变量时，进行初始化（写）；

  - 变量初始化过程中，所有读都被阻塞，直到初始化完成；

  - 变量仅初始化一次，初始化完成后驻留在内存里。

- `sync.Once` 仅提供了一个方法 `Do`，参数 f 是对象初始化函数。

  ```go
  func (o *Once) Do(f func())
  ```

## 使用示例

- 一个简单的 Demo
  - 考虑一个简单的场景，函数 ReadConfig 需要读取环境变量，并转换为对应的配置。环境变量在程序执行前已经确定，执行过程中不会发生改变。ReadConfig 可能会被多个协程并发调用，为了提升性能（减少执行时间和内存占用），使用 `sync.Once` 是一个比较好的方式。

- 标准库中 sync.Once 的使用

  - path.Cwd()

    ```go
    // Cwd returns the current working directory at the time of the first call.
    func Cwd() string {
    	cwdOnce.Do(func() {
    		var err error
    		cwd, err = os.Getwd()
    		if err != nil {
    			Fatalf(&#34;cannot determine current directory: %v&#34;, err)
    		}
    	})
    	return cwd
    }
    ```

## sync.Once 的原理

- 首先：保证变量仅被初始化一次，需要有个标志来判断变量是否已初始化过，若没有则需要初始化。

- 其次：线程安全，支持并发，无疑需要互斥锁来实现。

- 源码

  ```go
  package sync
  
  import (
  	&#34;sync/atomic&#34;
  )
  
  type Once struct {
  	done uint32
  	m    Mutex
  }
  
  func (o *Once) Do(f func()) {
  	if atomic.LoadUint32(&amp;o.done) == 0 {
  		// Outlined slow-path to allow inlining of the fast-path.
  		o.doSlow(f)
  	}
  }
  
  func (o *Once) doSlow(f func()) {
  	o.m.Lock()
  	defer o.m.Unlock()
  	if o.done == 0 {
  		defer atomic.StoreUint32(&amp;o.done, 1)
  		f()
  	}
  }
  
  ```

  

---

> 作者:   
> URL: http://localhost:1313/lang/go/go_advanced/8-5.-sync.once-%E5%8D%95%E4%BE%8B%E6%A8%A1%E5%BC%8F/  

