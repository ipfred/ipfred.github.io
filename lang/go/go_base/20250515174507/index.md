# 2-2. 数据类型

## 数据类型

Go 语言按类别有以下几种数据类型：

| 序号 | 类型和描述                                                   |
| ---- | ------------------------------------------------------------ |
| 1    | **布尔型**布尔型的值只可以是常量 true 或者 false。一个简单的例子：var b bool = true。 |
| 2    | **数字类型**整型 int 和浮点型 float32、float64，Go 语言支持整型和浮点型数字，并且支持复数，其中位的运算采用补码。 |
| 3    | **字符串类型:**字符串就是一串固定长度的字符连接起来的字符序列。Go 的字符串是由单个字节连接起来的。Go 语言的字符串的字节使用 UTF-8 编码标识 Unicode 文本。 |
| 4    | **派生类型:**  指针类型（Pointer） /数组类型   /结构化类型(struct)  / Channel 类型  / 函数类型   /切片类型   /接口类型（interface） /Map 类型 |

- 字符串格式化

  ![img](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/wpsCUQ8za.jpg)
  
  - `%v`是万能的
  - `%T`查看数据类型,类似`reflect.typeOf()`
  
  - ```
    普通占位符
    占位符 说明 举例 输出
    %v 相应值的默认格式。 Printf(&#34;%v&#34;, people) {zhangsan}，
    %&#43;v 打印结构体时，会添加字段名 Printf(&#34;%&#43;v&#34;, people) {Name:zhangsan}
    %#v 相应值的Go语法表示 Printf(&#34;#v&#34;, people) main.Human{Name:&#34;zhangsan&#34;}
    %T 相应值的类型的Go语法表示 Printf(&#34;%T&#34;, people) main.Human
    %% 字面上的百分号，并非值的占位符 Printf(&#34;%%&#34;) %
    布尔占位符
    占位符 说明 举例 输出
    %t true 或 false。 Printf(&#34;%t&#34;, true) true
    整数占位符
    占位符 说明 举例 输出
    %b 二进制表示 Printf(&#34;%b&#34;, 5) 101
    %c 相应Unicode码点所表示的字符 Printf(&#34;%c&#34;, 0x4E2D) 中
    %d 十进制表示 Printf(&#34;%d&#34;, 0x12) 18
    %o 八进制表示 Printf(&#34;%d&#34;, 10) 12
    %q 单引号围绕的字符字面值，由Go语法安全地转义 Printf(&#34;%q&#34;, 0x4E2D) &#39;中&#39;
    %x 十六进制表示，字母形式为小写 a-f Printf(&#34;%x&#34;, 13) d
    %X 十六进制表示，字母形式为大写 A-F Printf(&#34;%x&#34;, 13) D
    %U Unicode格式：U&#43;1234，等同于 &#34;U&#43;%04X&#34; Printf(&#34;%U&#34;, 0x4E2D) U&#43;4E2D
    浮点数和复数的组成部分（实部和虚部）
    占位符 说明 举例 输出
    %b 无小数部分的，指数为二的幂的科学计数法，
    与 strconv.FormatFloat 的 &#39;b&#39; 转换格式一致。例如 -123456p-78
    %e 科学计数法，例如 -1234.456e&#43;78 Printf(&#34;%e&#34;, 10.2) 1.020000e&#43;01
    %E 科学计数法，例如 -1234.456E&#43;78 Printf(&#34;%e&#34;, 10.2) 1.020000E&#43;01
    %f 有小数点而无指数，例如 123.456 Printf(&#34;%f&#34;, 10.2) 10.200000
    %g 根据情况选择 %e 或 %f 以产生更紧凑的（无末尾的0）输出 Printf(&#34;%g&#34;, 10.20) 10.2
    %G 根据情况选择 %E 或 %f 以产生更紧凑的（无末尾的0）输出 Printf(&#34;%G&#34;, 10.20&#43;2i) (10.2&#43;2i)
    字符串与字节切片
    占位符 说明 举例 输出
    %s 输出字符串表示（string类型或[]byte) Printf(&#34;%s&#34;, []byte(&#34;Go语言&#34;)) Go语言
    %q 双引号围绕的字符串，由Go语法安全地转义 Printf(&#34;%q&#34;, &#34;Go语言&#34;) &#34;Go语言&#34;
    %x 十六进制，小写字母，每字节两个字符 Printf(&#34;%x&#34;, &#34;golang&#34;) 676f6c616e67
    %X 十六进制，大写字母，每字节两个字符 Printf(&#34;%X&#34;, &#34;golang&#34;) 676F6C616E67
    指针
    占位符 说明 举例 输出
    %p 十六进制表示，前缀 0x Printf(&#34;%p&#34;, &amp;people) 0x4f57f0
    
    %4s 表示输出当前字符串宽度: (&#34;|%4s|,|%6s|&#34;, &#34;aa&#34;, &#34;dd&#34;)  ==&gt;  |  aa|,|    dd|
    ```
  
  - 示例
  
    ```go
    // Go 在传统的`printf` 中对字符串格式化提供了优异的支持。
    // 这里是一些基本的字符串格式化的人物的例子。
    
    package main
    
    import &#34;fmt&#34;
    import &#34;os&#34;
    
    type point struct {
        x, y int
    }
    
    func main() {
    
        // Go 为常规 Go 值的格式化设计提供了多种打印方式。例
        // 如，这里打印了 `point` 结构体的一个实例。
        p := point{1, 2}
        fmt.Printf(&#34;%v\n&#34;, p)
        // 输出：{1 2}
    
        // 如果值是一个结构体，`%&#43;v` 的格式化输出内容将包括
        // 结构体的字段名。
        fmt.Printf(&#34;%&#43;v\n&#34;, p)
        // 输出：{x:1 y:2}
    
        // `%#v` 形式则输出这个值的 Go 语法表示。例如，值的
        // 运行源代码片段。
        fmt.Printf(&#34;%#v\n&#34;, p)
        // 输出：main.point{x:1, y:2}
    
        // 需要打印值的类型，使用 `%T`。
        fmt.Printf(&#34;%T\n&#34;, p)
        // 输出：main.point
    
        // 格式化布尔值是简单的。
        fmt.Printf(&#34;%t\n&#34;, true)
        // 输出：true
    
        // 格式化整形数有多种方式，使用 `%d`进行标准的十进
        // 制格式化。
        fmt.Printf(&#34;%d\n&#34;, 123)
        // 输出：123
    
        // 这个输出二进制表示形式。
        fmt.Printf(&#34;%b\n&#34;, 14)
        // 输出：1110
    
        // 这个输出给定整数的对应字符。
        fmt.Printf(&#34;%c\n&#34;, 33)
        // 输出：!
    
        // `%x` 提供十六进制编码。
        fmt.Printf(&#34;%x\n&#34;, 456)
        // 输出：1c8
    
        // 对于浮点型同样有很多的格式化选项。使用 `%f` 进
        // 行最基本的十进制格式化。
        fmt.Printf(&#34;%f\n&#34;, 78.9)
        // 输出：78.900000
    
        // `%e` 和 `%E` 将浮点型格式化为（稍微有一点不
        // 同的）科学技科学记数法表示形式。
        fmt.Printf(&#34;%e\n&#34;, 123400000.0)
        // 输出：1.234000e&#43;08
        fmt.Printf(&#34;%E\n&#34;, 123400000.0)
        // 输出：1.234000E&#43;08
    
        // 使用 `%s` 进行基本的字符串输出。
        fmt.Printf(&#34;%s\n&#34;, &#34;\&#34;string\&#34;&#34;)
        // 输出：&#34;string&#34;
    
        // 像 Go 源代码中那样带有双引号的输出，使用 `%q`。
        fmt.Printf(&#34;%q\n&#34;, &#34;\&#34;string\&#34;&#34;)
        // 输出：&#34;\&#34;string\&#34;&#34;
    
        // 和上面的整形数一样，`%x` 输出使用 base-16 编码的字
        // 符串，每个字节使用 2 个字符表示。
        fmt.Printf(&#34;%x\n&#34;, &#34;hex this&#34;)
        // 输出：6865782074686973
    
        // 要输出一个指针的值，使用 `%p`。
        fmt.Printf(&#34;%p\n&#34;, &amp;p)
        // 输出：0x42135100
    
        // 当输出数字的时候，你将经常想要控制输出结果的宽度和
        // 精度，可以使用在 `%` 后面使用数字来控制输出宽度。
        // 默认结果使用右对齐并且通过空格来填充空白部分。
        fmt.Printf(&#34;|%6d|%6d|\n&#34;, 12, 345)
        // 输出：|    12|   345|
    
        // 你也可以指定浮点型的输出宽度，同时也可以通过 宽度.
        // 精度 的语法来指定输出的精度。
        fmt.Printf(&#34;|%6.2f|%6.2f|\n&#34;, 1.2, 3.45)
        // 输出：|  1.20|  3.45|
    
        // 要左对齐，使用 `-` 标志。
        fmt.Printf(&#34;|%-6.2f|%-6.2f|\n&#34;, 1.2, 3.45)
        // 输出：|1.20  |3.45  |
    
        // 你也许也想控制字符串输出时的宽度，特别是要确保他们在
        // 类表格输出时的对齐。这是基本的右对齐宽度表示。
        fmt.Printf(&#34;|%6s|%6s|\n&#34;, &#34;foo&#34;, &#34;b&#34;)
        // 输出：|   foo|     b|
    
        // 要左对齐，和数字一样，使用 `-` 标志。
        fmt.Printf(&#34;|%-6s|%-6s|\n&#34;, &#34;foo&#34;, &#34;b&#34;)
        // 输出：|foo   |b     |
    
        // 到目前为止，我们已经看过 `Printf`了，它通过 `os.Stdout`
        // 输出格式化的字符串。`Sprintf` 则格式化并返回一个字
        // 符串而不带任何输出。
        s := fmt.Sprintf(&#34;a %s&#34;, &#34;string&#34;)
        fmt.Println(s)
        // 输出：a string
    
        // 你可以使用 `Fprintf` 来格式化并输出到 `io.Writers`
        // 而不是 `os.Stdout`。
        fmt.Fprintf(os.Stderr, &#34;an %s\n&#34;, &#34;error&#34;)
        // 输出：an error
    }
    ```
  
    

### 字符串类型

```go
a := &#34;hello&#34;
unsafe.Sizeof(a)
/*
输出结果为：16
字符串类型在 go 里是个结构, 包含指向底层数组的指针和长度,这两部分每部分都是 8 个字节，所以字符串类型大小为 16 个字节。
*/
```

- 关键字string,用&#34;&#34;或者``反引号表示(&#39;&#34;&#34;支持控制符号,反引号所有的都会原样输出)
- 占位符 &#39;%s
- 万能占位符&#39;%v&#39;

#### 字符串常用操作

- 长度 **len(str) ** ,返回一个int, 返回的是字节的长度, 中文的字节长3
- 拼接: 使用**&#43;**或者**fmt.Sprintf()**

- 分割: **strings.Split(str,&#39;分割标识&#39;)**, 返回一个切片
- 是否存在: 

| 函数\|返回值                                           | 作用                                                         |
| :----------------------------------------------------- | :----------------------------------------------------------- |
| **strconv 包：**                                       |                                                              |
| **Atoi(s string) (int, error)**                        | **字符串转整型**                                             |
| **strings 包：**                                       |                                                              |
| **Count(s, substr string) int**                        | **计算子串`substr`在字符串`s`中出现的次数**                  |
| Compare(a, b string) int                               | 比较字符串大小                                               |
| **Contains(s, substr string) bool**                    | **判断字符串`s`中是否包含子串`substr`**                      |
| ContainsAny(s, chars string) bool                      | 判断字符串`s`中是否包含`chars`中的某个Unicode字符            |
| ContainsRune(s string, r rune) bool                    | 判断字符串`s`中是否包含rune型值为`r`的字符                   |
| **Index(s, substr string) int**                        | **查找子串`substr`在字符串`s`中第一次出现的位置，如果找不到则返回 -1，如果`substr`为空，则返回 0** |
| LastIndex(s, substr string) int                        | 查找子串`substr`在字符串`s`中最后出现的位置                  |
| IndexRune(s string, r rune) int                        | 查找rune型值为`r`的字符在字符串`s`中出现的起始位置           |
| IndexAny(s, chars string) int                          | 查找字符串`chars`中字符，在字符串`s`中出现的起始位置         |
| LastIndexAny(s, chars string) int                      | 查找字符串`s`中出现`chars`中字符的最后位置                   |
| LastIndexByte(s string, c byte) int                    | 查找byte型字符`c`在字符串`s`中的位置                         |
| SplitN(s, sep string, n int) []string                  | 以字符串`sep`为分隔符，将字符串`s`切分成`n`个子串，结果中**不包含**`sep`本身。如果`sep`为空则将`s`切分为 Unicode 字符列表，如果`s`中没有`sep`子串则整个`s`作为切片 []string 中的第一个元素返回。参数`n`表示最多切出几个子串，`s`超出切分大小时，超出部分不再切分。`n`超出切分子串个数时，返回实际切分子串数。如果`n`为 0，则返回 nil；如果`n`小于 0，则不限制切分个数，全部切分 |
| SplitAfterN(s, sep string, n int) []string             | 以字符串`sep`为分隔符，将字符串`s`切分成`n`个子串，结果中**包含**`sep`本身。如果`sep`为空则将`s`切分为 Unicode 字符列表，如果`s`中没有`sep`子串则整个`s`作为切片 []string 中的第一个元素返回。参数`n`表示最多切出几个子串，`s`超出切分大小时，超出部分不再切分。`n`超出切分子串个数时，返回实际切分子串数。如果`n`为 0，则返回 nil；如果`n`小于 0，则不限制切分个数，全部切分 |
| **Split(s, sep string) []string**                      | **以字符串`sep`为分隔符，将`s`切分成多个子串，结果中不包含`sep`本身。如果`sep`为空，则将`s`切分成 Unicode 字符列表，如果`s`中没有`sep`子串，则将整个`s`作为 []string 的第一个元素返回** |
| SplitAfter(s, sep string) []string                     | 以字符串`sep`为分隔符，将`s`切分成多个子串，结果中**包含**`sep`本身。如果`sep`为空则将`s`切分为 Unicode 字符列表，如果`s`中没有`sep`子串则整个`s`作为切片 []string 中的第一个元素返回。 |
| Fields(s string) []string                              | 以连续的空白字符为分隔符，将`s`切分成多个子串，结果中不包含空白字符本身。空白字符有：\t, \n, \v, \f, \r, &#39;&#39;, U&#43;0085 (NEL), U&#43;00A0 (NBSP) 。如果`s`中只包含空白字符，则返回一个空切片 |
| FieldsFunc(s string, f func(rune) bool) []string       | 以一个或多个满足函数`f(rune)`的字符为分隔符，将`s`切分成多个子串，结果中不包含分隔符本身。如果`s`中没有满足`f(rune)`的字符，则返回一个空切片 |
| **Join(a []string, sep string) string**                | **以`sep`为拼接符，拼接切片`a`中的字符串**                   |
| **HasPrefix(s, prefix string) bool**                   | **判断字符串`s`是否以`prefix`字符串开头，是返回 true，否则返回 false** |
| **HasSuffix(s, suffix string) bool**                   | **判断字符串`s`是否以`suffix`字符串结尾，是返回 true，否则返回 false** |
| Map(f func(rune) rune, s string) string                | 将字符串`s`中满足函数`f(rune)`的字符替换为`f(rune)`的返回值。如果`f(rune)`返回负数，则相应的字符将被删除 |
| Repeat(s string, count int) string                     | 返回字符串`s`重复`count`次数后的结果                         |
| **ToUpper(s string) string**                           | **将字符串`s`中的小写字符转为大写**                          |
| **ToLower(s string) string**                           | **将字符串`s`中的大写字符转为小写**                          |
| ToTitle(s string) string                               | 将字符串`s`中的首个单词转为`Title`形式，大部分字符的`Title`格式就是`Upper`格式 |
| ToUpperSpecial(c unicode.SpecialCase, s string) string | 将字符串`s`中的所有字符修改为其大写格式，优先使用`c`中的规则进行转换 |
| ToLowerSpecial(c unicode.SpecialCase, s string) string | 将字符串`s`中的所有字符修改为其小写格式，优先使用`c`中的规则进行转换 |
| ToTitleSpecial(c unicode.SpecialCase, s string) string | 将字符串`s`中的所有字符修改为其`Title`格式，优先使用`c`中的规则进行转换 |
| Title(s string) string                                 | 将字符串`s`中的所有单词的首字母修改为其`Title`格式（BUG: Title 规则不能正确处理 Unicode 标点符号） |
| TrimLeftFunc(s string, f func(rune) bool) string       | 删除字符串`s`左边连续满足`f(rune)`的字符                     |
| TrimRightFunc(s string, f func(rune) bool) string      | 删除字符串`s`右边连续满足`f(rune)`的字符                     |
| TrimFunc(s string, f func(rune) bool) string           | 删除字符串`s`左右两边连续满足`f(rune)`的字符                 |
| IndexFunc(s string, f func(rune) bool) int             | 查找字符串`s`中第一个满足`f(rune)`的字符的字节位置，没有返回 -1 |
| LastIndexFunc(s string, f func(rune) bool) int         | 查找字符串`s`中最后一个满足`f(rune)`的字符的字节位置，没有返回 -1 |
| **Trim(s string, cutset string) string**               | **删除字符串`s`左右两边连续包含`cutset`的字符**              |
| TrimLeft(s string, cutset string) string               | 删除字符串`s`左边连续包含`cutset`的字符                      |
| TrimRight(s string, cutset string) string              | 删除字符串`s`右边连续包含`cutset`的字符                      |
| **TrimSpace(s string) string**                         | **删除字符串`s`左右两边连续的空白字符**                      |
| TrimPrefix(s, prefix string) string                    | 删除字符串`s` 头部的`prefix`字符串                           |
| TrimSuffix(s, suffix string) string                    | 删除字符串`s` 尾部的`suffix`字符串                           |
| **Replace(s, old, new string, n int) string**          | **替换字符串`s`中的`old`为`new`，如果`old`为空则在`s`中的每个字符间插入`new`包括首尾，`n`为替换次数， -1 时替换所有** |
| **EqualFold(s, t string) bool**                        | **忽略大小写比较字符串`s`和`t`，相同返回 true，反之返回 false** |

**1. 字符串转数字**

- strconv.Atoi：

```go
package main

import (
    &#34;fmt&#34;
    &#34;strconv&#34;
)

func main() {
    var str = &#34;111&#34;
    i, _ := strconv.Atoi(str)
    fmt.Printf(&#34;%d\n&#34;, i) // 输出：111
}
```

**2. 大小写规则转换**
strings.ToUpperSpecial：将字符串`s`中的所有字符修改为其大写格式，优先使用`c`中的规则进行转换
strings.ToLowerSpecial：将字符串`s`中的所有字符修改为其小写格式，优先使用`c`中的规则进行转换
strings.ToTitleSpecial：将字符串`s`中的所有字符修改为其`Title`格式，优先使用`c`中的规则进行转换
`c`规则说明，以下列语句为例：
unicode.CaseRange{&#39;A&#39;, &#39;Z&#39;, [unicode.MaxCase]rune{3, -3, 0}}

- 其中 &#39;A&#39;, &#39;Z&#39; 表示此规则只影响 &#39;A&#39; 到 &#39;Z&#39; 之间的字符。
- 其中`[unicode.MaxCase]rune`数组表示：
- 当使用 ToUpperSpecial 转换时，将字符的 Unicode 编码与第一个元素值（3）相加
- 当使用 ToLowerSpecial 转换时，将字符的 Unicode 编码与第二个元素值（-3）相加
- 当使用 ToTitleSpecial 转换时，将字符的 Unicode 编码与第三个元素值（0）相加

```go
package main

import (
    &#34;fmt&#34;
    &#34;strings&#34;
    &#34;unicode&#34;
)

func main() {
    // 定义转换规则
    var _MyCase = unicode.SpecialCase{
        // 将半角逗号替换为全角逗号，ToTitle 不处理
        unicode.CaseRange{&#39;,&#39;, &#39;,&#39;,
            [unicode.MaxCase]rune{&#39;，&#39; - &#39;,&#39;, &#39;，&#39; - &#39;,&#39;, 0}},
        // 将半角句号替换为全角句号，ToTitle 不处理
        unicode.CaseRange{&#39;.&#39;, &#39;.&#39;,
            [unicode.MaxCase]rune{&#39;。&#39; - &#39;.&#39;, &#39;。&#39; - &#39;.&#39;, 0}},
        // 将 ABC 分别替换为全角的 ＡＢＣ、ａｂｃ，ToTitle 不处理
        unicode.CaseRange{&#39;A&#39;, &#39;C&#39;,
            [unicode.MaxCase]rune{&#39;Ａ&#39; - &#39;A&#39;, &#39;ａ&#39; - &#39;A&#39;, 0}},
    }

    s := &#34;ABCDEF,abcdef.&#34;
    us := strings.ToUpperSpecial(_MyCase, s)
    fmt.Printf(&#34;%q\n&#34;, us) // 输出：&#34;ＡＢＣDEF，ABCDEF。&#34;
    ls := strings.ToLowerSpecial(_MyCase, s)
    fmt.Printf(&#34;%q\n&#34;, ls) // 输出：&#34;ａｂｃdef，abcdef。&#34;
    ts := strings.ToTitleSpecial(_MyCase, s)
    fmt.Printf(&#34;%q\n&#34;, ts) // 输出：&#34;ABCDEF,ABCDEF.&#34;
}
```

### 字符串原理

- 单引号是字符,双引号是字符串

- **字符串的底层就是一个byte数组,所以可以和 []byte 类型互相转换**

- 字符串是由byte字节组成的,所以字符串的长度是byte字节的长度

- rune类型用来表示utf8字符,一个rune字符由1个或多个byte组成

  - 对包含中文的字符串排序

    ```go
    package main
    
    import (
    	&#34;fmt&#34;
    )
    
    func main() {
    	str := &#34;ABCDEFGH你好&#34;
    	strRune := []rune(str)
    
    	for i:=0 ; i&lt;len(strRune)/2;i&#43;&#43;{
    		item := strRune[i]
    		strRune[i] = strRune[len(strRune)-1-i]
    		strRune[len(strRune)-1-i] = item
    	}
    	fmt.Println(len(str)) 	// 14
    	fmt.Println(strRune)    //  [22909 20320 72 71 70 69 68 67 66 65]
    	fmt.Println(string(strRune))   //好你HGFEDCBA
    
    }
    
    ```

    

- 扩展

  ```go
  //1、位：
  数据存储的最小单位。每个二进制数字0或者1就是1个位；
  
  //2、字节：
  8个位构成一个字节；即：1 byte (字节)= 8 bit(位)；
       1 KB = 1024 B(字节)；
       1 MB = 1024 KB;   (2^10 B)
       1 GB = 1024 MB;  (2^20 B)
       1 TB = 1024 GB;   (2^30 B)
  
  //3、字符：
       a、A、中、&#43;、*、の......均表示一个字符；
  		 unicode,万国码,32位既4个字节表示一个字符;
       一般 utf-8 编码下，一个汉字 字符 占用 3 个 字节；
       一般 gbk 编码下，一个汉字  字符  占用 2 个 字节；
  
  //4、字节和字符：
  字节是计算机传输数据的格式，供计算识别的。
  字符是供人类观看的内容
  
  //5、编码：
  编码（encoding）:把…译成密码。==》二进制
  解码（decoding）：破译(尤指密码) ==》破解密码成可以看的懂的
  
  //6.编码格式：
  字节和字符之间转换，参照的规则就是编码格式。
  Unicode 编码共有三种具体实现，分别为utf-8,utf-16,utf-32，其中utf-8占用一到四个字节，utf-16占用二或四个字节，utf-32占用四个字节。
  Unicode码的前128个字符就是ASCII码，之后是ASCII码的扩展码。
  ```




### 数字类型

- Go 也有基于架构的类型，例如：int、uint 和 uintptr。

- 所有整数初始化为0,所有浮点数初始化为0.0,布尔类型初始化为 false

| 序号 | 类型和描述                                                   |
| ---- | ------------------------------------------------------------ |
| 1    | **uint8**无符号 8 位整型 (0 到 255)                          |
| 2    | **uint16**无符号 16 位整型 (0 到 65535)                      |
| 3    | **uint32**无符号 32 位整型 (0 到 4294967295)                 |
| 4    | **uint64**无符号 64 位整型 (0 到 18446744073709551615)       |
| 5    | **int8**有符号 8 位整型 (-128 到 127)                        |
| 6    | **int16**有符号 16 位整型 (-32768 到 32767)                  |
| 7    | **int32**有符号 32 位整型 (-2147483648 到 2147483647)        |
| 8    | **int64**有符号 64 位整型 (-9223372036854775808 到 9223372036854775807) |

#### **浮点型**

| 序号 | 类型和描述                       |
| ---- | -------------------------------- |
| 1    | **float32**IEEE-754 32位浮点型数 |
| 2    | **float64**IEEE-754 64位浮点型数 |
| 3    | **complex64**32 位实数和虚数     |
| 4    | **complex128**64 位实数和虚数    |

------

#### 其他数字类型

以下列出了其他更多的数字类型：

| 序号 | 类型和描述                              |
| ---- | --------------------------------------- |
| 1    | **byte**类似 uint8                      |
| 2    | **rune**类似 int32                      |
| 3    | **uint**32 或 64 位                     |
| 4    | **int**与 uint 一样大小                 |
| 5    | **uintptr**无符号整型，用于存放一个指针 |


---

> 作者: [Fred](https://github.com/ipfred)  
> URL: https://ipfred.github.io/lang/go/go_base/20250515174507/  

