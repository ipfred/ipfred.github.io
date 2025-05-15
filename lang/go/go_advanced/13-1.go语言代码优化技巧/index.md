# 

## 1. sync.Pool

- sync.Pool 除了最常见的池化提升性能的思路，最重要的是减少 GC 。
- 常用于一些对象实例创建昂贵的场景。注意，Pool 是 Goroutine 并发安全的。

- 可以作为保存临时取还对象的一个“池子”。

- 特点
  1. Goroutine 并发安全的
  2. 存储的都是临时对象
  3. 自动移除, 清理完全是由runtime控制的, 随时都可能被无通知清除
  4. 当这个对象的引用只有sync.Pool持有时，这个对象内存会被释放
  5. 目的就是缓存并重用对象，减少GC的压力
  6. 自动扩容、缩容
  7. 不能对 Pool.Get 出来的对象做预判，有可能是新的（新分配的），有可能是旧的（之前人用过，然后 Put 进去的）
  8. 当用完一个从 Pool 取出的实例时候，一定要记得调用 Put，否则 Pool 无法复用这个实例，通常这个用 defer 完成；

- 应用场景
  1. 当多个 goroutine 都需要创建同⼀个对象的时候，如果 goroutine 数过多，导致对象的创建数⽬剧增，进⽽导致 GC 压⼒增大。形成 “并发⼤－占⽤内存⼤－GC 缓慢－处理并发能⼒降低－并发更⼤”这样的恶性循环。
  2. 对于很多需要重复分配、回收内存的地方，sync.Pool 是一个很好的选择。频繁地分配、回收内存会给 GC 带来一定的负担，严重的时候会引起 CPU 的毛刺，而 sync.Pool 可以将暂时不用的对象缓存起来，待下次需要的时候直接使用，不用再次经过内存分配，复用对象的内存，减轻 GC 的压力，提升系统的性能。
  3. 标准库中 `encoding/json` 也用到了 sync.Pool 来提升性能。
  4. 著名的 `gin` 框架，对 context 取用也到了 `sync.Pool`。
  5. fasthttp 大量使用sync.Pool

## 2. string相关

### 2.1 字符串拼接使用 strings.Builder

&gt; 官网说: A Builder is used to efficiently build a string using Write methods. It minimizes memory copying. 

- 字符串拼接方法
  1. 使用 `&#43;`
  2. 使用`fmt.Sprintf`
  3. 使用`strings.Builder`
  4. 使用`strings.Buffer`
  5. 使用`bytes.Buffer`

- 从基准测试的结果来看，使用 `&#43;` 和 `fmt.Sprintf` 的效率是最低的，和其余的方式相比，性能相差约 1000 倍，而且消耗了超过 1000 倍的内存。当然 `fmt.Sprintf` 通常是用来格式化字符串的，一般不会用来拼接字符串。
- `strings.Builder`、`bytes.Buffer` 和 `[]byte` 的性能差距不大，而且消耗的内存也十分接近，性能最好且消耗内存最小的是 `preByteConcat`，这种方式预分配了内存，在字符串拼接的过程中，不需要进行字符串的拷贝，也不需要分配新的内存，因此性能最好，且内存消耗最小。

- string.Builder 和 &#43;

  - 字符串在 Go 语言中是不可变类型，占用内存大小是固定的，当使用 `&#43;` 拼接 2 个字符串时，生成一个新的字符串，那么就需要开辟一段新的空间，新空间的大小是原来两个字符串的大小之和。拼接第三个字符串时，再开辟一段新空间，新空间大小是三个字符串大小之和，以此类推。假设一个字符串大小为 10 byte，拼接 1w 次，需要申请的内存大小为：

    ```
    10 &#43; 2 * 10 &#43; 3 * 10 &#43; ... &#43; 10000 * 10 byte = 500 MB 
    ```

  - 而 `strings.Builder`，`bytes.Buffer`，包括切片 `[]byte` 的内存是以倍数申请的。例如，初始大小为 0，当第一次写入大小为 10 byte 的字符串时，则会申请大小为 16 byte 的内存（恰好大于 10 byte 的 2 的指数），第二次写入 10 byte 时，内存不够，则申请 32 byte 的内存，第三次写入内存足够，则不申请新的，以此类推。在实际过程中，超过一定大小，比如 2048 byte 后，申请策略上会有些许调整。

    - 2048 以前按倍数申请，2048 之后，以 640 递增，最后一次递增 24576 到 122880。总共申请的内存大小约 `0.52 MB`，约为上一种方式的千分之一。

