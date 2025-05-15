# 8-4. sync.Pool 复用对象

## sync.Pool

- sync.Pool 除了最常见的池化提升性能的思路，最重要的是减少 GC 。

- 常用于一些对象实例创建昂贵的场景。注意，Pool 是 Goroutine 并发安全的。

- 可以作为保存临时取还对象的一个“池子”。

## 特点

- Goroutine 并发安全的
- 存储的都是临时对象
- 自动移除, 清理完全是由runtime控制的, 随时都可能被无通知清除
- 当这个对象的引用只有sync.Pool持有时，这个对象内存会被释放
- 目的就是缓存并重用对象，减少GC的压力
- 自动扩容、缩容
- 不能对 Pool.Get 出来的对象做预判，有可能是新的（新分配的），有可能是旧的（之前人用过，然后 Put 进去的）
- 当用完一个从 Pool 取出的实例时候，一定要记得调用 Put，否则 Pool 无法复用这个实例，通常这个用 defer 完成；

## 源码解析

### 1.结构

```go
type Pool struct {
	noCopy noCopy

	local     unsafe.Pointer // local fixed-size per-P pool, actual type is [P]poolLocal
	localSize uintptr        // size of the local array

	victim     unsafe.Pointer // local from previous cycle
	victimSize uintptr        // size of victims array

	// New optionally specifies a function to generate
	// a value when Get would otherwise return nil.
	// It may not be changed concurrently with calls to Get.
	New func() interface{}
}

// Local per-P Pool appendix.
type poolLocalInternal struct {
	private interface{} // Can be used only by the respective P.
	shared  poolChain   // Local P can pushHead/popHead; any P can popTail.
}

type poolLocal struct {
	poolLocalInternal

	// Prevents false sharing on widespread platforms with
	// 128 mod (cache line size) = 0 .
	pad [128 - unsafe.Sizeof(poolLocalInternal{})%128]byte
}
```

- local这里面真正的是[P]poolLocal其中P就是GPM模型中的P，有多少个P数组就有多大，也就是每个P维护了一个本地的poolLocal。

- poolLocal里面维护了一个private一个shared，看名字其实就很明显了，private是给自己用的，而shared的是一个队列，可以给别人用的。注释写的也很清楚，自己可以从队列的头部存然后从头部取，而别的P可以从尾部取。

- victim这个从字面上面也可以知道，幸存者嘛，当进行gc的stw时候，会将local中的对象移到victim中去，也就是说幸存了一次gc，

### 2. Get 

```go
func (p *Pool) Get() interface{} {
    ......
    l, pid := p.pin()
    x := l.private
    l.private = nil
    if x == nil {
        // Try to pop the head of the local shard. We prefer
        // the head over the tail for temporal locality of
        // reuse.
        x, _ = l.shared.popHead()
        if x == nil {
            x = p.getSlow(pid)
        }
    }
    runtime_procUnpin()
    ......
    if x == nil &amp;&amp; p.New != nil {
        x = p.New()
    }
    return x
}

func (p *Pool) getSlow(pid int) interface{} {
    // See the comment in pin regarding ordering of the loads.
    size := atomic.LoadUintptr(&amp;p.localSize) // load-acquire
    locals := p.local                        // load-consume
    // Try to steal one element from other procs.
    for i := 0; i &lt; int(size); i&#43;&#43; {
        l := indexLocal(locals, (pid&#43;i&#43;1)%int(size))
        if x, _ := l.shared.popTail(); x != nil {
            return x
        }
    }

    // Try the victim cache. We do this after attempting to steal
    // from all primary caches because we want objects in the
    // victim cache to age out if at all possible.
    size = atomic.LoadUintptr(&amp;p.victimSize)
    if uintptr(pid) &gt;= size {
        return nil
    }
    locals = p.victim
    l := indexLocal(locals, pid)
    if x := l.private; x != nil {
        l.private = nil
        return x
    }
    for i := 0; i &lt; int(size); i&#43;&#43; {
        l := indexLocal(locals, (pid&#43;i)%int(size))
        if x, _ := l.shared.popTail(); x != nil {
            return x
        }
    }

    // Mark the victim cache as empty for future gets don&#39;t bother
    // with it.
    atomic.StoreUintptr(&amp;p.victimSize, 0)

    return nil
}
```

