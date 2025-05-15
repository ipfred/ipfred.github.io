# 4-1. 结构体

# 结构体

- Go语言中没有“类”的概念，也不支持“类”的继承等面向对象的概念。Go语言中通过结构体的内嵌再配合接口比面向对象具有更高的扩展性和灵活性。
- Go语言中的基础数据类型可以表示一些事物的基本属性，但是当我们想表达一个事物的全部或部分属性时，这时候再用单一的基本数据类型明显就无法满足需求了，Go语言提供了一种自定义数据类型，可以封装多个基本数据类型，这种数据类型叫结构体，英文名称`struct`。 也就是我们可以通过`struct`来定义自己的类型了。
- **Go语言中通过`struct`来实现面向对象。**

## 0. Struct转换成map

### 0.1 Struct转json再转map

```go
mport (
    &#34;encoding/json&#34;
    &#34;fmt&#34;
    &#34;reflect&#34;
    &#34;time&#34;
)

type Persion struct {
    Id       int
    Name     string
    Address  string
}

func main() {
    StructToMapViaJson()
    //StructToMapViaReflect()
}

func StructToMapViaJson() {
    m := make(map[string]interface{})
    t := time.Now()
    person := Persion{
        Id:       98439,
        Name:     &#34;zhaondifnei&#34;,
        Address:  &#34;大沙地&#34;,
    }
    j, _ := json.Marshal(person)
    json.Unmarshal(j, &amp;m)
    fmt.Println(m)
    fmt.Println(time.Now().Sub(t))
}
```



### 0.2 反射Struct 转map

&gt; 是方法一速度的两倍

```go
func StructToMapViaReflect() {
    m := make(map[string]interface{})
    t := time.Now()
    person := Persion{
        Id:       98439,
        Name:     &#34;zhaondifnei&#34;,
        Address:  &#34;大沙地&#34;,
    }
    elem := reflect.ValueOf(&amp;person).Elem()
    relType := elem.Type()
    for i := 0; i &lt; relType.NumField(); i&#43;&#43; {
        m[relType.Field(i).Name] = elem.Field(i).Interface()
    }
    fmt.Println(m)
    fmt.Printf(&#34;duration:%d&#34;, time.Now().Sub(t))
}
```



## 1. 结构体的定义

- 使用`type`和`struct`关键字来定义结构体，具体代码格式如下：

  ```go
  type 类型名 struct {
      字段名 字段类型
      字段名 字段类型
      …
  }
  ```

  - 类型名：标识自定义结构体的名称，在同一个包内不能重复。
  - 字段名：表示结构体字段名。结构体中的字段名必须唯一。
  - 字段类型：表示结构体字段的具体类型。

- 举个例子，我们定义一个`Person`（人）结构体，代码如下：

  ```go
  type person struct {
  	name string
  	city string
  	age  int8
  }
  ```

  

- 同样类型的字段也可以写在一行，

  ```go
  type person1 struct {
  	name, city string
  	age        int8
  }
  ```

&gt; 结构体中字段大写开头表示可公开访问，小写表示私有（仅在定义当前结构体的包中可访问）。

## 2. 结构体实例化

- 只有当结构体实例化时，才会真正地分配内存。也就是必须实例化后才能使用结构体的字段。

- 结构体本身也是一种类型，我们可以像声明内置类型一样使用`var`关键字声明结构体类型。

```go
var 结构体实例 结构体类型
```

### 2.1 基本实例化

```go
type person struct {
	name string
	city string
	age  int8
}

func main() {
	var p1 person
	p1.name = &#34;沙河娜扎&#34;
	p1.city = &#34;北京&#34;
	p1.age = 18
	fmt.Printf(&#34;p1=%v\n&#34;, p1)  //p1={沙河娜扎 北京 18}
	fmt.Printf(&#34;p1=%#v\n&#34;, p1) //p1=main.person{name:&#34;沙河娜扎&#34;, city:&#34;北京&#34;, age:18}
}
```

- 我们通过`.`来访问结构体的字段（成员变量）,例如`p1.name`和`p1.age`等。

