# 2-1. 变量

## 	位-字节-字符-编码

```go
1、位：数据存储的最小单位。每个二进制数字0或者1就是1个位；

2、字节：8个位构成一个字节；即：1 byte (字节)= 8 bit(位)；
     1 KB = 1024 B(字节)；
     1 MB = 1024 KB;   (2^10 B)
     1 GB = 1024 MB;  (2^20 B)
     1 TB = 1024 GB;   (2^30 B)

3、字符：a、A、中、&#43;、*、の......均表示一个字符 ；
	  unioncode 一个汉字 4 个字节表示
    一般 utf-8 编码下，一个汉字 字符 占用 3 个 字节；
    一般 gbk 编码下，一个汉字  字符  占用 2 个 字节；

```

## assic码

| ASCII值 | 控制字符 | ASCII值 | 控制字符 | ASCII值 | 控制字符 | ASCII值 | 控制字符 |
| :------ | :------- | :------ | :------- | :------ | :------- | :------ | :------- |
| 0       | NUT      | 32      | (space)  | 64      | @        | 96      | 、       |
| 1       | SOH      | 33      | !        | **65**  | A        | **97**  | a        |
| 2       | STX      | 34      | &#34;        | 66      | B        | 98      | b        |
| 3       | ETX      | 35      | #        | 67      | C        | 99      | c        |
| 4       | EOT      | 36      | $        | 68      | D        | 100     | d        |
| 5       | ENQ      | 37      | %        | 69      | E        | 101     | e        |
| 6       | ACK      | 38      | &amp;        | 70      | F        | 102     | f        |
| 7       | BEL      | 39      | ,        | 71      | G        | 103     | g        |
| 8       | BS       | 40      | (        | 72      | H        | 104     | h        |
| 9       | HT       | 41      | )        | 73      | I        | 105     | i        |
| 10      | LF       | 42      | *        | 74      | J        | 106     | j        |
| 11      | VT       | 43      | &#43;        | 75      | K        | 107     | k        |
| 12      | FF       | 44      | ,        | 76      | L        | 108     | l        |
| 13      | CR       | 45      | -        | 77      | M        | 109     | m        |
| 14      | SO       | 46      | .        | 78      | N        | 110     | n        |
| 15      | SI       | 47      | /        | 79      | O        | 111     | o        |
| 16      | DLE      | **48**  | 0        | 80      | P        | 112     | p        |
| 17      | DCI      | 49      | 1        | 81      | Q        | 113     | q        |
| 18      | DC2      | 50      | 2        | 82      | R        | 114     | r        |
| 19      | DC3      | 51      | 3        | 83      | S        | 115     | s        |
| 20      | DC4      | 52      | 4        | 84      | T        | 116     | t        |
| 21      | NAK      | 53      | 5        | 85      | U        | 117     | u        |
| 22      | SYN      | 54      | 6        | 86      | V        | 118     | v        |
| 23      | TB       | 55      | 7        | 87      | W        | 119     | w        |
| 24      | CAN      | 56      | 8        | 88      | X        | 120     | x        |
| 25      | EM       | 57      | 9        | 89      | Y        | 121     | y        |
| 26      | SUB      | 58      | :        | 90      | Z        | 122     | z        |
| 27      | ESC      | 59      | ;        | 91      | [        | 123     | {        |
| 28      | FS       | 60      | &lt;        | 92      | /        | 124     | \|       |
| 29      | GS       | 61      | =        | 93      | ]        | 125     | }        |
| 30      | RS       | 62      | &gt;        | 94      | ^        | 126     | `        |
| 31      | US       | 63      | ?        | 95      | _        | 127     | DEL      |

## 命名

- go语言中的函数名、变量名、常量名、类型名、语句标号和包名等所有的命名，都遵循一个简单的命名规则

  - 一个名字必须以一个字母或下划线开头，后面可以跟任意数量的字母、数字或下划线

- go语言中有25个关键字，不能用于自定义名字

  ```go 
  break        default          func           interface         select
  case         defer            go             map               struct
  chan         else             goto           package           switch
  const        fallthrough      if             range             type
  continue     for              import         return            var
  ```

- 还有30多个预定义的名字，用于内建的常量、类型和函数

  ```go
  //内建常量:
      true false iota nil
  //内建类型:
      int int8 int16 int32 int64
      uint uint8 uint16 uint32 uint64 uintptr
      float32 float64 complex128 complex64
      bool byte rune string error
  //内建函数:
      make len cap new append copy close delete
      complex real imag
      panic recover
  ```

## 变量

```go
package main

import &#34;fmt&#34;

func main()  {
	var a int
	var b , c string
	var (/* */
		d int
		e string
		f bool
	)
	fmt.Println(a,b,c,d,e,f)  // 0   0  false
}

