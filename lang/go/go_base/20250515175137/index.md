# 10. 内置函数&amp;库

# 内置函数

## close 

close 用于 channel 通讯。使用它来关闭 channel

## delete

delete 用于在 map 中删除实例。 

## len 和 cap 

len 和 cap 可用于不同的类型，len 用于返回字符串、slice 和数组的长度。

## new

new 用于各种类型的内存分配, new 返回一个指针类型,但是不会初始化变量, 会将值全部置为零值

## make 

make 用于内建类型（map、slice 和 channel）的内存分配。返回这三个的引用变量

## copy 

copy 用于复制 slice。

## append 

append 用于追加 slice

## panic

panic 和 recover 用于异常处理机制。

## print 和 println

print 和 println 是底层打印函数，可以在不引入 fmt 包的情况下使用。它们主要用于调试。

## complex、real 和 imag

complex、real 和 imag 全部用于处理 复数





# 库/包

# OS

- os.Args  获取命令行参数,返回值是一个切片,第一个参数是可执行文件名称

  ```
  go run main.go   // os.Args[1] = &#34;n&#34;
  ```

  



## container/list

- list是一个双向链表。该结构具有链表的所有功能。

- type Element
  - func (e *Element) Next() *Element //返回该元素的下一个元素，如果没有下一个元素则返回nil
  - func (e *Element) Prev() *Element//返回该元素的前一个元素，如果没有前一个元素则返回nil。
  - element.Value  获取元素的值

```go
type Element struct {
        Value interface{}   //在元素中存储的值
}
```



- type List
  - func New() *List //返回一个初始化的list
  - func (l *List) Back() *Element //获取list l的最后一个元素
  - func (l *List) Front() *Element //获取list l的第一个元素
  - func (l *List) Init() *List //list l初始化或者清除list l
  - func (l *List) InsertAfter(v interface{}, mark *Element) *Element //在list l中元素mark之后插入一个值为v的元素，并返回该元素，如果mark不是list中元素，则list不改变。
  - func (l *List) InsertBefore(v interface{}, mark *Element) *Element//在list l中元素mark之前插入一个值为v的元素，并返回该元素，如果mark不是list中元素，则list不改变。
  - func (l *List) Len() int //获取list l的长度
  - func (l *List) MoveAfter(e, mark *Element) //将元素e移动到元素mark之后，如果元素e或者mark不属于list l，或者e==mark，则list l不改变。
  - func (l *List) MoveBefore(e, mark *Element)//将元素e移动到元素mark之前，如果元素e或者mark不属于list l，或者e==mark，则list l不改变。
  - func (l *List) MoveToBack(e *Element)//将元素e移动到list l的末尾，如果e不属于list l，则list不改变。
  - func (l *List) MoveToFront(e *Element)//将元素e移动到list l的首部，如果e不属于list l，则list不改变。
  - func (l *List) PushBack(v interface{}) *Element//在list l的末尾插入值为v的元素，并返回该元素。
  - func (l *List) PushBackList(other *List)//在list l的尾部插入另外一个list，其中l和other可以相等。
  - func (l *List) PushFront(v interface{}) *Element//在list l的首部插入值为v的元素，并返回该元素。
  - func (l *List) PushFrontList(other *List)//在list l的首部插入另外一个list，其中l和other可以相等。
  - func (l *List) Remove(e *Element) interface{}//如果元素e属于list l，将其从list中删除，并返回元素e的值。

举例说明其用法。

```go
package main

import (
	&#34;container/list&#34;
	&#34;fmt&#34;
)

func main() {
	l := list.New() //创建一个新的list
	for i := 0; i &lt; 5; i&#43;&#43; {
		l.PushBack(i)
	}
	for e := l.Front(); e != nil; e = e.Next() {
		fmt.Print(e.Value) //输出list的值,01234
	}
	fmt.Println(&#34;&#34;)
	fmt.Println(l.Front().Value) //输出首部元素的值,0
	fmt.Println(l.Back().Value)  //输出尾部元素的值,4
	l.InsertAfter(6, l.Front())  //首部元素之后插入一个值为10的元素
	for e := l.Front(); e != nil; e = e.Next() {
		fmt.Print(e.Value) //输出list的值,061234
	}
	fmt.Println(&#34;&#34;)
	l.MoveBefore(l.Front().Next(), l.Front()) //首部两个元素位置互换
	for e := l.Front(); e != nil; e = e.Next() {
		fmt.Print(e.Value) //输出list的值,601234
	}
	fmt.Println(&#34;&#34;)
	l.MoveToFront(l.Back()) //将尾部元素移动到首部
	for e := l.Front(); e != nil; e = e.Next() {
		fmt.Print(e.Value) //输出list的值,460123
	}
	fmt.Println(&#34;&#34;)
	l2 := list.New()
	l2.PushBackList(l) //将l中元素放在l2的末尾
	for e := l2.Front(); e != nil; e = e.Next() {
		fmt.Print(e.Value) //输出l2的值,460123
	}
	fmt.Println(&#34;&#34;)
	&lt;span style=&#34;color:#FF0000;&#34;&gt;l.Init()           //清空l&lt;/span&gt;
	fmt.Print(l.Len()) //0
	for e := l.Front(); e != nil; e = e.Next() {
		fmt.Print(e.Value) //输出list的值,无内容
	}

}
```

## sync.WaitGroup

- WaitGroup提供3个方法实现优雅退出

1. Add() ：每收到http/mq请求，会在计数器&#43;1
2. Done()：每执行完http/mq请求，会在计数器-1
3. Wait()：计数器=0，即没有正在处理的 请求

```go
import &#34;sync&#34;
import &#34;os/signal&#34;
import &#34;fmt&#34;

func main() {
	var wg sync.WaitGroup
	sig := make(chan os.Signal, 1)
    signal.Notify(sigs, syscall.SIGINT, syscall.SIGTERM)
	httpHandle()
	
	go func() {
		sig := &lt;-sig
		wg.Wait()
		done &lt;- true
	}()
	
	&lt;-done
	fmt.println(&#34;quit success&#34;)
}

func httpHandle() {
	wg.Add(1)
	defer wg.Done()
	xxx     
}

```



## sync.Map()

- 支持并发的map

  ```go
  
  type Map
      //删除指定key
      func (m *Map) Delete(key interface{})
      
      //查询指定key
      func (m *Map) Load(key interface{}) (value interface{}, ok bool)
   
      //查询，查不到则追加
      func (m *Map) LoadOrStore(key, value interface{}) (actual interface{}, loaded bool)
   
      //遍历map
      func (m *Map) Range(f func(key, value interface{}) bool)
   
      //添加
      func (m *Map) Store(key, value interface{})
  ```

  

---

> 作者: [Fred](https://github.com/ipfred)  
> URL: https://ipfred.github.io/lang/go/go_base/20250515175137/  