- strings.Builder 和 bytes.Buffer

  - `strings.Builder` 和 `bytes.Buffer` 底层都是 `[]byte` 数组，但 `strings.Builder` 性能比 `bytes.Buffer` 略快约 10% 。一个比较重要的区别在于，`bytes.Buffer` 转化为字符串时重新申请了一块空间，存放生成的字符串变量，而 `strings.Builder` 直接将底层的 `[]byte` 转换成了字符串类型返回了回来。

    - bytes.Buffer

    ```go
    // To build strings more efficiently, see the strings.Builder type.
    func (b *Buffer) String() string {
    	if b == nil {
    		// Special case, useful in debugging.
    		return &#34;&lt;nil&gt;&#34;
    	}
    	return string(b.buf[b.off:])
    }
    ```

    - strings.Builder

    ```go
    // String returns the accumulated string.
    func (b *Builder) String() string {
    	return *(*string)(unsafe.Pointer(&amp;b.buf))
    }
    ```

### 2.2 字符串截取 strings.Repeat

#### 

&gt; 一个子字符串表达式的结果（子）字符串和基础字符共享一个承载底层字节序列的内存块。不仅节省内存，而且还减少了CPU消耗。 但是有时候它会造成暂时性的内存泄露。

- demo

  ```go
  var s0 string // 一个包级变量
  
  // 一个演示目的函数。
  func f(s1 string) {
  	s0 = s1[:50]
  	// 目前，s0和s1共享着承载它们的字节序列的同一个内存块。
  	// 虽然s1到这里已经不再被使用了，但是s0仍然在使用中，
  	// 所以它们共享的内存块将不会被回收。虽然此内存块中
  	// 只有50字节被真正使用，而其它字节却无法再被使用。
  }
  
  func demo() {
  	s := createStringWithLengthOnHeap(1 &lt;&lt; 20) // 1M bytes
  	f(s)
  }
  ```

- **解决办法**

  1. 将子字符串表达式的结果转换为一个字节切片，然后再转换回来。此种防止临时性内存泄露的方法不是很高效，因为在此过程中底层的字节序列被复制了两次，其中一次是不必要的。

     ```go
     func f(s1 string) {
     	s0 = string([]byte(s1[:50]))
     }
     ```

     

  2. [**推荐**]使用`strings.Builder`类型来防止一次不必要的复制。

     ```go
     import &#34;strings&#34;
     
     func f(s1 string) {
     	var b strings.Builder
     	b.Grow(50)
     	b.WriteString(s1[:50])
     	s0 = b.String()
     }
     ```

  3. 使用`strings.Repeat`, 此方法底层也是`strings.Builder`的封装

## 3. 使用协程池

- 协程池作用
  1. 可以限制goroutine数量，避免无限制的增长。
  2. 减少栈扩容的次数。
  3. 频繁创建goroutine的场景下，资源复用，节省内存。（需要一定规模。一般场景下，效果不太明显。）