### 2.2 匿名结构体

- 在定义一些临时数据结构等场景下还可以使用匿名结构体。

```go
package main
     
import (
    &#34;fmt&#34;
)
     
func main() {
    var user struct{Name string; Age int}
    user.Name = &#34;小王子&#34;
    user.Age = 18
    fmt.Printf(&#34;%#v\n&#34;, user)
}
```

### 2.3 创建指针类型结构体

- 我们还可以通过使用`new`关键字对结构体进行实例化，得到的是结构体的地址。 格式如下：

```go
var p2 = new(person)
fmt.Printf(&#34;%T\n&#34;, p2)     //*main.person
fmt.Printf(&#34;p2=%#v\n&#34;, p2) //p2=&amp;main.person{name:&#34;&#34;, city:&#34;&#34;, age:0}
```

- 需要注意的是在Go语言中支持对结构体指针直接使用`.`来访问结构体的成员。

```go
var p2 = new(person)
p2.name = &#34;小王子&#34;
p2.age = 28
p2.city = &#34;上海&#34;
fmt.Printf(&#34;p2=%#v\n&#34;, p2) //p2=&amp;main.person{name:&#34;小王子&#34;, city:&#34;上海&#34;, age:28}
```

### 2.4 取结构体的地址实例化

使用`&amp;`对结构体进行取地址操作相当于对该结构体类型进行了一次`new`实例化操作。

```go
p3 := &amp;person{}
fmt.Printf(&#34;%T\n&#34;, p3)     //*main.person
fmt.Printf(&#34;p3=%#v\n&#34;, p3) //p3=&amp;main.person{name:&#34;&#34;, city:&#34;&#34;, age:0}
p3.name = &#34;七米&#34;
p3.age = 30
p3.city = &#34;成都&#34;
fmt.Printf(&#34;p3=%#v\n&#34;, p3) //p3=&amp;main.person{name:&#34;七米&#34;, city:&#34;成都&#34;, age:30}
```

`p3.name = &#34;七米&#34;`其实在底层是`(*p3).name = &#34;七米&#34;`，这是Go语言帮我们实现的语法糖。



## 3. 结构体初始化

没有初始化的结构体，其成员变量都是对应其类型的零值。

```go
type person struct {
	name string
	city string
	age  int8
}

func main() {
	var p4 person
	fmt.Printf(&#34;p4=%#v\n&#34;, p4) //p4=main.person{name:&#34;&#34;, city:&#34;&#34;, age:0}
}
```

### 3.1 使用键值对初始化

- 使用键值对对结构体进行初始化时，键对应结构体的字段，值对应该字段的初始值。

```go
p5 := person{
	name: &#34;小王子&#34;,
	city: &#34;北京&#34;,
	age:  18,
}
fmt.Printf(&#34;p5=%#v\n&#34;, p5) //p5=main.person{name:&#34;小王子&#34;, city:&#34;北京&#34;, age:18}
```

- 也可以对结构体指针进行键值对初始化，例如：

```go
p6 := &amp;person{
	name: &#34;小王子&#34;,
	city: &#34;北京&#34;,
	age:  18,
}
fmt.Printf(&#34;p6=%#v\n&#34;, p6) //p6=&amp;main.person{name:&#34;小王子&#34;, city:&#34;北京&#34;, age:18}
```

当某些字段没有初始值的时候，该字段可以不写。此时，没有指定初始值的字段的值就是该字段类型的零值。

```go
p7 := &amp;person{
	city: &#34;北京&#34;,
}
fmt.Printf(&#34;p7=%#v\n&#34;, p7) //p7=&amp;main.person{name:&#34;&#34;, city:&#34;北京&#34;, age:0}
```

### 3.2 使用值的列表初始化

初始化结构体的时候可以简写，也就是初始化的时候不写键，直接写值：

```go
p8 := &amp;person{
	&#34;沙河娜扎&#34;,
	&#34;北京&#34;,
	28,
}
fmt.Printf(&#34;p8=%#v\n&#34;, p8) //p8=&amp;main.person{name:&#34;沙河娜扎&#34;, city:&#34;北京&#34;, age:28}
```