```

1. **第一种，指定变量类型，如果没有初始化，则变量默认为零值**。

   *零值就是变量没有做初始化时系统默认设置的值*。

- 数值类型（包括complex64/128）为 **0**

- 布尔类型为 **false**

- 字符串为 **&#34;&#34;**（空字符串）

- 以下几种类型为 **nil**：

  ```go
  var a *int
  var a []int
  var a map[string] int
  var a chan int
  var a func(string) int
  var a error // error 是接口
  ```

2. **第二种，根据值自行判定变量类型。**

   ```go
   package main
   import &#34;fmt&#34;
   func main() {
     var d = true
     fmt.Println(d)
   }
   ```

3. **第三种，省略 var, 注意 := 左侧如果没有声明新的变量，就产生编译错误，格式：**

   ```go
   v_name := value
   //可以将 var f string = &#34;Runoob&#34; 简写为 f := &#34;Runoob&#34;：
   ```

   

## 常量

常量的定义格式：

```go
const identifier [type] = value
```

- **在定义常量组时，如果不提供初始值，则表示将使用上行的表达式。**
- 常量是一个简单值的标识符，在程序运行时，不会被修改的量。

- 常量中的数据类型只可以是布尔型、数字型（整数型、浮点型和复数）和字符串型。

### iota

- iota，特殊**常量**，可以认为是一个可以被编译器修改的常量, 变量中不可使用。

- iota 在 const关键字出现时将被重置为 0(const 内部的第一行之前)，const 中每新增一行常量声明将使 iota 计数一次(iota 可理解为 const 语句块中的行索引)。

iota 可以被用作枚举值：

```go
const (
	a = iota  //0
	b = iota  //1
	c = iota  //2
)
```

第一个 iota 等于 0，每当 iota 在新的一行被使用时，它的值都会自动加 1；所以 a=0, b=1, c=2 可以简写为如下形式：

```go
const (
    a = iota
    b
    c
)
```

- **iota 只是在同一个 const 常量组内递增，每当有新的 const 关键字时，iota 计数会重新开始。**

### iota 用法

```go
package main
import &#34;fmt&#34;

func main() {
  const (
      a = iota  *//0*
      b      *//1*
      c      *//2*
      d = &#34;ha&#34;  *//独立值，iota &#43;= 1*
      e      *//&#34;ha&#34;  iota &#43;= 1*
      f = 100   *//iota &#43;=1*
      g      *//100  iota &#43;=1*
      h = iota  *//7,恢复计数*
      i      *//8*
  )
  fmt.Println(a,b,c,d,e,f,g,h,i)
}
// 0 1 2 ha ha 100 100 7 8
```

- **iota 只是在同一个 const 常量组内递增，每当有新的 const 关键字时，iota 计数会重新开始。**

### Iota 和左右运算符

```go
package main

import &#34;fmt&#34;
const (
  i=1&lt;&lt;iota
  j=3&lt;&lt;iota
  k
  l
)

func main() {
  fmt.Println(&#34;i=&#34;,i)
  fmt.Println(&#34;j=&#34;,j)
  fmt.Println(&#34;k=&#34;,k)
  fmt.Println(&#34;l=&#34;,l)
}

/*
i= 1
j= 6
k= 12
l= 24 */
```

iota 表示从 0 开始自动加 1，所以 **i=1&lt;&lt;0**, **j=3&lt;&lt;1**（**&lt;&lt;** 表示左移的意思），即：i=1, j=6，这没问题，关键在 k 和 l，从输出结果看 **k=3&lt;&lt;2**，**l=3&lt;&lt;3**。

简单表述:

- **i=1**：左移 0 位,不变仍为 1;

- **j=3**：左移 1 位,变为二进制 110, 即 6;
- **k=3**：左移 2 位,变为二进制 1100, 即 12;
- **l=3**：左移 3 位,变为二进制 11000,即 24。

注：**&lt;&lt;n==\*(2^n)**。

```go
// 左移运算符 &lt;&lt; 是双目运算符。左移 n 位就是乘以 2 的 n 次方。 其功能把 &lt;&lt; 左边的运算数的各二进位全部左移若干位，由 &lt;&lt; 右边的数指定移动的位数，高位丢弃，低位补 0。

//右移运算符 &gt;&gt; 是双目运算符。右移 n 位就是除以 2 的 n 次方。 其功能是把 &gt;&gt; 左边的运算数的各二进位全部右移若干位， &gt;&gt; 右边的数指定移动的位数。
```



---

> 作者: [Fred](https://github.com/ipfred)  
> URL: https://ipfred.github.io/lang/go/go_base/20250515174450/  

