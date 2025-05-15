# 13.go 泛型

&gt; 在go语言里，对泛型的争议从未停止过，go也在1.18支持了泛型
&gt;
&gt; 原文参考：https://www.jb51.net/article/277511.htm

## 泛型初识

- 在强类型语言中（java，go），因为存在类型的强制约束，导致了数据类型在应用时没有弱类型语言（go、python）灵活

- 问题假如一个求和函数, 无法计算int类型之外的和.如果想计算浮点或者字符串的和该怎么办？

  ```go
  func Add(a int, b int) int {
      return a &#43; b
  }
  ```

- 泛型引入之前，解决办法之一就是为不同类型定义不同的函数

  ```go
  func AddFloat32(a float32, b float32) float32 {
      return a &#43; b
  }
   
  func AddString(a string, b string) string {
      return a &#43; b
  }
  ```

- 在引入泛型之后

  ```go
   func[T int | float32 | string](a, b T) T {
          return a &#43; b
  } 
  ```

  - 在上面这段伪代码中， T 被称为 类型形参(type parameter)，它不是具体的类型，在定义函数时类型并不确定。因为 T 的类型并不确定，所以需要像函数的形参那样，在调用函数的时候再传入具体的类型。这样我们不就能一个函数同时支持多个不同的类型了, 在这里被传入的具体类型被称为 类型实参(type argument)
  - **通过引入 类型形参 和 类型实参 这两个概念，让一个函数获得了处理多种不同类型数据的能力，这种编程方式被称为 泛型编程**

- 通过Go的 接口&#43;反射 不也能实现这样的动态数据处理吗？是的，泛型能实现的功能通过接口&#43;反射也基本能实现, 但是go的反射机制存在很多问题：用起来麻烦、失去了编译时的类型检查、性能不太理想

## 泛型基础概念

&gt; 通过引入 类型形参 和 类型实参 这两个概念，让一个函数获得了处理多种不同类型数据的能力，这种编程方式被称为 泛型编程
&gt;
&gt; Go在1.18 泛型中引入了很多全新的概念

![modb_20230311_b60ea10e-bfc8-11ed-864e-38f9d3cd240d](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-01/20230526230037.png)

- **类型形参 (Type parameter)**: 
  - `type Slice[T int|float32|float64 ] []T` T 就是类型形参(Type parameter)在定义Slice类型的时候 T 代表的具体类型并不确定，类似一个占位符
- **类型实参(Type argument)**
  - `var b Slice[float32] = []float32{1.0, 2.0, 3.0} ` float32 就是类型实参
