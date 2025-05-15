# 4-2. 接口

# 接口

- 接口（interface）定义了一个对象的行为规范，只定义规范不实现，由具体的对象来实现规范的细节。
- 牢记**接口（interface）是一种类型**。

- 接口类型是一种抽象的类型, 不会暴露出他所代表的内部属性的结构, 只会展示出他自己的方法,因此接口不能被实例化;

- **Golang中的接口没有数据字段, 只有定义的方法;**接口是方法的集合

- 接口命名规范:

  - 单个函数的接口名以&#34;er&#34;作为后缀, 接口的实现则去掉&#34;er&#34;
  - 两个函数的接口名综合两个函数名, 以&#34;er&#34;作为后缀, 接口的实现则去掉&#34;er&#34;
  - 三个以上函数以上的接口名, 抽象这个接口的功能, 类似于结构体命名;


## 1. 鸭子类型

- 鸭子模型: 当看到一只鸟走起来像鸭子,游泳像鸭子, 叫起来像鸭子, 那这只鸟就可以被称为鸭子;

- 在程序设计中, 鸭子模型 是动态类型语言的一种风格;**一个对象有效的语义, 不是由继承自特定的类或实现特定的接口来决定, 而是由当前方法和属性的集合决定;**
- go语言通过接口实现了鸭子模型, 对于Go来说, 我们不关心对象是什么类型,或者到底是不是鸭子, 我们只关心行为, 关心他是如何使用的.
- 一般来说, 静态类型语言在编译时便已经确定了变量的类型, 但是go是在编译时推断变量的类型

## 2. 接口的定义

Go语言提倡面向接口编程。

每个接口由数个方法组成，接口的定义格式如下：

```go
type 接口类型名 interface{
    方法名1( 参数列表1 ) 返回值列表1
    方法名2( 参数列表2 ) 返回值列表2
    …
}
```

其中：

- 接口名：使用`type`将接口定义为自定义的类型名。Go语言的接口在命名时，一般会在单词后面添加`er`，如有写操作的接口叫`Writer`，有字符串功能的接口叫`Stringer`等。接口名最好要能突出该接口的类型含义。
- 方法名：当方法名首字母是大写且这个接口类型名首字母也是大写时，这个方法可以被接口所在的包（package）之外的代码访问。
- 参数列表、返回值列表：参数列表和返回值列表中的参数变量名可以省略。

举个例子：

```go
type writer interface{
    Write([]byte) error
}
```

当你看到这个接口类型的值时，你不知道它是什么，唯一知道的就是可以通过它的Write方法来做一些事情。

## 3. 接口的实现

- 一个对象只要全部实现了接口中的方法，那么就实现了这个接口。换句话说，接口就是一个**需要实现的方法列表**。

- 一个类型可以实现多个接口, 多个类型也可以实现同一个接口, **任何一个类型都实现了空接口**;

```go
package main

import &#34;fmt&#34;

type Animal interface {
	Talk()
	Eat()
	Name() string
}

type Dog struct {
}

func (d Dog) Talk()  {
	fmt.Println(&#34;汪汪汪&#34;)

}

func (d Dog) Eat()  {
	fmt.Println(&#34;我在吃骨头&#34;)

}
func (d Dog) Name() string   {
	fmt.Println(&#34;我的名字叫旺财&#34;)
	return &#34;旺财&#34;

}

func main() {
	var d Dog  // Dog 中需要全部实现AnimaI的方法集合
	var a Animal
  a = d  // 接口赋值
	a.Name()
	a.Eat()
	a.Talk()
}
```

## 4. 接口类型变量

那实现了接口有什么用呢？

接口类型变量能够存储所有实现了该接口的实例。 例如上面的示例中，`Sayer`类型的变量能够存储`dog`和`cat`类型的变量。

```go
func main() {
	var x Sayer // 声明一个Sayer类型的变量x
	a := cat{}  // 实例化一个cat
	b := dog{}  // 实例化一个dog
	x = a       // 可以把cat实例直接赋值给x
	x.say()     // 喵喵喵
	x = b       // 可以把dog实例直接赋值给x
	x.say()     // 汪汪汪
}
```

**Tips：** 观察下面的代码，体味此处`_`的妙用

