# 

## O、内存相关

- 下列程序为什么会卡死(测试不会被卡死)

  ```go
  package main
  import (
   &#34;fmt&#34;
   &#34;runtime&#34;
  )
  func main() {
   var i byte
   go func() {
     for i = 0; i &lt;= 255; i&#43;&#43; {}
    }()
  fmt.Println(&#34;Dropping mic&#34;)
  // Yield execution to force executing other goroutines runtime.Gosched()
  runtime.GC()
  fmt.Println(&#34;Done&#34;)
  }  
  ```

  &gt; 解析： Golang 中，byte 其实被 alias 到 uint8 上了。所以上⾯的 for 循环会始终成⽴，因为 i&#43;&#43; 到 i=255 的时候会溢出，i &lt;= 255 ⼀定成⽴。 也即是， for 循环永远⽆法退出，所以上⾯的代码其实可以等价于这样：
  &gt;
  &gt; go func() { for {} }
  &gt;
  &gt; 正在被执⾏的 goroutine 发⽣以下情况时让出当前 goroutine 的执⾏权，并调度后⾯的 goroutine 执⾏：
  &gt;
  &gt; - IO 操作 
  &gt; -  Channel 阻塞 
  &gt; - system call 
  &gt; - 运⾏较⻓时间 
  &gt;
  &gt; 如果⼀个 goroutine 执⾏时间太⻓，scheduler 会在其 G 对象上打上⼀个标志（ preempt），当这个 goroutine 内部发⽣函数调⽤的时候，会先主动检查这个标志，如 果为 true 则会让出执⾏权。
  &gt;
  &gt;  main 函数⾥启动的 goroutine 其实是⼀个没有 IO 阻塞、没有 Channel 阻塞、没有 system call、没有函数调⽤的死循环。 也就是，它⽆法主动让出⾃⼰的执⾏权，即使已经执⾏很⻓时间，scheduler 已经标志 了 preempt。 ⽽ golang 的 GC 动作是需要所有正在运⾏ goroutine 都停⽌后进⾏的。因此，程序 会卡在 runtime.GC() 等待所有协程退出。

## 一、数据定义

### (1).函数返回值问题

&gt; 下面代码是否可以编译通过？

```go
package main

/*
    下面代码是否编译通过?
*/
func myFunc(x,y int)(sum int,error){
    return x&#43;y,nil
}

func main() {
    num, err := myFunc(1, 2)
    fmt.Println(&#34;num = &#34;, num)
}
```

答案:

编译报错理由:

```sh
# command-line-arguments
./test1.go:6:21: syntax error: mixed named and unnamed function parameters
```

&gt; 考点：函数返回值命名

&gt; 结果：编译出错。

&gt; 在函数有多个返回值时，只要有一个返回值有指定命名，其他的也必须有命名。 如果返回值有有多个返回值必须加上括号； 如果只有一个返回值并且有命名也需要加上括号； 此处函数第一个返回值有sum名称，第二个未命名，所以错误。

### (2).结构体比较问题

&gt; 下面代码是否可以编译通过？为什么？

```go
package main

import &#34;fmt&#34;

func main() {

	sn1 := struct {
		age  int
		name string
	}{age: 11, name: &#34;qq&#34;}

	sn2 := struct {
		age  int
		name string
	}{age: 11, name: &#34;qq&#34;}

	if sn1 == sn2 {
		fmt.Println(&#34;sn1 == sn2&#34;)
	}

	sm1 := struct {
		age int
		m   map[string]string
	}{age: 11, m: map[string]string{&#34;a&#34;: &#34;1&#34;}}

	sm2 := struct {
		age int
		m   map[string]string
	}{age: 11, m: map[string]string{&#34;a&#34;: &#34;1&#34;}}

	if sm1 == sm2 {
		fmt.Println(&#34;sm1 == sm2&#34;)
	}
}
```

结果

编译不通过

```
./test2.go:31:9: invalid operation: sm1 == sm2 (struct containing map[string]string cannot be compared)
```

