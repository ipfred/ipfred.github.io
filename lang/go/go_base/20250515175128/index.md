# 7. 反射



## 反射定义

### Python 反射

- python一切皆对象,所以想要通过字符串的形式操作内部成员都可以通过反射去完成操作.

- py文件 包 类 对象...(导入包操作类调用方法)

- 反射:根据字符串的形式去某个对象操作对象的成员.

  - getattr(对象名,&#34;方法名&#34;)    

    - **根据字符串的形式去某个对象中获取对象的成员**.

    - attribute属性

    ```python
    class Foo(object):
        def __init__(self,name):
          self.name = name
        def login(self):
          pass
    obj = Foo(&#39;alex&#39;)
    
    # 获取变量
    v1 = getattr(obj,&#39;name&#39;)
    # 获取方法
    method_name = getattr(obj,&#39;login&#39;)
    method_name()
    ```

  - setattr(对象名称,&#34;变量&#34;,值 )   

    - **根据字符串的形式去某个对象中设置成员**.

    ```python
    class Foo:
      pass
    obj = Foo()
      obj.k1 = 999
    setattr(obj,&#39;k1&#39;,123) # obj.k1 = 123
    
      print(obj.k1)
    ```

  - hasattr(对象名称,&#34;方法名&#34;)

    - **根据字符串的形式去某个对象中判断是否含有某成员**.返回布尔类型

    ```python
      class Foo:
          pass
      
      obj = Foo()
      obj.k1 = 999
      hasattr(obj,&#39;k1&#39;)
      print(obj.k1)
    ```

    

  - delattr(对象,&#34;方法名&#34;)

  - **根据字符串的形式去某个对象中删除某成员**.

  ```python
  class Foo:
      pass
  
  obj = Foo()
  obj.k1 = 999
  delattr(obj,&#39;k1&#39;)
  print(obj.k1)
  ```



- **importlib** 用字符串的形式导入模块

  ```python
  模块 = importlib.import_module(&#39;utils.redis&#39;)
  ```

  - 示例:

  ```python
  import importlib
  
  #用字符串的模式导入模块
  redis = importlib.import_module(&#34;utils.redis&#34;)
  #用字符串的形式去对象(模块)找到他的成员
  getattr(redis,&#34;func&#34;)()
  ```

  ```python
  self.MIDDLEWARE_CLASSES = [
              &#39;utils.session.SessionMiddleware&#39;,
              &#39;utils.auth.AuthMiddleware&#39;,
              &#39;utils.csrf.CrsfMiddleware&#39;,
          ]
  for mes in self.MIDDLEWARE_CLASSES:
      module_path,class_name=mes.rsplit(&#39;.&#39;,maxsplit=1)       #切割路径和类名
      module_object = importlib.import_module(module_path)    #插入模块-字符串操作
      cla=getattr(module_object,class_name)        #根据模块对象找到类名(字符串操作-反射)
      obj = cla()      #实例化对象
  	obj.process()      #运行内部函数process
  ```

  

### Golang反射

- 反射是指程序在运行期间，动态地更新、获取变量的值，包括获取字段类型、名称、调用类变量对应的方法等。

- **在运行时更新变量和检查他们的值并调用他们的方法和他们支持的内在操作, 但是在编译时b并不知道这些变量的具体类型; 这种机制叫做反射**

- python的反射和Golang反射的区别
  - python反射:
    - 确认对象的类
    - 确认类中所有的成员变量和方法
    - 动态调用任意一个对象的方法
  - golang反射
    - **不支持对字符串解析!**
    - **获取对象的值和类型,获取结构体成员的类型, 调用结构体方法**
    - **只能作用于已经存在的对象上**
- 变量包括两部分:
  - 类型信息, 这部分是原信息, 是预定义好的
  - 值类型, 这部分在程序运行过程中是动态改变的

- 反射与空接口
  - 在运行时动态的获取一个变量的类型信息和值信息就是反射
- reflect 
  - **reflect.TypeOf()**  获取变量类型信息, 返回一个type接口
  - **reflect.ValueOf()**  获取变量值信息
  - **reflect.Kind()**   获取变量类型

```go
package main
import (
	&#34;fmt&#34;
	&#34;reflect&#34;
)
func main() {
	var x float64 = 3.4
	t := reflect.TypeOf(x)  // t是一个type对象
	fmt.Println(reflect.TypeOf(t))  //*reflect.rtype
	fmt.Println(&#34;type:&#34;, t.Kind())  // float64
}
```

## 反射的基本用法

- reflect 
  - **t := reflect.TypeOf()**  获取变量类型信息, 返回一个type接口
  - **t := reflect.ValueOf()**  获取变量值信息
  - **t.Kind()**   获取变量类型
- **t.Elem()**   反射获取指针变量所指向的元素类型
  
  - **t.Name()**   获取变量类型名称
  
- reflect.Kind() 获取到的变量种类:

  ```go
  const  (
  	Invalid Kind = iota   // 非法类型
  	Bool
  	Int		//有符号整形
  	Int8   // 有符号8位整形
  	Int16
  	Int32
  	Int64
  	Uint   //无符号整形
  	Uint8	// 无符号8位整形
  	Uint16
  	Uint32
  	Uint64
  	Uintptr   //指针
  	Float32
  	Float64
  	Complex64  //64位复数
  	Complex128  //128位复数
  	Array
  	Chan
  	Func
  	Interface
  	Map
  	Ptr   //指针
  	Slice
  	String
  	Struct
  	UnsafePointer   //底层指针
  )
  
  ```

### 获取类型信息

- **reflect.TypeOf()**  获取变量类型信息, 返回一个type接口

```go

package main

import (
	&#34;fmt&#34;
	&#34;reflect&#34;
)

type Number int
type Person struct {
}

func checkType(t reflect.Type) {
	if t.Kind() == reflect.Ptr {
		fmt.Printf(&#34;变量的类型名称:%v; 指向的变量为:%s\n&#34;, t.Kind(), t.Elem())

	}
	fmt.Printf(&#34;变量的类型名称=&gt; %v; ;类型种类=&gt;:%s\n&#34;, t.Name(), t.Kind())
}

func main() {
	var number Number = 1
	typeOfNumber := reflect.TypeOf(number)
	fmt.Println(&#34;type of number:&#34;)
	checkType(typeOfNumber)

	var person Person
	typeOfPerson := reflect.TypeOf(person)
	fmt.Println(&#34;type of person&#34;)
	checkType(typeOfPerson)

	typeOfPersonPtr := reflect.TypeOf(&amp;person)
	fmt.Println(&#34;type of &amp;person&#34;)
	checkType(typeOfPersonPtr)

}
/**
type of number:
变量的类型名称=&gt; Number; ;类型种类=&gt;:int
type of person
变量的类型名称=&gt; Person; ;类型种类=&gt;:struct
type of &amp;person
变量的类型名称:ptr; 指向的变量为:main.Person
变量的类型名称=&gt; ; ;类型种类=&gt;:ptr
*/
```



### 获取类型的值

- **t :=reflect.ValueOf()**  获取变量值信息
- 获取变量的值和反射调用函数都需要使用**reflect.ValueOf()**
- **t.Kind()** == **reflect.TypeOf()**
- **t.Set**Int( intnumber )  改变类型的值,因为是值传递,必须传入指针才能改变值

```go
package main 

import (
	&#34;fmt&#34;
	&#34;reflect&#34;
)

func getVlaue(arg interface{}) {
	argReflectValue := reflect.ValueOf(arg)
	fmt.Printf(&#34;arg value : %v\n&#34;, argReflectValue)
	fmt.Printf(&#34;arg type :%v\n&#34;, argReflectValue.Kind())

	switch argReflectValue.Kind() {
	case reflect.Int:
		fmt.Println(&#34;arg type is int&#34;)
	case reflect.Float32:
		// fmt.Println(&#34;arg type is float32&#34;)
		res := argReflectValue.Elem()
		res.SetFloat(2.6)
		fmt.Println(res)
	case reflect.Ptr:
		fmt.Println(&#34;arg type is pointer&#34;)
		res := argReflectValue.Elem()
		res.SetFloat(2.6)     // 改变传入参数的值, 必须传入指针类型才可以改变
		fmt.Println(res)

	default:
		fmt.Println(&#34;not fond&#34;)

	}
}

func main() {
	var f float32 = 1.3
	getVlaue(&amp;f)

}


```

### 反射调用函数

- 反射调用函数: 使用reflect.ValueOf() 方法传入想要反射的函数名, 获取到reflect.Value对象, 再通过该对象啊的call方法调用该函数;

- Call 方法需要提前声明; 

  ```go
  func (v Value) Call (in []Value) []Value
  ```

  - call 方法输入的参数in调用v持有的函数.  
  - 例如, 如果len(in) == 3, v.Call(in) 代表调用v(int[0],int[1],int[2]), 其中value值表示其持有值.  如果v的kind不是func将会panic..  它返回函数所有输出结果的Value封装的切片。和Go代码一样,每一个输入实参的持有值都必须可以直接赋值给函数对应输入参数的类型。如果v持有值是可变参数函数,Call方法会自行创建一个代表可变参数的切片,将对应可变参数的值都拷贝到里面。

  ```go
  
  package main
  
  import (
  	&#34;fmt&#34;
  	&#34;reflect&#34;
  )
  
  func Equal(a, b int) bool {
  	if a == b {
  		return true
  	}
  	return false
  }
  
  func main() {
  	// reflect.Value 类型
  	valueOfFunc := reflect.ValueOf(Equal)
  	// 构造函数参数
  	args := []reflect.Value{reflect.ValueOf(1), reflect.ValueOf(2)}
  
  	// 同各国反射调用函数计算
  	result := valueOfFunc.Call(args)
  
  	fmt.Println(&#34;函数运行结果: &#34;, result[0].Bool())  // 函数运行结果:  false
  }
  
  ```

## 结构体反射

- 反射可以获取结构体成员的类型、结构体成员的值、以及调用结构体的方法

### 1. 获取结构体成员类型

- t := reflect.TypeOf(&#34;结构体&#34;)   // t是reflect.Type类型

  - t.NumField()  获取结构体成员的数量

  - t.Field(index) 可以**根据索引**返回结构体字段详细信息; 具体信息如下

    ```go
    type StructField struct{
    	Name string  	//字段名
    	PkgPath string  //字段路径
    	Type Type		//字段反射类型对象
    	Tag StruntTag	//字段结构体标签
    	Offset uinptr	//字段在结构体中的偏移
    	Index  []int 	//字段的索引值
    	Anonymous bool 	//是否为匿名字段
    }
    ```

  - t.FieldByName(name string)    通过**字段名**来获取字段信息
  - t.FieldByIndex(index []int)   通过**下标**来获取字段信息

  ```go
  // type StructField struct{
  // 	Name string  	//字段名
  // 	PkgPath string  //字段路径
  // 	Type Type		//字段反射类型对象
  // 	Tag StruntTag	//字段结构体标签
  // 	Offset uinptr	//字段在结构体中的偏移
  // 	Index  []int 	//字段的索引值
  // 	Anonymous bool 	//是否为匿名字段
  // }
  
  package main
  
  import (
  	&#34;fmt&#34;
  	&#34;reflect&#34;
  )
  
  type Person struct {
  	Name string
  	Age  int `json:&#34;age&#34;`
  	string
  }
  
  func main() {
  	person := Person{&#34;Evan&#34;, 18, &#34;备注&#34;}
  	typeOfPerson := reflect.TypeOf(person)
  	// 遍历结构体成员, 获取字段信息
  	for i := 0; i &lt; typeOfPerson.NumField(); i&#43;&#43; {
  		field := typeOfPerson.Field(i)
  		name := field.Name
  		tag := field.Tag
  		anonymous := field.Anonymous
  		fmt.Printf(&#34;字段名:%v 字段标签:%v 是否为匿名字段:%v \n&#34;, name, tag, anonymous)
  	}
  	// 通过字段名获取字段信息
  	if field, ok := typeOfPerson.FieldByName(&#34;Age&#34;); ok {
  		fmt.Println(&#34;通过字段名&#34;)
  		fmt.Printf(&#34;字段名:%v  字段标签中json为: %v \n&#34;, field.Name, field.Tag.Get(&#34;json&#34;))
  	}
  
  	// 通过下标获取字段信息
  	field := typeOfPerson.FieldByIndex([]int{1})
  	fmt.Printf(&#34;字段名:%v 字段标签: %v \n&#34;, field.Name, field.Tag)
  
  }
  /**
  字段名:Name 字段标签: 是否为匿名字段:false
  字段名:Age 字段标签:json:&#34;age&#34; 是否为匿名字段:false
  字段名:string 字段标签: 是否为匿名字段:true
  通过字段名
  字段名:Age  字段标签中json为: age
  字段名:Age 字段标签: json:&#34;age&#34;
  */
  ```

  

### 2. 获取结构体成员字段的值

- t := reflect.Valueof(&#34;结构体&#34;) 

  - t.NumField()  获取结构体成员的数量
  - t.Feild(index)  根据 索引返回的对应结构体字段的reflect.Value 反射类型

  ```go
  // type StructField struct{
  // 	Name string  	//字段名
  // 	PkgPath string  //字段路径
  // 	Type Type		//字段反射类型对象
  // 	Tag StruntTag	//字段结构体标签
  // 	Offset uinptr	//字段在结构体中的偏移
  // 	Index  []int 	//字段的索引值
  // 	Anonymous bool 	//是否为匿名字段
  // }
  
  package main
  
  import (
  	&#34;fmt&#34;
  	&#34;reflect&#34;
  )
  
  type Person struct {
  	Name string
  	Age  int `json:&#34;age&#34;`
  	string
  }
  
  func main() {
  	person := Person{&#34;Evan&#34;, 18, &#34;备注&#34;}
  	valueOfPerson := reflect.ValueOf(person)
  
  	// 结构体字段数量
  	fieldNum := valueOfPerson.NumField()
  	fmt.Printf(&#34;结构体字段数量: %d \n&#34;, fieldNum) //结构体字段数量: 3
  
  	// 通过下标获取字段值
  	field1 := valueOfPerson.Field(1)
  	fmt.Printf(&#34;字段值:%v \n&#34;, field1.Int())    //字段值:18
  	field2 := valueOfPerson.Field(0)
  	fmt.Printf(&#34;字段值:%v \n&#34;, field2.String())  //字段值:Evan
  
  	// 通过字段名获取字段信息
  	field  := valueOfPerson.FieldByName(&#34;Age&#34;)
  	fmt.Printf(&#34;字段值: %v \n&#34;, field.Interface())
  
  	//
  	// 通过下标索引获取字段信息
  	field = valueOfPerson.FieldByIndex([]int{1})
  	fmt.Printf(&#34;字段值: %v \n&#34;, field.Interface())
  
  }
  
  ```

### 3. 反射执行结构体方法

- 结构体方法需要使用reflect.ValueOf() 获取reflect.Value对象, 然后调用该对象的MethodByName(name) 函数, 找到对应要反射调用的方法, 再通过Call函数进行反射调用

  ```go
  package main
  
  import (
  	&#34;fmt&#34;
  	&#34;reflect&#34;
  )
  
  type Person struct {
  	Name string
  	Age  int `json:&#34;age&#34;`
  	string
  }
  
  func (p Person)GetName(a int)  {
  	fmt.Println(p.Name)  // Evan
  
  	fmt.Println(a)  // 1
  }
  
  func main() {
  	valueOfPerson := reflect.ValueOf(Person{&#34;Evan&#34;,18,&#34;备注&#34;})
  	callObj  := valueOfPerson.MethodByName(&#34;GetName&#34;)
  	callObj.Call([]reflect.Value{reflect.ValueOf(1)})
  }
  
  ```

### 4. 获取结构体tag的值

- ```go
  package main
  
  import (
  	&#34;fmt&#34;
  	&#34;reflect&#34; // 这里引入reflect模块
  )
  
  type User struct {
  	Name   string `json:&#34;user_name&#34;`
  	Passwd string `json:&#34;user_password&#34;`
  }
  
  func main() {
  	user := &amp;User{&#34;chronos&#34;, &#34;pass&#34;}
  	s := reflect.TypeOf(user).Elem() //通过反射获取type定义
  	for i := 0; i &lt; s.NumField(); i&#43;&#43; {
  		fmt.Println(s.Field(i).Tag.Get(&#34;json&#34;)) //将tag输出出来
  	}
  }
  
  ```

## 反射三定律

### 1. 接口到反射类型的转换

- 反射类型: `relect.Type` 和 `relect.Value`

- 反射可以将接口类型变量转换为反射类型变量

- `reflect.TypeOf()` 函数将换入的interface{}类型的变量进行解析后返回`refluect.Type`类型

- `reflect.ValueOf()` 函数将换入的interface{}类型的变量进行解析后返回`reflect.Value`类型

  ```go
  package main
  
  import (
  	&#34;fmt&#34;
  	&#34;reflect&#34;
  )
  
  type Person struct {
  	Name string
  	Age  int `json:&#34;age&#34;`
  	string
  }
  
  func main() {
  	person := Person{}
  	reflectType := reflect.TypeOf(person)
  	reflectValue := reflect.ValueOf(person)
  	fmt.Printf(&#34;person type:%T\n&#34;,person)	//person type:main.Person
  	fmt.Printf(&#34;reflectType type:%T\n&#34;,reflectType)	//reflectType type:*reflect.rtype
  	fmt.Printf(&#34;reflectValue type:%T\n&#34;,reflectValue)	//reflectValue type:reflect.Value
  }
  ```

  

### 2. 反射到接口类型的转换

- 反射可以将反射类型转换为接口类型

- `reflect.Value` 对象的 `interface() `方法, 可以将反射类型变量转换为接口变量

  ```go
  package main
  
  import (
  	&#34;fmt&#34;
  	&#34;reflect&#34;
  )
  
  func main() {
  
  	var num = 1
  	valueOfNum := reflect.ValueOf(num)
  	fmt.Println(valueOfNum.Interface())  // 1
  	fmt.Printf(&#34;%T&#34;,valueOfNum.Interface())  // int
  
  }
  ```

  

### 3. 修改反射类型对象

- 想要使用反射修改变量的值, 其值必须是可写的( CanSet ).这个值必须满足两个条件:

  1. 变量可以被寻址(CanAddr)
  2. 变量时可以导出的(结构体字段名首字母大写)

  ```go
  package main
  
  import (
  	&#34;fmt&#34;
  	&#34;reflect&#34;
  )
  
  type Person struct {
  	Name string
  	age  int `json:&#34;age&#34;`
  	string
  }
  
  func main() {
  	person := Person{&#34;Evan&#34;,28,&#34;备注&#34;}
  	fmt.Printf(&#34;修改前: %v&#34;,person)
  	valueOfPerson := reflect.ValueOf(&amp;person)
  	res := valueOfPerson.Elem()
  	res.Field(0).SetString(&#34;Jerry&#34;)
  	res.Field(1).SetInt(26)
  	//res.Field(2).SetString(&#34;啊啊&#34;)  // panic
  	fmt.Printf(&#34;修改后: %v&#34;,person)
  }
  
  ```

  

## 反射的性能

- Golang反射性能极差 , 如果需要考虑性能的地方不推荐使用

---

> 作者: [Fred](https://github.com/ipfred)  
> URL: https://ipfred.github.io/lang/go/go_base/20250515175128/  