- **类型形参列表( Type parameter list)**
  - 中括号里的`T int|float32|float64`  这一整串因为定义了所有的类型形参(在这个例子里只有一个类型形参T），所以我们称其为 类型形参列表(type parameter list)
- **类型约束(Type constraint)** 
  - 类型约束 指定了类型形参可接受的类型集合，只有属于这个集合中的类型才能替换形参用于实例化 
  - `int|float32|float64 ` 这部分被称为类型约束(Type constraint)，中间的 | 的意思是告诉编译器，类型形参 T 只可以接收 int 或 float32 或 float64 这三种类型的实参
- **泛型类型(Generic type)**
  - `type Slice[T int|float32|float64 ] []T`这种类型定义的方式中带了类型形参，将这种类型定义中带 **类型形参** 的类型，称之为 泛型类型(Generic type)
- **实例化(Instantiations)**
  - `var a Slice[int] = []int{1, 2, 3}` 泛型类型不能直接拿来使用，必须传入类型实参(Type argument) 将其确定为具体的类型之后才可使用。而传入类型实参确定具体类型的操作被称为 实例化(Instantiations) : 
- **泛型接收器(Generic receiver)**
- **泛型函数(Generic function)**
- ...

## 泛型基本使用

### 1. demo

```go
// 定义了一个普通的类型 Slice[int] ，它的底层类型是 []int
type Slice[int] []int     

// MyMap类型定义了两个类型形参 KEY 和 VALUE。分别为两个形参指定了不同的类型约束
// 这个泛型类型的名字叫： MyMap[KEY, VALUE]
type MyMap[KEY int | string, VALUE float32 | float64] map[KEY]VALUE  
 
// 用类型实参 string 和 flaot64 替换了类型形参 KEY 、 VALUE，泛型类型被实例化为具体的类型：MyMap[string, float64]
var a MyMap[string, float64] = map[string]float64{
    &#34;jack_score&#34;: 9.6,
    &#34;bob_score&#34;:  8.4,
}


// 一个泛型类型的结构体。可用 int 或 sring 类型实例化
type MyStruct[T int | string] struct {  
    Name string
    Data T
}

// 一个泛型接口(关于泛型接口在后半部分会详细讲解）
type IPrintData[T int | float32 | string] interface {
    Print(data T)
}

// 一个泛型通道，可用类型实参 int 或 string 实例化
type MyChan[T int | string] chan T

```

### 2. 类型形参的互相套用和约束

- 类型形参是可以互相套用

  ```go
  type WowStruct[T int | float32, S []T] struct {
      Data     S
      MaxValue T
      MinValue T
  }
  
  // 实例化
  var ws WowStruct[int, []int]   // 泛型类型 WowStuct[T, S] 被实例化后的类型名称就叫 WowStruct[int, []int]
  ```

- 上面为T传入了实参 `int`，然后因为 S 的定义是 `[]T` ，所以 S 的实参自然是 `[]int` ，实例化之后 WowStruct[T,S] 的定义类似如下

  ```go
  type WowStruct[int, []int] struct {
      Data     []int
      MaxValue int
      MinValue int
  }
  ```

- 下面两种定义作用一样

  ```go
  type WowStruct[T int|string] struct {
      Name string
      Data []T
  }
   
  type WowStruct2[T []int|[]string] struct {
      Name string
      Data T
  }
  ```

- 推荐使用方式

  ```go
  ype WowStruct3[T int | string] struct {
      Data     []T
      MaxValue T
      MinValue T
  }
  ```

  

### 3. 泛型的嵌套和约束

- 泛型和普通的类型一样，可以互相嵌套定义出更加复杂的新类型

  ```go
  // 先定义个泛型类型 Slice[T]
  type Slice[T int|string|float32|float64] []T
  
  // ✗ 错误。泛型类型Slice[T]的类型约束中不包含uint, uint8
  type UintSlice[T uint|uint8] Slice[T]  
  
  // ✓ 正确。基于泛型类型Slice[T]定义了新的泛型类型 FloatSlice[T] 。FloatSlice[T]只接受float32和float64两种类型
  type FloatSlice[T float32|float64] Slice[T] 
  
  // ✓ 正确。基于泛型类型Slice[T]定义的新泛型类型 IntAndStringSlice[T]
  type IntAndStringSlice[T int|string] Slice[T]  
  // ✓ 正确 基于IntAndStringSlice[T]套娃定义出的新泛型类型
  type IntSlice[T int] IntAndStringSlice[T] 
  
  // 在map中套一个泛型类型Slice[T]
  type WowMap[T int|string] map[string]Slice[T]
  // 在map中套Slice[T]的另一种写法
  type WowMap2[T Slice[int] | Slice[string]] map[string]T
  
  ```

### 4. 动态判断变量的类型

- 使用接口的时候经常会用到类型断言或 type swith 来确定接口具体的类型，然后对不同类型做出不同的处理; 

  ```go
  var i interface{} = 123
  i.(int) // 类型断言
   
  // type switch
  switch i.(type) {
      case int:
          // do something
      case string:
          // do something
      default:
          // do something
      }
  }
  ```

- 对于 valut T 这样通过类型形参定义的变量，我们能不能判断具体类型然后对不同类型做出不同处理呢？答案是不允许的

  ```go
  func (q *Queue[T]) Put(value T) {
      value.(int) // 错误。泛型类型定义的变量不能使用类型断言
   
      // 错误。不允许使用type switch 来判断 value 的具体类型
      switch value.(type) {
      case int:
          // do something
      case string:
          // do something
      default:
          // do something
      }
      
      // ...
  }
  ```

  

### 5. `~`  指定底层类型

```go
var s1 Slice[int] // 正确 
 
type MyInt int

var s2 Slice[MyInt] // ✗ 错误。MyInt类型底层类型是int但并不是int类型，不符合 Slice[T] 的类型约束
```

- 泛型类型 `Slice[T] `允许的是 `int` 作为类型实参，而不是 `MyInt` （虽然 `MyInt `类型底层类型是` int `，但它依旧不是 `int `类型）

- 为了解决这个问题，Go新增了一个符号 `~` ，在类型约束中使用类似 `~int` 这种写法的话，就代表着不光是 `int `，所有以 `int `为底层类型的类型也都可用于实例化

- `~`使用限制

  1. `~`**后面的类型不能为接口**

  2. `~`**后面的类型必须为基本类型**

- demo

  ```go
  type Int interface {
      ~int | ~int8 | ~int16 | ~int32 | ~int64
  }
   
  type Uint interface {
      ~uint | ~uint8 | ~uint16 | ~uint32
  }
  type Float interface {
      ~float32 | ~float64
  }
   
  type Slice[T Int | Uint | Float] []T 
   
  var s Slice[int] // 正确
   
  type MyInt int
  var s2 Slice[MyInt]  // MyInt底层类型是int，所以可以用于实例化
   
  type MyMyInt MyInt
  var s3 Slice[MyMyInt]  // 正确。MyMyInt 虽然基于 MyInt ，但底层类型也是int，所以也能用于实例化
   
  type MyFloat32 float32  // 正确
  var s4 Slice[MyFloat32]
  
  ----------------------------------
  type MyInt int
   
  type _ interface {
      ~[]byte  // 正确
      ~MyInt   // 错误，~后的类型必须为基本类型
      ~error   // 错误，~后的类型不能为接口
  }
  ```

  

### 6. 常见使用错误

- 定义泛型类型的时候，基础类型不能只有类型形参

  ```go
  // 错误，类型形参不能单独使用
  type CommonType[T int|string|float32] T
  ```

- 当类型约束的一些写法会被编译器误认为是表达式时会报错

  ```go
  //✗ 错误。T *int会被编译器误认为是表达式 T乘以int，而不是int指针
  type NewType[T *int] []T
  // 上面代码再编译器眼中：它认为你要定义一个存放切片的数组，数组长度由 T 乘以 int 计算得到
  type NewType [T * int][]T 
  
  //✗ 错误。和上面一样，这里不光*被会认为是乘号，| 还会被认为是按位或操作
  type NewType2[T *int|*float64] []T 
  
  //✗ 错误
  type NewType2 [T (int)] []T 
  	
  ///////////// 解决方式 ///////////// 
  
  // 推荐统一用 interface{} 解决问题
  type NewType[T interface{*int}] []T
  type NewType2[T interface{*int|*float64}] []T 
   
  // 如果类型约束中只有一个类型，可以添加个逗号消除歧义
  type NewType3[T *int,] []T
   
  //✗ 错误。如果类型约束不止一个类型，加逗号是不行的
  type NewType4[T *int|*float32,] []T 
  ```

- 匿名结构体不支持泛型

  ```go
  // 下面的用法是错误的：
  testCase := struct[T int|string] {
          caseName string
          got      T
          want     T
      }[int]{
          caseName: &#34;test OK&#34;,
          got:      100,
          want:     100,
      }
  ```

- 虽然type switch和类型断言不能用，但我们可通过反射机制达到目的：

  ```go
  func (receiver Queue[T]) Put(value T) {
      // Printf() 可输出变量value的类型(底层就是通过反射实现的)
      fmt.Printf(&#34;%T&#34;, value) 
   
      // 通过反射可以动态获得变量value的类型从而分情况处理
      v := reflect.ValueOf(value)
   
      switch v.Kind() {
      case reflect.Int:
          // do something
      case reflect.String:
          // do something
      }
   
      // ...
  }
  ```

  &gt; 为了避免使用反射而选择了泛型，结果到头来又为了一些功能在在泛型中使用反射, 当出现这种情况的时候你可能需要重新思考一下，自己的需求是不是真的需要用泛型（毕竟泛型机制本身就很复杂了，再加上反射的复杂度，增加的复杂度并不一定值得）

## 泛型接收器(receiver)

### 1. 泛型切片

- 单纯的泛型类型实际上对开发来说用处并不大。但是如果将泛型类型和接下来要介绍的泛型receiver相结合的话，泛型就有了非常大的实用性了

  ```go
  type MySlice[T int | float32] []T
   
  func (s MySlice[T]) Sum() T {
      var sum T
      for _, value := range s {
          sum &#43;= value
      }
      return sum
  }
  
  var s MySlice[int] = []int{1, 2, 3, 4}
  fmt.Println(s.Sum()) // 输出：10
   
  var s2 MySlice[float32] = []float32{1.0, 2.0, 3.0, 4.0}
  fmt.Println(s2.Sum()) // 输出：10.0
  ```

### 2. 泛型结构体

- 泛型结构体和泛型方法

  ```go
  package main
  
  import &#34;fmt&#34;
  
  type MyStruct[T int | string | float32] struct {
  	Res T
  }
  
  func (m *MyStruct[T]) Add(a ...T) {
  	var r T
  	for _, i := range a {
  		r &#43;= i
  	}
  	fmt.Println(r)
  }
  
  func main() {
  	m1 := MyStruct[int]{}
  	m1.Add(1, 2, 3, 4, 5)
  	m2 := MyStruct[float32]{}
  	m2.Add(1.0, 2.0, 3.1, 4.2, 5.1)
  	m3 := MyStruct[string]{}
  	m3.Add(&#34;1.0&#34;, &#34;2&#34;, &#34;3&#34;, &#34;4&#34;, &#34;5&#34;)
  }
  
  ```

### 3. 泛型队列

- 队列是一种先入先出的数据结构

  ```go
  // 这里类型约束使用了空接口，代表的意思是所有类型都可以用来实例化泛型类型 Queue[T] (关于接口在后半部分会详细介绍）
  type Queue[T interface{}] struct {
      elements []T
  }
   
  // 将数据放入队列尾部
  func (q *Queue[T]) Put(value T) {
      q.elements = append(q.elements, value)
  }
   
  // 从队列头部取出并从头部删除对应数据
  func (q *Queue[T]) Pop() (T, bool) {
     var value T
      if len(q.elements) == 0 {
          return value, true
      }
   
      value = q.elements[0]
      q.elements = q.elements[1:]
      return value, len(q.elements) == 0
  }
   
  // 队列大小
  func (q Queue[T]) Size() int {
      return len(q.elements)
  }
  -----------------------------------------------------
  
  var q1 Queue[int]  // 可存放int类型数据的队列
  q1.Put(1)
  q1.Put(2)
  q1.Put(3)
  q1.Pop() // 1
  q1.Pop() // 2
  q1.Pop() // 3
   
  var q2 Queue[string]  // 可存放string类型数据的队列
  q2.Put(&#34;A&#34;)
  q2.Put(&#34;B&#34;)
  q2.Put(&#34;C&#34;)
  q2.Pop() // &#34;A&#34;
  q2.Pop() // &#34;B&#34;
  q2.Pop() // &#34;C&#34;
   
  var q3 Queue[struct{Name string}] 
  var q4 Queue[[]int] // 可存放[]int切片的队列
  var q5 Queue[chan int] // 可存放int通道的队列
  var q6 Queue[io.Reader] // 可存放接口的队列
  ```

  

## 泛型函数

### 1. 泛型函数

- 泛型函数定义

  ```go
  func Add[T int | float32 | float64](a T, b T) T {
      return a &#43; b
  }
  ```

- 泛型函数调用

  1. 声明调用

     ```go
     Add[int](1,2) // 传入类型实参int，计算结果为 3
     Add[float32](1.0, 2.0) // 传入类型实参float32, 计算结果为 3.0
     ```

  2. Go还支持类型实参的自动推导

     ```go
     Add(1, 2)  // 1，2是int类型，编译请自动推导出类型实参T是int
     Add(1.0, 2.0) // 1.0, 2.0 是浮点，编译请自动推导出类型实参T是float32
     ```



### 2. 匿名函数

&gt; 匿名函数不支持泛型，匿名函数不能自己定义类型形参

- 匿名函数不能自己定义类型形参

  ```go
  // 错误，匿名函数不能自己定义类型实参
  fnGeneric := func[T int | float32](a, b T) T {
          return a &#43; b
  } 
  ```

- 匿名函数可以使用别处定义好的类型实参

  ```go
  func MyFunc[T int | float32 | float64](a, b T) {
      // 匿名函数可使用已经定义好的类型形参
      fn2 := func(i T, j T) T {
          return i*2 - j*2
      }
      fn2(a, b)
  }
  ```

### 3. 泛型方法

- 目前Go的方法并不支持泛型

  ```go
  type A struct {
  }
   
  // 错误 不支持泛型方法
  func (receiver A) Add[T int | float32 | float64](a T, b T) T {
      return a &#43; b
  }
  ```

- 但是receiver支持泛型， 所以通过receiver使用类型形参在方法中使用泛型的话

  ```go
  type A[T int | float32 | float64] struct {
  }
   
  // 方法可以使用类型定义中的形参 T 
  func (receiver A[T]) Add(a T, b T) T {
      return a &#43; b
  }
   
  // 用法：
  var a A[int]
  a.Add(1, 2)
   
  var aa A[float32]
  aa.Add(1.0, 2.0)
  ```

  

## 泛型接口

### 1. 泛型接口定义

- 有时候使用泛型编程时会书写长长的类型约束

  ```go
  // 一个可以容纳所有int,uint以及浮点类型的泛型切片
  type Slice[T int | int8 | int16 | int32 | int64 | uint | uint8 | uint16 | uint32 | uint64 | float32 | float64] []T
  
  ```

- 为了方便维护，go支持如下定义

  ```go
  type IntUintFloat interface {
      int | int8 | int16 | int32 | int64 | uint | uint8 | uint16 | uint32 | uint64 | float32 | float64
  }
  
  type Slice[T IntUintFloat] []T
  
  ```

- 也可以通过接口组合，更加灵活

  ```go
  type Int interface {
      int | int8 | int16 | int32 | int64
  }
  
  type Uint interface {
      uint | uint8 | uint16 | uint32
  }
  
  type Float interface {
      float32 | float64
  }
  
  type Slice[T Int | Uint | Float] []T  // 使用 &#39;|&#39; 将多个接口类型组合
  
  ```

- 接口嵌套

  ```go
  type SliceElement interface {
      Int | Uint | Float | string // 组合了三个接口类型并额外增加了一个 string 类型
  }
   
  type Slice[T SliceElement] []T 
  
  ```

### 2. 从方法集到类型集

- 在Go1.18之前，Go官方对 `接口(interface)` 的定义是：**接口是一个方法集(method set)**  `An interface type specifies a method set called its interface`.  

- `ReadWriter` 接口定义了一个接口(方法集)，这个集合中包含了 `Read()` 和 `Write()` 这两个方法。所有同时定义了这两种方法的类型被视为实现了这一接口。

  ```go
  type ReadWriter interface {
      Read(p []byte) (n int, err error)
      Write(p []byte) (n int, err error)
  }
  ```

- 如果换一个角度来重新思考上面这个接口的话，会发现接口的定义实际上还能这样理解：可以把 ReaderWriter 接口看成代表了一个 类型的集合，所有实现了 `Read() Writer()` 这两个方法的类型都在接口代表的类型集合当中。

- 通过换个角度看待接口，在我们眼中接口的定义就从 `方法集(method set)` 变为了 `类型集(type set)`。而Go1.18开始就是依据这一点将接口的定义正式更改为了 **类型集(Type set)**。

- 接口类型 Float 代表了一个 类型集合， 所有以 float32 或 float64 为底层类型的类型，都在这一类型集之中; 而 `type Slice[T Float] []T` 中， 类型约束 的真正意思是：类型约束 指定了类型形参可接受的类型集合，只有属于这个集合中的类型才能替换形参用于实例化

  ```go
  var s Slice[int]      // int 属于类型集 Float ，所以int可以作为类型实参
  var s Slice[chan int] // chan int 类型不在类型集 Float 中，所以错误
  
  ```

- **接口实现(implement)定义的变化**：既然接口定义发生了变化，那么从Go1.18开始 `接口实现(implement)` 的定义自然也发生了变化：当满足以下条件时，可以说 **类型 T 实现了接口 I ( type T implements interface I)：**
  1. **T 不是接口时：类型 T 是接口 I 代表的类型集中的一个成员 (T is an element of the type set of I)。**
  2. **T 是接口时： T 接口代表的类型集是 I 代表的类型集的子集(Type set of T is a subset of the type set of I)。**

### 3. 类型的交集和并集

- 并集`|`

  ```go
  type Uint interface {  // 类型集 Uint 是 ~uint 和 ~uint8 等类型的并集
      ~uint | ~uint8 | ~uint16 | ~uint32 | ~uint64
  }
  ```

- 交集

  ```go
  type AllInt interface {
      ~int | ~int8 | ~int16 | ~int32 | ~int64 | ~uint | ~uint8 | ~uint16 | ~uint32 | ~uint32
  }
  
  type Uint interface {
      ~uint | ~uint8 | ~uint16 | ~uint32 | ~uint64
  }
  
  type A interface { // 接口A代表的类型集是 AllInt 和 Uint 的交集 ~uint | ~uint8 | ~uint16 | ~uint32 | ~uint64
      AllInt
      Uint
  }
  
  type B interface { // 接口B代表的类型集是 AllInt 和 ~int 的交集  ~int
      AllInt
      ~int
  }
  
  ```

  

- 空集

  ```go
  type Bad interface {
      int
      float32 
  } // 类型 int 和 float32 没有相交的类型，所以接口 Bad 代表的类型集为空
  
  ```

  &gt; **没有任何一种类型属于空集**。虽然这样的写法是可以编译的，但实际上并没有什么意义。

### 4. interface{} 和 any

- Go1.18开始接口的定义发生了改变，所以 `interface{}` 的定义也发生了一些变更： 空接口代表了所有类型的集合

- 对于Go1.18之后的空接口应该这样理解：

  1. 虽然空接口内没有写入任何的类型，但它代表的是所有类型的集合，而非一个**空集**。
  2. 类型约束中指定 **空接口** 的意思是指定了一个包含所有类型的类型集，并不是类型约束限定了只能使用 **空接口** 来做类型形参。

  ```go
  // 空接口代表所有类型的集合。写入类型约束意味着所有类型都可拿来做类型实参
  type Slice[T interface{}] []T
  
  var s1 Slice[int]    // 正确
  var s2 Slice[map[string]string]  // 正确
  var s3 Slice[chan int]  // 正确
  var s4 Slice[interface{}]  // 正确
  ```

- 在Go1.18&#43;中，`any `等价于 `interface{}` , 这个是官方提供为了方便使用而设定的新的关键字

  ```go
  type Slice[T any] []T // 代码等价于 type Slice[T interface{}] []T
  ```

- `any` 的定义就位于Go语言的 `builtin.go` 文件中（参考如下）， `any` 实际上就是 `interaface{}` 的别名(alias)，两者完全等价。

  ```go
  // any is an alias for interface{} and is equivalent to interface{} in all ways.
  type any = interface{} 
  ```

- 所以从 Go 1.18 开始，所有可以用到空接口的地方其实都可以直接替换为any：

  ```go
  var s []any // 等价于 var s []interface{}
  var m map[string]any // 等价于 var m map[string]interface{}
  
  func MyPrint(value any){
      fmt.Println(value)
  }
  ```

- 如果你高兴的话，项目迁移到 Go1.18 之后可以使用下面这行命令直接把整个项目中的空接口全都替换成 any。当然因为并不强制，所以到底是用 `interface{}` 还是 `any` 全看自己喜好。

  ```go
  gofmt -w -r &#39;interface{} -&gt; any&#39; ./...
  ```

- Go语言项目中就曾经有人提出过把Go语言中所有 interface{ }替换成 any 的 issue，然后因为影响范围过大过而且影响因素不确定，理所当然被驳回了。

### 5. comparable和 ordered

&gt; **comparable(可比较) 和 可排序(ordered)**

- 对于一些数据类型，我们需要在类型约束中限制只接受能 `!=` 和 `==` 对比的类型，以Go直接内置了一个叫 `comparable` 的接口，它代表了所有可用 `!=` 以及 `==` 对比的类型：

  ```go
  // 错误。因为 map 中键的类型必须是可进行 != 和 == 比较的类型
  type MyMap[KEY any, VALUE any] map[KEY]VALUE 
  
  // 正确
  type MyMap[KEY comparable, VALUE any] map[KEY]VALUE 
  ```

- `comparable` 比较容易引起误解的一点是很多人容易把他与可排序搞混淆。可比较指的是 可以执行 `!= ==` 操作的类型，并没确保这个类型可以执行大小比较（ `&gt;,&lt;,&lt;=,&gt;= `）。、

  ```go
  type OhMyStruct struct {
      a int
  }
  
  var a, b OhMyStruct
  
  a == b // 正确。结构体可使用 == 进行比较
  a != b // 正确
  
  a &gt; b // 错误。结构体不可比大小
  
  ```

- 而可进行大小比较的类型被称为 `Orderd` 。目前Go语言并没有像 `comparable` 这样直接内置对应的关键词，所以想要的话需要自己来定义相关接口，比如我们可以参考Go官方包 `golang.org/x/exp/constraints` 如何定义：

  ```go
  // Ordered 代表所有可比大小排序的类型
  type Ordered interface {
      Integer | Float | ~string
  }
  
  type Integer interface {
      Signed | Unsigned
  }
  
  type Signed interface {
      ~int | ~int8 | ~int16 | ~int32 | ~int64
  }
  
  type Unsigned interface {
      ~uint | ~uint8 | ~uint16 | ~uint32 | ~uint64 | ~uintptr
  }
  
  type Float interface {
      ~float32 | ~float64
  }
  
  ```

- 虽然可以直接使用官方包 `golang.org/x/exp/constraints` ，但因为这个包属于实验性质的 x 包，今后可能会发生非常大变动，所以并不推荐直接使用

### 6. 接口的两种类型

- Go1.18开始将接口分为了两种类型
  1. **基本接口(Basic interface)**
  2. **通用接口(General interface)**

- **基本接口(Basic interface)**:接口定义中如果只有方法的话，那么这种接口被称为基本接口(Basic interface)。这种接口就是Go1.18之前的接口，用法也基本和Go1.18之前保持一致。

  ```go
  type MyError interface { // 接口中只有方法，所以是基本接口
      Error() string
  }
  
  // 用法和 Go1.18之前保持一致
  var err MyError = fmt.Errorf(&#34;hello world&#34;)
  
  // io.Reader 和 io.Writer 都是基本接口，也可以用在类型约束中
  type MySlice[T io.Reader | io.Writer]  []Slice
  ```

  

- **通用接口(General interface)**:果接口内不光只有方法，还有类型的话，这种接口被称为 通用接口(General interface) ；**通用接口，只能用于类型约束，不得用于变量定义**， 这一限制保证了一般接口的使用被限定在了泛型之中，不会影响到Go1.18之前的代码，同时也极大减少了书写代码时的心智负担

  ```go
  type Uint interface { // 接口 Uint 中有类型，所以是通用接口
      ~uint | ~uint8 | ~uint16 | ~uint32 | ~uint64
  }
  
  type ReadWriter interface {  // ReadWriter 接口既有方法也有类型，所以是通用接口
      ~string | ~[]rune
  
      Read(p []byte) (n int, err error)
      Write(p []byte) (n int, err error)
  }
  
  // 类型 StringReadWriter 实现了接口 Readwriter
  type StringReadWriter string 
   
  func (s StringReadWriter) Read(p []byte) (n int, err error) {
      // ...
  }
   
  func (s StringReadWriter) Write(p []byte) (n int, err error) {
   // ...
  }
   
  // 类型BytesReadWriter 没有实现接口 Readwriter  
  // StringReadWriter 存在于接口 ReadWriter 代表的类型集中，而 BytesReadWriter 因为底层类型是 []byte（既不是string也是不[]rune） ，所以它不属于 ReadWriter 代表的类型集
  type BytesReadWriter []byte 
   
  func (s BytesReadWriter) Read(p []byte) (n int, err error) {
   ...
  }
   
  func (s BytesReadWriter) Write(p []byte) (n int, err error) {
   ...
  }
  
  --------------------------------
  type Uint interface {
      ~uint | ~uint8 | ~uint16 | ~uint32 | ~uint64
  }
  
  var uintInf Uint // 错误。Uint是通用接口，只能用于类型约束，不得用于变量定义  
  
  ```

## go泛型的其他限制

&gt; 1. 用 `|` 连接多个类型的时候，类型之间不能有相交的部分(即必须是不交集）; 但是相交的类型中是接口的话，则不受这一限制
&gt; 2. 类型的并集中不能有类型形参
&gt; 3. 接口不能直接或间接地并入自己
&gt; 4. 接口的并集成员个数大于一的时候不能直接或间接并入 comparable 接口
&gt; 5. 带方法的接口(无论是基本接口还是一般接口)，都不能写入接口的并集中

1. **用 `|` 连接多个类型的时候，类型之间不能有相交的部分(即必须是不交集）; 但是相交的类型中是接口的话，则不受这一限制：**

   ```go
   type MyInt int
   
   // 错误，MyInt的底层类型是int,和 ~int 有相交的部分
   type _ interface {
       ~int | MyInt
   }
   
   type MyInt int
   
   type _ interface {
       ~int | interface{ MyInt }  // 正确
   }
   
   type _ interface {
       interface{ ~int } | MyInt // 也正确
   }
   
   type _ interface {
       interface{ ~int } | interface{ MyInt }  // 也正确
   }
   
   ```

2. **类型的并集中不能有类型形参**

   ```go
   type MyInf[T ~int | ~string] interface {
       ~float32 | T  // 错误。T是类型形参
   }
   
   type MyInf2[T ~int | ~string] interface {
       T  // 错误
   }
   
   ```

3. **接口不能直接或间接地并入自己**

   ```go
   type Bad interface {
       Bad // 错误，接口不能直接并入自己
   }
   
   type Bad2 interface {
       Bad1
   }
   type Bad1 interface {
       Bad2 // 错误，接口Bad1通过Bad2间接并入了自己
   }
   
   type Bad3 interface {
       ~int | ~string | Bad3 // 错误，通过类型的并集并入了自己
   }
   
   ```

4. **接口的并集成员个数大于一的时候不能直接或间接并入 comparable 接口**

   ```go
   type OK interface {
       comparable // 正确。只有一个类型的时候可以使用 comparable
   }
   
   type Bad1 interface {
       []int | comparable // 错误，类型并集不能直接并入 comparable 接口
   }
   
   type CmpInf interface {
       comparable
   }
   type Bad2 interface {
       chan int | CmpInf  // 错误，类型并集通过 CmpInf 间接并入了comparable
   }
   type Bad3 interface {
       chan int | interface{comparable}  // 理所当然，这样也是不行的
   }
   
   ```

5. **带方法的接口(无论是基本接口还是一般接口)，都不能写入接口的并集中**

   ```go
   type _ interface {
       ~int | ~string | error // 错误，error是带方法的接口(一般接口) 不能写入并集中
   }
   
   type DataProcessor[T any] interface {
       ~string | ~[]byte
   
       Process(data T) (newData T)
       Save(data T) error
   }
   
   // 错误，实例化之后的 DataProcessor[string] 是带方法的一般接口，不能写入类型并集
   type _ interface {
       ~int | ~string | DataProcessor[string] 
   }
   
   type Bad[T any] interface {
       ~int | ~string | DataProcessor[T]  // 也不行
   }
   
   ```

   


---

> 作者: [Fred](https://github.com/ipfred)  
> URL: https://ipfred.github.io/lang/go/go_base/20250515175146/  