考点:**结构体比较**

&gt; **结构体比较规则注意1**：只有相同类型的结构体才可以比较，结构体是否相同不但与属性类型个数有关，还与属性顺序相关.

比如：

```
sn1 := struct {
	age  int
	name string
}{age: 11, name: &#34;qq&#34;}

sn3:= struct {
    name string
    age  int
}{age:11, name:&#34;qq&#34;}
```

`sn3`与`sn1`就不是相同的结构体了，不能比较。

&gt; **结构体比较规则注意2**：结构体是相同的，但是结构体属性中有不可以比较的类型，如`map`,`slice`，则结构体不能用`==`比较。

可以使用reflect.DeepEqual进行比较

```
if reflect.DeepEqual(sm1, sm2) {
		fmt.Println(&#34;sm1 == sm2&#34;)
} else {
		fmt.Println(&#34;sm1 != sm2&#34;)
}
```

### (3).string与nil类型

&gt; 下面代码是否能够编译通过？为什么？

```go
package main

import (
    &#34;fmt&#34;
)

func GetValue(m map[int]string, id int) (string, bool) {
	if _, exist := m[id]; exist {
		return &#34;存在数据&#34;, true
	}
	return nil, false
}

func main()  {
	intmap:=map[int]string{
		1:&#34;a&#34;,
		2:&#34;bb&#34;,
		3:&#34;ccc&#34;,
	}

	v,err:=GetValue(intmap,3)
	fmt.Println(v,err)
}
```

考点：**函数返回值类型**

答案：编译不会通过。

分析：

nil 可以用作 interface、function、pointer、map、slice 和 channel 的“空值”。但是如果不特别指定的话，Go 语言不能识别类型，所以会报错。通常编译的时候不会报错，但是运行是时候会报:`cannot use nil as type string in return argument`.

所以将`GetValue`函数改成如下形式就可以了

```
func GetValue(m map[int]string, id int) (string, bool) {
	if _, exist := m[id]; exist {
		return &#34;存在数据&#34;, true
	}
    return &#34;不存在数据&#34;, false
}
```

### (4) 常量

&gt; 下面函数有什么问题？

```go
package main

const cl = 100

var bl = 123

func main()  {
    println(&amp;bl,bl)
    println(&amp;cl,cl)
}
```

解析

考点:**常量**
常量不同于变量的在运行期分配内存，常量通常会被编译器在预处理阶段直接展开，作为指令数据使用，

```
cannot take the address of cl
```

内存四区概念：

#### A.数据类型本质：

 固定内存大小的别名

#### B. 数据类型的作用：

 编译器预算对象(变量)分配的内存空间大小。

![img](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/c9fcb2200f908c3a2ceb887b66e2c0d7_1358x910.png)

#### C. 内存四区

![img](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/c9fcb2200f908c3a2ceb887b66e2c0d7_1358x910.png)

流程说明

1、操作系统把物理硬盘代码load到内存

2、操作系统把c代码分成四个区

3、操作系统找到main函数入口执行

##### 栈区(Stack)：

- 空间较小，要求数据读写性能高，数据存放时间较短暂。由编译器自动分配和释放，存放函数的参数值、函数的调用流程方法地址、局部变量等(局部变量如果产生逃逸现象，可能会挂在在堆区)

##### 堆区(heap):

-  空间充裕，数据存放时间较久。一般由开发者分配及释放(但是Golang中会根据变量的逃逸现象来选择是否分配到栈上或堆上)，启动Golang的GC由GC清除机制自动回收。

##### 全局区-静态全局变量区:

-  全局变量的开辟是在程序在`main`之前就已经放在内存中。而且对外完全可见。即作用域在全部代码中，任何同包代码均可随时使用，在变量会搞混淆，而且在局部函数中如果同名称变量使用`:=`赋值会出现编译错误。

- 全局变量最终在进程退出时，由操作系统回收。

&gt;我们尽量减少使用全局变量的设计

##### 全局区-常量区：