```go
// 摘自gin框架routergroup.go
type IRouter interface{ ... }

type RouterGroup struct { ... }

var _ IRouter = &amp;RouterGroup{}  // 确保RouterGroup实现了接口IRouter
```



## 5. 接口赋值

- 用户自定义的类型实现了某个接口类型的所有方法, 那么这个用户自定义的类型的值就可以赋值给这个接口. 这个赋值会把用户定义类型的值存入到接口类型变量中;
- **接口变量**: 接口变量存储了两部分信息, 一部分是分配给接口变量的具体值(接口实现者的值), 二是值得类型的描述器(接口实现者的类型)
- 接口赋值存在两种情况
  1. 将对象实例赋值给接口
  2. 将一个接口赋值给另一个接口
     - 只要两个接口拥有相同的方法集就可以, 就可以相互赋值
     - 如果两个接口方法集不同, 接口A的方法集是接口B的方法集的子集, 那么接口A可以赋值给接口B, 反之不成立

- 将对象实例赋值给接口

```go
package main

import &#34;fmt&#34;

type IDatabaser interface {
	Connect() error
	DisConnect() error
}

type Redis struct {
	DBName string
}

func (redis Redis) Connect() error  {
	fmt.Println(&#34;redis connect DB=&gt; &#34; &#43; redis.DBName)
	fmt.Println(&#34;redis connect success!&#34;)
	return nil
}

func (redis Redis) DisConnect() error  {
	fmt.Println(&#34;redis disconnect success!&#34;)
	return nil
}

func main() {
	var redisObj = Redis{DBName: &#34;Evan&#34;}
	var idb IDatabaser  // 接口变量
	idb = redisObj    //接口赋值: redisObj这个类型实例实现了接口中的所有方法
	_ = idb.Connect()
	_ = idb.DisConnect()
}
/**
redis connect DB=&gt; Evan
redis connect success!
redis disconnect success!
*/
```

## 6. 接口嵌套(继承)

- 接口的嵌入也叫接口组合, 在python中叫继承如果接口A作为接口B的一个嵌入字段, 那么接口B中就包含了接口A中的所有方法
  - 一个接口只接受其它接口的嵌入, 嵌入结构体和其他类型会报错
  - 一个接口不能嵌入接口本身, 包括直接嵌入和间接嵌入

```go

type Peopler1 interface {
	GetName() string
}
type Peopler2 interface {
	Peopler1
	GetAge() int
}
type Peopler3 interface {
	Peopler2
	GetHeight() int
}

type People struct {
	name string
	age int
	height int
}

func (p People) GetHeight() int {
	return p.height
}
func (p People) GetName() string {
	return p.name
}
func (p People) GetAge() int {
	return p.age
}

func main() {
	var p = People{&#34;evan&#34;,18,180}
	fmt.Println(p.GetName())
	fmt.Println(p.GetAge())
	fmt.Println(p.GetHeight())
	var p3 Peopler3
	p3 = p
	fmt.Println(p3.GetName())
	fmt.Println(p3.GetAge())
	fmt.Println(p3.GetHeight())
}

```

## 7. 空接口

- 空接口 `interface{}`是golang中最特殊的接口, **在python3中,任何一个类都会继承Object基类, 而go中的空接口interface{}类似于python3中的基类Object**
- 空接口中不包含热河方法, 所以类型其实都实现了空接口;空接口可以接收任何值;
- 不能将空接口赋值到其他类型, 如果需要这么做必须使用类型断言. 

```go
package main

import &#34;fmt&#34;

func main() {
	var a = &#34;1&#34;
	var nullInterface interface{} = a
	var b string = nullInterface.(string)  // 类型断言,接口类型向普通类型转换
	fmt.Println(b)
}
```

### 7.1 空接口的应用

1. **空接口作为函数的参数**

使用空接口实现可以接收任意类型的函数参数。

```go
// 空接口作为函数参数
func show(a interface{}) {
	fmt.Printf(&#34;type:%T value:%v\n&#34;, a, a)
}
```

2. **空接口作为map的值**

使用空接口实现可以保存任意值的字典。