- 推荐第三方库 [ants](https://github.com/panjf2000/ants)

- go对goroutine有一定的复用能力。所以要根据场景选择是否使用协程池，不恰当的场景不仅得不到收益，反而增加系统复杂性。

## 4. for 和 range 选择

- range 在迭代过程中返回的是迭代值的拷贝
- 如果每次迭代的元素的内存占用很低，那么 for 和 range 的性能几乎是一样，例如 `[]int`。
- 如果迭代的元素内存占用较高，例如一个包含很多属性的 struct 结构体，那么 for 的性能将显著地高于 range，有时候甚至会有上千倍的性能差异。对于这种场景，建议使用 for，如果使用 range，建议只迭代下标，通过下标访问迭代值，这种使用方式和 for 就没有区别了。
- 如果想使用 range 同时迭代下标和值，则需要将切片/数组的元素改为指针，才能不影响性能。
- 尽量使用for,而不是range

## 5. 减小锁的资源消耗

- 对临界区加锁比较常见, 性能损耗也是非常严重的

- 标准库中sync.map针对读操作的优化消除了rwlock，是一个标准的案例. 用原子操作代替互斥锁也是一种经典的lock-free技巧。

## 6. 不要使用反射, 除非忍不住

- 反射可以帮助抽象和简化代码，提高开发效率。但是go语言反射效率不高.
- 反射创建对象效率相差不大, 但是动态修改字段的值效率极低!

## 7. 结构体声明考虑内存对齐

- CPU 访问内存时并不是逐个字节访问，而是以字长（word size）为单位访问，例如 **32位的CPU 字长是4字节**，**64位的是8字节**。如果变量的地址没有对齐，可能需要多次访问才能完整读取到变量内容，而对齐后可能就只需要一次内存访问，因此内存对齐可以减少CPU访问内存的次数，加大CPU访问内存的吞吐量。
- 在实际开发中，我们可以通过调整变量位置，优化内存占用（一般按照变量内存大小顺序排列，整体占用内存更小）

## 8. slice 相关

### 8.1 创建slice和map声明cap

- 尽可能的声明容量
- 使用append向Slice追加元素时，如果Slice空间不足，将会触发Slice扩容，扩容实际上是重新分配一块更大的内存，将原Slice数据拷贝进新Slice，然后返回新Slice，扩容后再将数据追加进去。
- 扩容容量的选择遵循以下规则：
  - **如果原Slice容量小于1024，则新Slice容量将扩大为原来的2倍；**
  - **如果原Slice容量大于等于1024，则新Slice容量将扩大为原来的1.25倍；**
- 扩容消耗资源

### 8.2 slice 的截取[::]和拷贝

&gt; slice 使用方式不对容易造成内存的伪泄露、数据篡改等问题

&gt; 切片截取子切片时，会造成临时内存泄露， 主要原因有两个
&gt;
&gt; 1. 切片截取时，新旧切片会共用一个底层数组
&gt; 2. 切片的底层结构体指向数组的指针只是一个头指针

- demo

  ```go
  package main
  
  import &#34;fmt&#34;
  
  func main() {
  	a := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
  	c := a[1:2]
  	fmt.Println(len(c), cap(c))     // 1,9   c的数组头指针执行索引1，所以容量为9
  }
  ```

- 解决办法

  1. 使用copy，不过要注意copy时的长度和容量问题
  2. 使用`slice [1:2:3]` 两个冒号语法截取：`[startIndex:endIndex:max]`, **其中 max 的值一定要大于 endIndex**
     - 新切片的容量就是`max - startIndex`,
     - 实际引用的数组时从数组`startIndex`索引开始到`max`索引为止，但不包括max索引处的元素,
     - 新切片的长度就是`endIndex - startIndex `

## 9. 空占位符使用struct{}

- 空结构体在内存中不占用空间

- 用法

  1. 与map结合实现set

     - Go 语言标准库没有提供 Set 的实现，通常使用 map 来代替。事实上，对于集合来说，只需要 map 的键，而不需要值。即使是将值设置为 bool 类型，也会多占据 1 个字节，那假设 map 中有一百万条数据，就会浪费 1MB 的空间
     - 将 map 作为集合(Set)使用时，可以将值类型定义为空结构体，仅作为占位符使用即可。

  2.  制造伪迭代器

     ```go
     for range make([]struct{}, 100) {
       fmt.Println(&#34;迭代器&#34;)
     }
     ```

  3. 不发送数据的channel

     ```go
     func worker(ch chan struct{}) {
     	&lt;-ch
     	fmt.Println(&#34;do something&#34;)
     	close(ch)
     }
     
     func main() {
     	ch := make(chan struct{})
     	go worker(ch)
     	ch &lt;- struct{}{}
     }
     ```


## 10. 考虑内存逃逸

- 控制变量不发生逃逸，将其控制在栈上，减少堆变量的分配，降低GC成本，提高程序性能。

- 变量逃逸一般发生在如下几种情况：

  - 变量较大（栈空间不足）

  - 变量大小不确定（如slice长度或容量不定）

  - 返回地址

  - 返回引用（引用变量的底层是指针）

  - 返回值类型不确定（不能确定大小）

  - 闭包

## 11. 返回值VS返回指针

- 值传递会拷贝整个对象，而指针传递只会拷贝地址，指向的对象是同一个。传指针可以减少值的拷贝，但是会导致内存分配逃逸到堆中，增加垃圾回收（GC）的负担。在对象频繁创建和删除的场景下，返回指针导致的GC开销可能会严重影响性能。
- 一般情况下，对于需要修改原对象，或占用内存比较大的对象，返回指针。对于只读或占用内存较小的对象，返回值能够获得更好的性能。




&gt; 持续完善...

---

> 作者:   
> URL: http://localhost:1313/lang/go/go_advanced/13-1.go%E8%AF%AD%E8%A8%80%E4%BB%A3%E7%A0%81%E4%BC%98%E5%8C%96%E6%8A%80%E5%B7%A7/  