- 常量区也归属于全局区，常量为存放数值字面值单位，即不可修改。或者说的有的常量是直接挂钩字面值的。

- 比如:

```
const cl = 10
```

- cl是字面量10的对等符号。

**所以在golang中，常量是无法取出地址的，因为字面量符号并没有地址而言。**

### (5) defer练习

```go
package main

import &#34;fmt&#34;

func DeferFunc1(i int) (t int) {
	t = i
	defer func() {
		t &#43;= 3
	}()
	return t
}

func DeferFunc2(i int) int {
	t := i
	defer func() {
		t &#43;= 3
	}()
	return t
}

func DeferFunc3(i int) (t int) {
	defer func() {
		t &#43;= i
	}()
	return 2
}

func DeferFunc4() (t int) {
	defer func(i int) {
		fmt.Println(i)
		fmt.Println(t)
	}(t)
	t = 1
	return 2
}

func main() {
	fmt.Println(DeferFunc1(1)) // 4
	fmt.Println(DeferFunc2(1)) // 1
	fmt.Println(DeferFunc3(1)) // 3
	DeferFunc4()  // 0 2
}

```

## 二、数组和切片

### (1) 切片的初始化与追加

&gt;1.1

```go
package main

import &#34;fmt&#34;

func main() {
    s1 := []int{1, 2, 3}
    fmt.Println(len(s1), cap(s1), s1) //输出 3 3 [1 2 3]
    s2 := s1[1:] //索引从第二个元素截取开始
    fmt.Println(len(s2), cap(s2), s2) //输出 2 2 [2 3]
    for i := range s2 {
        s2[i] &#43;= 20
    }
    //仍然引用同一数组
    fmt.Println(s1) //s1 在s2修改了后面2个元素，所以s1也是更新了。输出 [1 22 23]
    fmt.Println(s2) //输出 [22 23]
    s2 = append(s2, 4) // 注意s2的容量是2，追加新元素后将导致分配一个新的数组 [22 23 4]
    for i := range s2 {
        s2[i] &#43;= 10
    }
    //s1 仍然是更新后的历史老数据
    fmt.Println(s1) //输出 [1 22 23]
    fmt.Println(s2) //输出 [32 33 14]
}	
```

&gt; 1.2 写出程序运行的结果

```go
package main

import (
    &#34;fmt&#34;
)

func main(){
    s := make([]int, 10)

    s = append(s, 1, 2, 3)

    fmt.Println(s)
}
```

**考点**

切片追加, make初始化均为0

**结果**

```
[0 0 0 0 0 0 0 0 0 0 1 2 3]
```



### (2) slice拼接问题

&gt; 下面是否可以编译通过？

&gt; test6.go

```go
package main

import &#34;fmt&#34;

func main() {
	s1 := []int{1, 2, 3}
	s2 := []int{4, 5}
	s1 = append(s1, s2)
	fmt.Println(s1)
}
```

**结果**

编译失败

两个slice在append的时候，记住需要进行将第二个slice进行`...`打散再拼接。

```
s1 = append(s1, s2...)
```

### (3) slice中new的使用

&gt; 下面代码是否可以编译通过？

```go
package main

import &#34;fmt&#34;

func main() {
  
	list := new([]int)
  
	list = append(list, 1)
  
	fmt.Println(list)
}
```

**结果**：

编译失败，`./test9.go:9:15: first argument to append must be slice; have *[]int`

**分析**：

&gt; 切片指针的解引用。

&gt; 可以使用list:=make([]int,0) list类型为切片

&gt; 或使用*list = append(*list, 1) list类型为指针

**new和make的区别：**

 二者都是内存的分配（堆上），但是make只用于slice、map以及channel的初始化（非零值）；而new用于类型的内存分配，并且内存置为零。所以在我们编写程序的时候，就可以根据自己的需要很好的选择了。

 make返回的还是这三个引用类型本身；而new返回的是指向类型的指针。

## 三、Map

### (1) Map的Value赋值

&gt; 下面代码编译会出现什么结果？