- 使用这种格式初始化时，需要注意：

1. 必须初始化结构体的所有字段。
2. 初始值的填充顺序必须与字段在结构体中的声明顺序一致。
3. 该方式不能和键值初始化方式混用。

## 4. 结构体内存布局

结构体占用一块连续的内存。

```go
type test struct {
	a int8
	b int8
	c int8
	d int8
}
n := test{
	1, 2, 3, 4,
}
fmt.Printf(&#34;n.a %p\n&#34;, &amp;n.a)
fmt.Printf(&#34;n.b %p\n&#34;, &amp;n.b)
fmt.Printf(&#34;n.c %p\n&#34;, &amp;n.c)
fmt.Printf(&#34;n.d %p\n&#34;, &amp;n.d)
```

输出：

```bash
n.a 0xc0000a0060
n.b 0xc0000a0061
n.c 0xc0000a0062
n.d 0xc0000a0063
```

## 5.构造函数

Go语言的结构体没有构造函数，我们可以自己实现。 例如，下方的代码就实现了一个`person`的构造函数。 因为`struct`是值类型，如果结构体比较复杂的话，值拷贝性能开销会比较大，所以该构造函数返回的是结构体指针类型。

```go
func newPerson(name, city string, age int8) *person {
	return &amp;person{
		name: name,
		city: city,
		age:  age,
	}
}
```

调用构造函数

```go
p9 := newPerson(&#34;张三&#34;, &#34;沙河&#34;, 90)
fmt.Printf(&#34;%#v\n&#34;, p9) //&amp;main.person{name:&#34;张三&#34;, city:&#34;沙河&#34;, age:90}
```

## 6.方法和接收者

**Go语言中的`方法（Method）`是一种作用于特定类型变量的函数。**这种特定类型变量叫做`接收者（Receiver）`。接收者的概念就类似于其他语言中的`this`或者 `self`。

方法的定义格式如下：

```go
func (接收者变量 接收者类型) 方法名(参数列表) (返回参数) {
    函数体
}
```

其中，

- 接收者变量：接收者中的参数变量名在命名时，官方建议使用接收者类型名称首字母的小写，而不是`self`、`this`之类的命名。例如，`Person`类型的接收者变量应该命名为 `p`，`Connector`类型的接收者变量应该命名为`c`等。
- 接收者类型：接收者类型和参数类似，可以是指针类型和非指针类型。
- 方法名、参数列表、返回参数：具体格式与函数定义相同。

举个例子：

```go
//Person 结构体
type Person struct {
	name string
	age  int8
}

//NewPerson 构造函数
func NewPerson(name string, age int8) *Person {
	return &amp;Person{
		name: name,
		age:  age,
	}
}

//Dream Person做梦的方法
func (p Person) Dream() {
	fmt.Printf(&#34;%s的梦想是学好Go语言！\n&#34;, p.name)
}

func main() {
	p1 := NewPerson(&#34;Evan&#34;, 27)
	p1.Dream()
}
```

方法与函数的区别是，函数不属于任何类型，方法属于特定的类型。

### 6.1 指针类型的接收者

指针类型的接收者由一个结构体的指针组成，由于指针的特性，调用方法时修改接收者指针的任意成员变量，在方法结束后，修改都是有效的。这种方式就十分接近于其他语言中面向对象中的`this`或者`self`。 例如我们为`Person`添加一个`SetAge`方法，来修改实例变量的年龄。

```go
// SetAge 设置p的年龄
// 使用指针接收者
func (p *Person) SetAge(newAge int8) {
	p.age = newAge
}
```

调用该方法：

```go
func main() {
	p1 := NewPerson(&#34;小王子&#34;, 25)
	fmt.Println(p1.age) // 25
	p1.SetAge(30)
	fmt.Println(p1.age) // 30
}
```

### 6.2 值类型的接收者