```go
// 空接口作为map值
	var studentInfo = make(map[string]interface{})
	studentInfo[&#34;name&#34;] = &#34;沙河娜扎&#34;
	studentInfo[&#34;age&#34;] = 18
	studentInfo[&#34;married&#34;] = false
	fmt.Println(studentInfo)
```

## 8. 断言

- 断言是使用在接口变量上的操作, 简单来说,接口类型向普通类型转换就是类型断言

- 断言语法

  ```go
  t, ok := X.(T)
  // 判断接口变量X的类型是不是T, 如果断言成功,则ok为true,t的值为X动态值; 否则ok为false,t的值为类型T的初始值, t的类型始终是T
  ```

- 断言有两种方式

  1. Ok-pattern 适用于接口类型较少

     ```go
     if value , ok := X.(T); ok ==true{
       
     }
     ```

  2. switch-type  要断言的接口类型较多

     ```go
     switch value := 接口变量.(type){
       case type1:
       	// dosomething
       case type2:
       	// dosomething
       default:
       	// 当接口类型没有被捕获到的时候do
     }
     ```
  
3. 示例

   ```go
   package main
   
   import (
   	&#34;fmt&#34;
   )
   
   type e interface {}
   
   func Map(f func(a e)e ,arr []e) []e{
   	res := make([]e,0,1)
   	for _ ,item := range arr{
   		_res := f(item)
   		res = append(res,_res)
   	}
   	return res
   }
   
   
   func main() {
   	a:= []e{&#34;1&#34;,&#34;2&#34;,&#34;3&#34;,1,2,3}
   	b:= []e{1,2,3,4,5}
   
   	fu := func(arg e)e{
   		var res e
   		switch arg.(type) {  // 断言
   		case string:
   			res =  arg.(string)&#43;&#34;--&#34;
   		case int:
   			res =  arg.(int)* 2
   		}
   		return res
   	}
   	res:= Map(fu,a)
   	fmt.Println(res)
   	res = Map(fu,b)
   	fmt.Println(res)
   }
   /**
   *[1-- 2-- 3-- 2 4 6]
   *[2 4 6 8 10]
   */
   ```

   

## 9. 侵入式接口和非侵入式接口

- 侵入式接口和非侵入式接口的区别
  - 侵入式接口: 需要显式的创建一个类去实现某一个接口
  - 非侵入式接口: 不需要显式的创建一个类去实现接口; 非侵入式更加简洁灵活, 更注重实用性.

- 在golang中, 接口是非侵入式的, 非侵入式的优点: 
  1. 去掉了繁琐的继承体系, golang中类的继承并无意义, 你只需要知道这个类实现了哪些方法, 每个方法是何含义就可以了;
  2. 实现类的接口时, 只需要关心自己应该提供哪些方法, 不用纠结接口需要拆分的多细才合适, 接口由使用方按需定义, 而不用事前规划;
  3. 不用为了实现一个接口而导入一个包, 多引用一个外部的包, 就意味着更多的耦合, 接口由使用方按需定义, 使用方不用关心是否有其他模块定义过类似的接口

- 代码

  ```go
  package main
  
  import (
  	&#34;fmt&#34;
  )
  
  type IPhoner interface {
  	Call() error
  	Video() error
  	Game() error
  }
  
  type Phone struct {
  	Name string
  }
  
  func (p *Phone) Call() error  {
  	fmt.Println(p.Name , &#34;start call&#34;)
  	return nil
  }
  
  func (p *Phone) Video() error{
  	fmt.Println(p.Name , &#34;start video&#34;)
  	return nil
  }
  
  func (p *Phone) Game() error{
  	fmt.Println(p.Name, &#34;start game&#34;)
  	return nil
  }
  
  func main() {
  	var iphone IPhoner = &amp;Phone{Name: &#34;iphone&#34;}
  	var huawei IPhoner = &amp;Phone{Name: &#34;huawei&#34;}
  	iphone.Call()
  	huawei.Game()
  
  }
  ```

  

## 10. 值接收者和指针接收者实现接口的区别

使用值接收者实现接口和使用指针接收者实现接口有什么区别呢？接下来我们通过一个例子看一下其中的区别。

我们有一个`Mover`接口和一个`dog`结构体。

```go
type Mover interface {
	move()
}

type dog struct {}
```

### 10.1 值接收者实现接口