```go
package main

import &#34;fmt&#34;

type Student struct {
	Name string
}

var list map[string]Student

func main() {

	list = make(map[string]Student)

	student := Student{&#34;Aceld&#34;}

	list[&#34;student&#34;] = student
	list[&#34;student&#34;].Name = &#34;LDB&#34;

	fmt.Println(list[&#34;student&#34;])
}
```

**结果**

编译失败，`./test7.go:18:23: cannot assign to struct field list[&#34;student&#34;].Name in map`

**分析**

`map[string]Student` 的value是一个Student结构值，所以当`list[&#34;student&#34;] = student`,是一个值拷贝过程。而`list[&#34;student&#34;]`则是一个值引用。那么值引用的特点是`只读`。所以对`list[&#34;student&#34;].Name = &#34;LDB&#34;`的修改是不允许的。

**方法一：**

```go
package main

import &#34;fmt&#34;

type Student struct {
	Name string
}

var list map[string]Student

func main() {

	list = make(map[string]Student)

	student := Student{&#34;Aceld&#34;}

	list[&#34;student&#34;] = student
	//list[&#34;student&#34;].Name = &#34;LDB&#34;

    /*
        方法1:
    */
    tmpStudent := list[&#34;student&#34;]
    tmpStudent.Name = &#34;LDB&#34;
    list[&#34;student&#34;] = tmpStudent

	fmt.Println(list[&#34;student&#34;])
}
```

其中

```go
    /*
        方法1:
    */
    tmpStudent := list[&#34;student&#34;]
    tmpStudent.Name = &#34;LDB&#34;
    list[&#34;student&#34;] = tmpStudent
```

是先做一次值拷贝，做出一个`tmpStudent副本`,然后修改该副本，然后再次发生一次值拷贝复制回去，`list[&#34;student&#34;] = tmpStudent`,但是这种会在整体过程中发生2次结构体值拷贝，性能很差。

**方法二**：

```go
package main

import &#34;fmt&#34;

type Student struct {
	Name string
}

var list map[string]*Student

func main() {

	list = make(map[string]*Student)

	student := Student{&#34;Aceld&#34;}

	list[&#34;student&#34;] = &amp;student
	list[&#34;student&#34;].Name = &#34;LDB&#34;

	fmt.Println(list[&#34;student&#34;])
}
```

我们将map的类型的value由Student值，改成Student指针。

```
var list map[string]*Student
```

这样，我们实际上每次修改的都是指针所指向的Student空间，指针本身是常指针，不能修改，`只读`属性，但是指向的Student是可以随便修改的，而且这里并不需要值拷贝。只是一个指针的赋值。

### (2) map的遍历赋值

------

&gt; 以下代码有什么问题，说明原因

```go
package main

import (
    &#34;fmt&#34;
)

type student struct {
    Name string
    Age  int
}

func main() {
    //定义map
    m := make(map[string]*student)

    //定义student数组
    stus := []student{
        {Name: &#34;zhou&#34;, Age: 24},
        {Name: &#34;li&#34;, Age: 23},
        {Name: &#34;wang&#34;, Age: 22},
    }

    //将数组依次添加到map中
    for _, stu := range stus {
        m[stu.Name] = &amp;stu
    }

    //打印map
    for k,v := range m {
        fmt.Println(k ,&#34;=&gt;&#34;, v.Name)
    }
}
```

**结果**

遍历结果出现错误，输出结果为

```
zhou =&gt; wang
li =&gt; wang
wang =&gt; wang
```

map中的3个key均指向数组中最后一个结构体。

**分析**

foreach中，stu是结构体的一个拷贝副本，所以`m[stu.Name]=&amp;stu`实际上一致指向同一个指针， 最终该指针的值为遍历的最后一个`struct的值拷贝`。

![img](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/440218d6f132a32686ccd4a7d84b3f9b_1920x1080-20211205173756661.jpeg)

**正确写法**