当方法作用于值类型接收者时，Go语言会在代码运行时将接收者的值复制一份。**在值类型接收者的方法中可以获取接收者的成员值，但修改操作只是针对副本，无法修改接收者变量本身。**

```go
// SetAge2 设置p的年龄
// 使用值接收者
func (p Person) SetAge2(newAge int8) {
	p.age = newAge
}

func main() {
	p1 := NewPerson(&#34;小王子&#34;, 25)
	p1.Dream()
	fmt.Println(p1.age) // 25
	p1.SetAge2(30) // (*p1).SetAge2(30)
	fmt.Println(p1.age) // 25
}
```

### 6.3 什么时候应该使用指针类型接收者

1. 需要修改接收者中的值
2. 接收者是拷贝代价比较大的大对象
3. 保证一致性，如果有某个方法使用了指针接收者，那么其他的方法也应该使用指针接收者。

### 6.4 任意类型添加方法

在Go语言中，接收者的类型可以是任何类型，不仅仅是结构体，任何类型都可以拥有方法。 举个例子，我们基于内置的`int`类型使用type关键字可以定义新的自定义类型，然后为我们的自定义类型添加方法。

```go
//MyInt 将int定义为自定义MyInt类型
type MyInt int

//SayHello 为MyInt添加一个SayHello的方法
func (m MyInt) SayHello() {
	fmt.Println(&#34;Hello, 我是一个int。&#34;)
}
func main() {
	var m1 MyInt
	m1.SayHello() //Hello, 我是一个int。
	m1 = 100
	fmt.Printf(&#34;%#v  %T\n&#34;, m1, m1) //100  main.MyInt
}
```

**注意事项：** 非本地类型不能定义方法，也就是说我们不能给别的包的类型定义方法。

## 7. 结构体的匿名字段

结构体允许其成员字段在声明时没有字段名而只有类型，这种没有名字的字段就称为匿名字段。

```go
//Person 结构体Person类型
type Person struct {
	string
	int
}

func main() {
	p1 := Person{
		&#34;小王子&#34;,
		18,
	}
	fmt.Printf(&#34;%#v\n&#34;, p1)        //main.Person{string:&#34;北京&#34;, int:18}
	fmt.Println(p1.string, p1.int) //北京 18
}
```

**注意：**这里匿名字段的说法并不代表没有字段名，而是默认会采用类型名作为字段名，结构体要求字段名称必须唯一，因此一个结构体中同种类型的匿名字段只能有一个。

## 8. 嵌套结构体

&gt;结构体嵌入接口值，结构体初始化后可以返回嵌入的接口值类型

一个结构体中可以嵌套包含另一个结构体或结构体指针，就像下面的示例代码那样。

```go
//Address 地址结构体
type Address struct {
	Province string
	City     string
}

//User 用户结构体
type User struct {
	Name    string
	Gender  string
	Address Address
}

func main() {
	user1 := User{
		Name:   &#34;小王子&#34;,
		Gender: &#34;男&#34;,
		Address: Address{
			Province: &#34;山东&#34;,
			City:     &#34;威海&#34;,
		},
	}
	fmt.Printf(&#34;user1=%#v\n&#34;, user1)//user1=main.User{Name:&#34;小王子&#34;, Gender:&#34;男&#34;, Address:main.Address{Province:&#34;山东&#34;, City:&#34;威海&#34;}}
}
```

### 8.1 嵌套匿名字段

上面user结构体中嵌套的`Address`结构体也可以采用匿名字段的方式，例如：

```go
//Address 地址结构体
type Address struct {
	Province string
	City     string
}

//User 用户结构体
type User struct {
	Name    string
	Gender  string
	Address //匿名字段
}

func main() {
	var user2 User
	user2.Name = &#34;小王子&#34;
	user2.Gender = &#34;男&#34;
	user2.Address.Province = &#34;山东&#34;    // 匿名字段默认使用类型名作为字段名
	user2.City = &#34;威海&#34;                // 匿名字段可以省略
	fmt.Printf(&#34;user2=%#v\n&#34;, user2) //user2=main.User{Name:&#34;小王子&#34;, Gender:&#34;男&#34;, Address:main.Address{Province:&#34;山东&#34;, City:&#34;威海&#34;}}
}
```