```go
func (d dog) move() {
	fmt.Println(&#34;狗会动&#34;)
}
```

此时实现接口的是`dog`类型：

```go
func main() {
	var x Mover
	var wangcai = dog{} // 旺财是dog类型
	x = wangcai         // x可以接收dog类型
	var fugui = &amp;dog{}  // 富贵是*dog类型
	x = fugui           // x可以接收*dog类型
	x.move()
}
```

从上面的代码中我们可以发现，使用值接收者实现接口之后，不管是dog结构体还是结构体指针*dog类型的变量都可以赋值给该接口变量。因为Go语言中有对指针类型变量求值的语法糖，dog指针`fugui`内部会自动求值`*fugui`。

### 10.2 指针接收者实现接口

同样的代码我们再来测试一下使用指针接收者有什么区别：

```go
func (d *dog) move() {
	fmt.Println(&#34;狗会动&#34;)
}
func main() {
	var x Mover
	var wangcai = dog{} // 旺财是dog类型
	x = wangcai         // x不可以接收dog类型
	var fugui = &amp;dog{}  // 富贵是*dog类型
	x = fugui           // x可以接收*dog类型
}
```

此时实现`Mover`接口的是`*dog`类型，所以不能给`x`传入`dog`类型的wangcai，此时x只能存储`*dog`类型的值。



## 11. 类型与接口的关系

### 11.1 一个类型实现多个接口

一个类型可以同时实现多个接口，而接口间彼此独立，不知道对方的实现。 例如，狗可以叫，也可以动。我们就分别定义Sayer接口和Mover接口，如下： `Mover`接口。

```go
// Sayer 接口
type Sayer interface {
	say()
}

// Mover 接口
type Mover interface {
	move()
}
```

dog既可以实现Sayer接口，也可以实现Mover接口。

```go
type dog struct {
	name string
}

// 实现Sayer接口
func (d dog) say() {
	fmt.Printf(&#34;%s会叫汪汪汪\n&#34;, d.name)
}

// 实现Mover接口
func (d dog) move() {
	fmt.Printf(&#34;%s会动\n&#34;, d.name)
}

func main() {
	var x Sayer
	var y Mover

	var a = dog{name: &#34;旺财&#34;}
	x = a
	y = a
	x.say()
	y.move()
}
```

### 11.2 多个类型实现同一接口(多态)

Go语言中不同的类型还可以实现同一接口 首先我们定义一个`Mover`接口，它要求必须由一个`move`方法。

```go
// Mover 接口
type Mover interface {
	move()
}
```

例如狗可以动，汽车也可以动，可以使用如下代码实现这个关系：

```go
type dog struct {
	name string
}

type car struct {
	brand string
}

// dog类型实现Mover接口
func (d dog) move() {
	fmt.Printf(&#34;%s会跑\n&#34;, d.name)
}

// car类型实现Mover接口
func (c car) move() {
	fmt.Printf(&#34;%s速度70迈\n&#34;, c.brand)
}
```

这个时候我们在代码中就可以把狗和汽车当成一个会动的物体来处理了，不再需要关注它们具体是什么，只需要调用它们的`move`方法就可以了。

```go
func main() {
	var x Mover
	var a = dog{name: &#34;旺财&#34;}
	var b = car{brand: &#34;保时捷&#34;}
	x = a
	x.move()
	x = b
	x.move()
}
```

上面的代码执行结果如下：

```go
旺财会跑
保时捷速度70迈
```

并且一个接口的方法，不一定需要由一个类型完全实现，接口的方法可以通过在类型中嵌入其他类型或者结构体来实现。

```go
// WashingMachine 洗衣机
type WashingMachine interface {
	wash()
	dry()
}

// 甩干器
type dryer struct{}

// 实现WashingMachine接口的dry()方法
func (d dryer) dry() {
	fmt.Println(&#34;甩一甩&#34;)
}

// 海尔洗衣机
type haier struct {
	dryer //嵌入甩干器
}

// 实现WashingMachine接口的wash()方法
func (h haier) wash() {
	fmt.Println(&#34;洗刷刷&#34;)
}
```



---

> 作者: [Fred](https://github.com/ipfred)  
> URL: https://ipfred.github.io/lang/go/go_base/20250515175100/  