```go
package main

import (
    &#34;fmt&#34;
)

type student struct {
    Name string
    Age  int
}

func main() {
    //定义map
    m := make(map[string]*student)

    //定义student数组
    stus := []student{
        {Name: &#34;zhou&#34;, Age: 24},
        {Name: &#34;li&#34;, Age: 23},
        {Name: &#34;wang&#34;, Age: 22},
    }

    // 遍历结构体数组，依次赋值给map
    for i := 0; i &lt; len(stus); i&#43;&#43;  {
        m[stus[i].Name] = &amp;stus[i]
    }

    //打印map
    for k,v := range m {
        fmt.Println(k ,&#34;=&gt;&#34;, v.Name)
    }
}
```

![img](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/55827855b68b88a35ae41179b29f55fa_1920x1080-20211205173816210.jpeg)

**运行结果**

```
zhou =&gt; zhou
li =&gt; li
wang =&gt; wang
```

## 四、interface

### (1) interface的赋值问题

&gt; 以下代码能编译过去吗？为什么？

```go
package main

import (
	&#34;fmt&#34;
)

type People interface {
	Speak(string) string
}

type Stduent struct{}

func (stu *Stduent) Speak(think string) (talk string) {
	if think == &#34;love&#34; {
		talk = &#34;You are a good boy&#34;
	} else {
		talk = &#34;hi&#34;
	}
	return
}

func main() {
	var peo People = Stduent{}
	think := &#34;love&#34;
	fmt.Println(peo.Speak(think))
}
```

- 继承与多态的特点在golang中对多态的特点体现从语法上并不是很明显。

- 我们知道发生多态的几个要素：

​	1、有interface接口，并且有接口定义的方法。

​	2、有子类去重写interface的接口。

​	3、有父类指针指向子类的具体对象

那么，满足上述3个条件，就可以产生多态效果，就是，父类指针可以调用子类的具体方法。

所以上述代码报错的地方在`var peo People = Stduent{}`这条语句， `Student{}`已经重写了父类`People{}`中的`Speak(string) string`方法，那么只需要用父类指针指向子类对象即可。

- 所以应该改成`var peo People = &amp;Student{}` 即可编译通过。（People为interface类型，就是指针类型）

### (2) interface的内部构造(非空接口iface情况)

&gt; 以下代码打印出来什么内容，说出为什么。

```go
package main

import (
	&#34;fmt&#34;
)

type People interface {
	Show()
}

type Student struct{}

func (stu *Student) Show() {

}

func live() People {
	var stu *Student
	return stu
}

func main() {
	if live() == nil {
		fmt.Println(&#34;AAAAAAA&#34;)
	} else {
		fmt.Println(&#34;BBBBBBB&#34;)
	}
}
```

**结果**

```
BBBBBBB
```

**分析：**

我们需要了解`interface`的内部结构，才能理解这个题目的含义。

interface在使用的过程中，共有两种表现形式

一种为**空接口(empty interface)**，定义如下：

```go
var MyInterface interface{}
```

另一种为**非空接口(non-empty interface)**, 定义如下：

```go
type MyInterface interface {
		function()
}
```