**当访问结构体成员时会先在结构体中查找该字段，找不到再去嵌套的匿名字段中查找。**

### 8.2 嵌套结构体的字段名冲突

嵌套结构体内部可能存在相同的字段名。在这种情况下为了避免歧义需要通过指定具体的内嵌结构体字段名。

```go
//Address 地址结构体
type Address struct {
	Province   string
	City       string
	CreateTime string
}

//Email 邮箱结构体
type Email struct {
	Account    string
	CreateTime string
}

//User 用户结构体
type User struct {
	Name   string
	Gender string
	Address
	Email
}

func main() {
	var user3 User
	user3.Name = &#34;沙河娜扎&#34;
	user3.Gender = &#34;男&#34;
	// user3.CreateTime = &#34;2019&#34; //ambiguous selector user3.CreateTime
	user3.Address.CreateTime = &#34;2000&#34; //指定Address结构体中的CreateTime
	user3.Email.CreateTime = &#34;2000&#34;   //指定Email结构体中的CreateTime
}
```

## 9. 结构体的“继承”

Go语言中使用结构体也可以实现其他编程语言中面向对象的继承。

```go
//Animal 动物
type Animal struct {
	name string
}

func (a *Animal) move() {
	fmt.Printf(&#34;%s会动！\n&#34;, a.name)
}

//Dog 狗
type Dog struct {
	Feet    int8
	*Animal //通过嵌套匿名结构体实现继承
}

func (d *Dog) wang() {
	fmt.Printf(&#34;%s会汪汪汪~\n&#34;, d.name)
}

func main() {
	d1 := &amp;Dog{
		Feet: 4,
		Animal: &amp;Animal{ //注意嵌套的是结构体指针
			name: &#34;乐乐&#34;,
		},
	}
	d1.wang() //乐乐会汪汪汪~
	d1.move() //乐乐会动！
}
```



## 10. 结构体与JSON序列化

JSON(JavaScript Object Notation) 是一种轻量级的数据交换格式。易于人阅读和编写。同时也易于机器解析和生成。JSON键值对是用来保存JS对象的一种方式，键/值对组合中的键名写在前面并用双引号`&#34;&#34;`包裹，使用冒号`:`分隔，然后紧接着值；多个键值之间使用英文`,`分隔。