- 如果 private 不是空的，那就直接拿来用
- 如果 private 是空的，那就先去本地的shared队列里面从头 pop 一个
- 如果本地的 shared 也没有了，那 getSlow 去拿，其实就是去别的P的 shared 里面偷，偷不到回去 victim 幸存者里面找
- 如果最后都没有，那就只能调用 New 方法创建一个了

### 3. Put

```go
// Put adds x to the pool.
func (p *Pool) Put(x interface{}) {
    if x == nil {
        return
    }
    ......
    l, _ := p.pin()
    if l.private == nil {
        l.private = x
        x = nil
    }
    if x != nil {
        l.shared.pushHead(x)
    }
    runtime_procUnpin()
    ......
}
```

- 如果 private 没有，就放在 private

- 如果 private 有了，那么就放到 shared 队列的头部

  

## 应用场景

1. 当多个 goroutine 都需要创建同⼀个对象的时候，如果 goroutine 数过多，导致对象的创建数⽬剧增，进⽽导致 GC 压⼒增大。形成 “并发⼤－占⽤内存⼤－GC 缓慢－处理并发能⼒降低－并发更⼤”这样的恶性循环。
2. 对于很多需要重复分配、回收内存的地方，sync.Pool 是一个很好的选择。频繁地分配、回收内存会给 GC 带来一定的负担，严重的时候会引起 CPU 的毛刺，而 sync.Pool 可以将暂时不用的对象缓存起来，待下次需要的时候直接使用，不用再次经过内存分配，复用对象的内存，减轻 GC 的压力，提升系统的性能。
3. 标准库中 `encoding/json` 也用到了 sync.Pool 来提升性能。
4. 著名的 `gin` 框架，对 context 取用也到了 `sync.Pool`。

## demo

### 1. 未使用sync.Pool

```go
package main

import (
	&#34;fmt&#34;
	&#34;sync&#34;
	&#34;sync/atomic&#34;
)

var createNum int32

func createBuffer() interface{} {
	atomic.AddInt32(&amp;createNum, 1)
	buffer := make([]byte, 1024)
	return buffer
}

func main() {
	workerPool := 1024 * 1024
	var wg sync.WaitGroup
	wg.Add(workerPool)

	for i := 0; i &lt; workerPool; i&#43;&#43; {
		go func() {
			defer wg.Done()
			buffer := createBuffer()
			_ = buffer.([]byte)
		}()
	}
	wg.Wait()
	fmt.Printf(&#34; %d buffer objects were created.\n&#34;, createNum)
}

```

**输出结果:  1048576 buffer objects were created., 对象被创建了1048576次**

### 2. 使用sync.Pool

```go
package main

import (
	&#34;fmt&#34;
	&#34;sync&#34;
	&#34;sync/atomic&#34;
)

var createNum int32

func createBuffer() interface{} {
	atomic.AddInt32(&amp;createNum, 1)
	buffer := make([]byte, 1024)
	return buffer
}

func main() {
	bufferPool := &amp;sync.Pool{New: createBuffer}

	workerPool := 1024 * 1024
	var wg sync.WaitGroup
	wg.Add(workerPool)

	for i := 0; i &lt; workerPool; i&#43;&#43; {
		go func() {
			defer wg.Done()
			buffer := bufferPool.Get()
			_ = buffer.([]byte)
			defer bufferPool.Put(buffer)
		}()
	}
	wg.Wait()
	fmt.Printf(&#34; %d buffer objects were created.\n&#34;, createNum)
}

```

**最终输出结果:  8 buffer objects were create. 也就是对象创建了8次**

---

> 作者: [Fred](https://github.com/ipfred)  
> URL: https://ipfred.github.io/lang/go/go_advanced/20250515180211/  