这两种interface类型分别用两种`struct`表示，空接口为`eface`, 非空接口为`iface`.
![img](https://img.kancloud.cn/8b/9a/8b9ad730048aceb7d7b7e3cf4631ad64_1920x1080.jpeg)

------

#### **空接口eface**

空接口eface结构，由两个属性构成，一个是类型信息_type，一个是数据信息。其数据结构声明如下：

```go
type eface struct {      //空接口
    _type *_type         //类型信息
    data  unsafe.Pointer //指向数据的指针(go语言中特殊的指针类型unsafe.Pointer类似于c语言中的void*)
}
```

**_type属性**：是GO语言中所有类型的公共描述，Go语言几乎所有的数据结构都可以抽象成 _type，是所有类型的公共描述，**type负责决定data应该如何解释和操作，**type的结构代码如下:

```go
type _type struct {
    size       uintptr  //类型大小
    ptrdata    uintptr  //前缀持有所有指针的内存大小
    hash       uint32   //数据hash值
    tflag      tflag
    align      uint8    //对齐
    fieldalign uint8    //嵌入结构体时的对齐
    kind       uint8    //kind 有些枚举值kind等于0是无效的
    alg        *typeAlg //函数指针数组，类型实现的所有方法
    gcdata    *byte
    str       nameOff
    ptrToThis typeOff
}
```

**data属性:** 表示指向具体的实例数据的指针，他是一个`unsafe.Pointer`类型，相当于一个C的万能指针`void*`。

![img](https://img.kancloud.cn/a5/dc/a5dc4728aa922c8bf1bc25e5252cdf49_1920x1080.jpeg)

------

#### 非空接口iface

iface 表示 non-empty interface 的数据结构，非空接口初始化的过程就是初始化一个iface类型的结构，其中`data`的作用同`eface`的相同，这里不再多加描述。

```go
type iface struct {
  tab  *itab
  data unsafe.Pointer
}
```

iface结构中最重要的是itab结构（结构如下），每一个 `itab` 都占 32 字节的空间。itab可以理解为`pair&lt;interface type, concrete type&gt;` 。itab里面包含了interface的一些关键信息，比如method的具体实现。

```go
type itab struct {
  inter  *interfacetype   // 接口自身的元信息
  _type  *_type           // 具体类型的元信息
  link   *itab
  bad    int32
  hash   int32            // _type里也有一个同样的hash，此处多放一个是为了方便运行接口断言
  fun    [1]uintptr       // 函数指针，指向具体类型所实现的方法
}
```

其中值得注意的字段，个人理解如下：

1. `interface type`包含了一些关于interface本身的信息，比如`package path`，包含的`method`。这里的interfacetype是定义interface的一种抽象表示。
2. `type`表示具体化的类型，与eface的 *type类型相同。*
3. `hash`字段其实是对`_type.hash`的拷贝，它会在interface的实例化时，用于快速判断目标类型和接口中的类型是否一致。另，Go的interface的Duck-typing机制也是依赖这个字段来实现。
4. `fun`字段其实是一个动态大小的数组，虽然声明时是固定大小为1，但在使用时会直接通过fun指针获取其中的数据，并且不会检查数组的边界，所以该数组中保存的元素数量是不确定的。

![img](https://img.kancloud.cn/bf/69/bf6927577682a3a1eadbef249ad3f24c_1920x1080.jpeg)

------

所以，People拥有一个Show方法的，属于非空接口，People的内部定义应该是一个`iface`结构体

```go
type People interface {
    Show()  
}  
```

![img](https://img.kancloud.cn/36/87/3687c0abd9da66c7ed58bbc2258cf00e_1920x1080.jpeg)

```go
func live() People {
    var stu *Student
    return stu      
}     
```

stu是一个指向nil的空指针，但是最后`return stu` 会触发`匿名变量 People = stu`值拷贝动作，所以最后`live()`放回给上层的是一个`People insterface{}`类型，也就是一个`iface struct{}`类型。 stu为nil，只是`iface`中的data 为nil而已。 但是`iface struct{}`本身并不为nil.

![img](https://img.kancloud.cn/af/13/af13c13498a74d3a9c90cf8cc208e4a0_1920x1080.jpeg)

所以如下判断的结果为`BBBBBBB`：

```go
func main() {   
    if live() == nil {  
        fmt.Println(&#34;AAAAAAA&#34;)      
    } else {
        fmt.Println(&#34;BBBBBBB&#34;)
    }
}
```

### (3) interface内部构造(空接口eface情况)

&gt; 下面代码结果为什么？

```go
func Foo(x interface{}) {
	if x == nil {
		fmt.Println(&#34;empty interface&#34;)
		return
	}
	fmt.Println(&#34;non-empty interface&#34;)
}
func main() {
	var p *int = nil
	Foo(p)
}
```

**结果**

```
non-empty interface
```

**分析**

不难看出，`Foo()`的形参`x interface{}`是一个空接口类型`eface struct{}`。

![img](https://img.kancloud.cn/39/37/3937d83d64ac00a29365513f1e9bece3_1920x1080.jpeg)

在执行`Foo(p)`的时候，触发`x interface{} = p`语句，所以此时 x结构如下。
![img](https://img.kancloud.cn/56/f5/56f50ff0127db53e6d28d3ce081e7520_1920x1080.jpeg)

所以 x 结构体本身不为nil，而是data指针指向的p为nil。

### (4) inteface{}与*interface{}

&gt; ABCD中哪一行存在错误？

```go
type S struct {
}

func f(x interface{}) {
}

func g(x *interface{}) {
}

func main() {
	s := S{}
	p := &amp;s
	f(s) //A
	g(s) //B
	f(p) //C
	g(p) //D
}
```

**结果**

```
B、D两行错误
B错误为： cannot use s (type S) as type *interface {} in argument to g:
	*interface {} is pointer to interface, not interface
	
D错误为：cannot use p (type *S) as type *interface {} in argument to g:
	*interface {} is pointer to interface, not interface
```

## 五、channel

### (1)Channel读写特性(15字口诀)

首先，我们先复习一下Channel都有哪些特性？

- 给一个 nil channel 发送数据，造成永远阻塞
- 从一个 nil channel 接收数据，造成永远阻塞
- 给一个已经关闭的 channel 发送数据，引起 panic
- 从一个已经关闭的 channel 接收数据，如果缓冲区中为空，则返回一个零值
- 无缓冲的channel是同步的，而有缓冲的channel是非同步的

以上5个特性是死东西，也可以通过口诀来记忆：“空读写阻塞，写关闭异常，读关闭空零”。

&gt; 执行下面的代码发生什么？

```go
package main

import (
	&#34;fmt&#34;
	&#34;time&#34;
)

func main() {
	ch := make(chan int, 1000)
	go func() {
		for i := 0; i &lt; 10; i&#43;&#43; {
			ch &lt;- i
		}
	}()
	go func() {
		for {
			a, ok := &lt;-ch
			if !ok {
				fmt.Println(&#34;close&#34;)
				return
			}
			fmt.Println(&#34;a: &#34;, a)
		}
	}()
	close(ch)
	fmt.Println(&#34;ok&#34;)
	time.Sleep(time.Second * 100)
}
```

15字口诀：“空读写阻塞，写关闭异常，读关闭空零”，往已经关闭的channel写入数据会panic的。因为main在开辟完两个goroutine之后，立刻关闭了ch， 结果不唯一：

```go
// 第二个协程中的打印有可能输出
panic: send on closed channel
```

## 六、WaitGroup

### (1) WaitGroup与goroutine的竞速问题

&gt; 编译并运行如下代码会发生什么？

```go
package main

import (
	&#34;sync&#34;
	//&#34;time&#34;
)

const N = 10

var wg = &amp;sync.WaitGroup{}

func main() {

	for i := 0; i &lt; N; i&#43;&#43; {
		go func(i int) {
			wg.Add(1)
			println(i)
			defer wg.Done()
		}(i)
	}
	wg.Wait()

}
```

**结果**

```
结果不唯一，代码存在风险, 所有go未必都能执行到
```

这是使用WaitGroup经常犯下的错误！请各位同学多次运行就会发现输出都会不同甚至又出现报错的问题。 这是因为`go`执行太快了，导致`wg.Add(1)`还没有执行main函数就执行完毕了。 改为如下试试

```go
package main

import (
	&#34;sync&#34;
)

const N = 10

var wg = &amp;sync.WaitGroup{}

func main() {

    for i:= 0; i&lt; N; i&#43;&#43; {
        wg.Add(1)
        go func(i int) {
            println(i)
            defer wg.Done()
        }(i)
    }

    wg.Wait()
}
```

---

> 作者:   
> URL: http://localhost:1313/lang/go/go_advanced/100.%E4%B9%A0%E9%A2%98%E7%BB%83%E4%B9%A0/  