```go
//Student 学生
type Student struct {
	ID     int
	Gender string
	Name   string
}

//Class 班级
type Class struct {
	Title    string
	Students []*Student
}

func main() {
	c := &amp;Class{
		Title:    &#34;101&#34;,
		Students: make([]*Student, 0, 200),
	}
	for i := 0; i &lt; 10; i&#43;&#43; {
		stu := &amp;Student{
			Name:   fmt.Sprintf(&#34;stu%02d&#34;, i),
			Gender: &#34;男&#34;,
			ID:     i,
		}
		c.Students = append(c.Students, stu)
	}
	//JSON序列化：结构体--&gt;JSON格式的字符串
	data, err := json.Marshal(c)
	if err != nil {
		fmt.Println(&#34;json marshal failed&#34;)
		return
	}
	fmt.Printf(&#34;json:%s\n&#34;, data)
	//JSON反序列化：JSON格式的字符串--&gt;结构体
	str := `{&#34;Title&#34;:&#34;101&#34;,&#34;Students&#34;:[{&#34;ID&#34;:0,&#34;Gender&#34;:&#34;男&#34;,&#34;Name&#34;:&#34;stu00&#34;},{&#34;ID&#34;:1,&#34;Gender&#34;:&#34;男&#34;,&#34;Name&#34;:&#34;stu01&#34;},{&#34;ID&#34;:2,&#34;Gender&#34;:&#34;男&#34;,&#34;Name&#34;:&#34;stu02&#34;},{&#34;ID&#34;:3,&#34;Gender&#34;:&#34;男&#34;,&#34;Name&#34;:&#34;stu03&#34;},{&#34;ID&#34;:4,&#34;Gender&#34;:&#34;男&#34;,&#34;Name&#34;:&#34;stu04&#34;},{&#34;ID&#34;:5,&#34;Gender&#34;:&#34;男&#34;,&#34;Name&#34;:&#34;stu05&#34;},{&#34;ID&#34;:6,&#34;Gender&#34;:&#34;男&#34;,&#34;Name&#34;:&#34;stu06&#34;},{&#34;ID&#34;:7,&#34;Gender&#34;:&#34;男&#34;,&#34;Name&#34;:&#34;stu07&#34;},{&#34;ID&#34;:8,&#34;Gender&#34;:&#34;男&#34;,&#34;Name&#34;:&#34;stu08&#34;},{&#34;ID&#34;:9,&#34;Gender&#34;:&#34;男&#34;,&#34;Name&#34;:&#34;stu09&#34;}]}`
	c1 := &amp;Class{}
	err = json.Unmarshal([]byte(str), c1)
	if err != nil {
		fmt.Println(&#34;json unmarshal failed!&#34;)
		return
	}
	fmt.Printf(&#34;%#v\n&#34;, c1)
}
```

## 11. 结构体标签（Tag）

`Tag`是结构体的元信息，可以在运行的时候通过反射的机制读取出来。 `Tag`在结构体字段的后方定义，由一对**反引号**包裹起来，具体的格式如下：

```bash
`key1:&#34;value1&#34; key2:&#34;value2&#34;`
```

结构体tag由一个或多个键值对组成。键与值使用冒号分隔，值用双引号括起来。同一个结构体字段可以设置多个键值对tag，不同的键值对之间使用空格分隔。

**注意事项：** 为结构体编写`Tag`时，必须严格遵守键值对的规则。结构体标签的解析代码的容错能力很差，一旦格式写错，编译和运行时都不会提示任何错误，通过反射也无法正确取值。例如不要在key和value之间添加空格。

例如我们为`Student`结构体的每个字段定义json序列化时使用的Tag：

```go
//Student 学生
type Student struct {
	ID     int    `json:&#34;id&#34;` //通过指定tag实现json序列化该字段时的key
	Gender string //json序列化是默认使用字段名作为key
	name   string //私有不能被json包访问
}

func main() {
	s1 := Student{
		ID:     1,
		Gender: &#34;男&#34;,
		name:   &#34;沙河娜扎&#34;,
	}
	data, err := json.Marshal(s1)
	if err != nil {
		fmt.Println(&#34;json marshal failed!&#34;)
		return
	}
	fmt.Printf(&#34;json str:%s\n&#34;, data) //json str:{&#34;id&#34;:1,&#34;Gender&#34;:&#34;男&#34;}
}
```

## 12. 结构体和方法补充知识点

因为slice和map这两种数据类型都包含了指向底层数据的指针，因此我们在需要复制它们时要特别注意。我们来看下面的例子：

```go
type Person struct {
	name   string
	age    int8
	dreams []string
}

func (p *Person) SetDreams(dreams []string) {
	p.dreams = dreams
}

func main() {
	p1 := Person{name: &#34;小王子&#34;, age: 18}
	data := []string{&#34;吃饭&#34;, &#34;睡觉&#34;, &#34;打豆豆&#34;}
	p1.SetDreams(data)

	// 你真的想要修改 p1.dreams 吗？
	data[1] = &#34;不睡觉&#34;
	fmt.Println(p1.dreams)  // ?
}
```

正确的做法是在方法中使用传入的slice的拷贝进行结构体赋值。

```go
func (p *Person) SetDreams(dreams []string) {
	p.dreams = make([]string, len(dreams))
	copy(p.dreams, dreams)
}
```

同样的问题也存在于返回值slice和map的情况，在实际编码过程中一定要注意这个问题。



---

> 作者: [Fred](https://github.com/ipfred)  
> URL: https://ipfred.github.io/lang/go/go_base/20250515175056/  

