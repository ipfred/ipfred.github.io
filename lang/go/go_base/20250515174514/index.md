# 2-3. 运算符

## 运算符

Go 语言内置的运算符有：

- 算术运算符
- 关系运算符
- 逻辑运算符
- 位运算符
- 赋值运算符
- 其他运算符

### 算术运算符

下表列出了所有Go语言的算术运算符。假定 A 值为 10，B 值为 20。

| 运算符 | 描述 | 实例               |
| :----- | :--- | :----------------- |
| &#43;      | 相加 | A &#43; B 输出结果 30  |
| -      | 相减 | A - B 输出结果 -10 |
| *      | 相乘 | A * B 输出结果 200 |
| /      | 相除 | B / A 输出结果 2   |
| %      | 求余 | B % A 输出结果 0   |
| &#43;&#43;     | 自增 | A&#43;&#43; 输出结果 11    |
| --     | 自减 | A-- 输出结果 9     |



### 关系运算符

下表列出了所有Go语言的关系运算符。假定 A 值为 10，B 值为 20。

| 运算符 | 描述                                                         | 实例              |
| :----- | :----------------------------------------------------------- | :---------------- |
| ==     | 检查两个值是否相等，如果相等返回 True 否则返回 False。       | (A == B) 为 False |
| !=     | 检查两个值是否不相等，如果不相等返回 True 否则返回 False。   | (A != B) 为 True  |
| &gt;      | 检查左边值是否大于右边值，如果是返回 True 否则返回 False。   | (A &gt; B) 为 False  |
| &lt;      | 检查左边值是否小于右边值，如果是返回 True 否则返回 False。   | (A &lt; B) 为 True   |
| &gt;=     | 检查左边值是否大于等于右边值，如果是返回 True 否则返回 False。 | (A &gt;= B) 为 False |
| &lt;=     | 检查左边值是否小于等于右边值，如果是返回 True 否则返回 False。 | (A &lt;= B) 为 True  |



### 逻辑运算符

下表列出了所有Go语言的逻辑运算符。假定 A 值为 True，B 值为 False。

| 运算符 | 描述                                                         | 实例               |
| :----- | :----------------------------------------------------------- | :----------------- |
| &amp;&amp;     | 逻辑 AND 运算符。 如果两边的操作数都是 True，则条件 True，否则为 False。 | (A &amp;&amp; B) 为 False  |
| \|\|   | 逻辑 OR 运算符。 如果两边的操作数有一个 True，则条件 True，否则为 False。 | (A \|\| B) 为 True |
| !      | 逻辑 NOT 运算符。 如果条件为 True，则逻辑 NOT 条件 False，否则为 True。 | !(A &amp;&amp; B) 为 True  |



### 位运算符

位运算符对整数在内存中的二进制位进行操作。

下表列出了位运算符 &amp;, |, 和 ^ 的计算：

| p    | q    | p &amp; q | p \| q | p ^ q |
| :--- | :--- | :---- | :----- | :---- |
| 0    | 0    | 0     | 0      | 0     |
| 0    | 1    | 0     | 1      | 1     |
| 1    | 1    | 1     | 1      | 0     |
| 1    | 0    | 0     | 1      | 1     |

假定 A = 60; B = 13; 其二进制数转换为：

```go
A = 0011 1100

B = 0000 1101

-----------------

A&amp;B = 0000 1100

A|B = 0011 1101

A^B = 0011 0001
```



Go 语言支持的位运算符如下表所示。假定 A 为60，B 为13：

| 运算符 | 描述                                                         | 实例                                   |
| :----- | :----------------------------------------------------------- | :------------------------------------- |
| &amp;      | 按位与运算符&#34;&amp;&#34;是双目运算符。 其功能是参与运算的两数各对应的二进位相与。 | (A &amp; B) 结果为 12, 二进制为 0000 1100  |
| \|     | 按位或运算符&#34;\|&#34;是双目运算符。 其功能是参与运算的两数各对应的二进位相或 | (A \| B) 结果为 61, 二进制为 0011 1101 |
| ^      | 按位异或运算符&#34;^&#34;是双目运算符。 其功能是参与运算的两数各对应的二进位相异或，当两对应的二进位相异时，结果为1。 | (A ^ B) 结果为 49, 二进制为 0011 0001  |
| &lt;&lt;     | 左移运算符&#34;&lt;&lt;&#34;是双目运算符。左移n位就是乘以2的n次方。 其功能把&#34;&lt;&lt;&#34;左边的运算数的各二进位全部左移若干位，由&#34;&lt;&lt;&#34;右边的数指定移动的位数，高位丢弃，低位补0。 | A &lt;&lt; 2 结果为 240 ，二进制为 1111 0000 |
| &gt;&gt;     | 右移运算符&#34;&gt;&gt;&#34;是双目运算符。右移n位就是除以2的n次方。 其功能是把&#34;&gt;&gt;&#34;左边的运算数的各二进位全部右移若干位，&#34;&gt;&gt;&#34;右边的数指定移动的位数。 | A &gt;&gt; 2 结果为 15 ，二进制为 0000 1111  |

以下实例演示了位运算符的用法：

```go
package main

import &#34;fmt&#34;

func main() {

  var a uint = 60   /* 60 = 0011 1100 */
  var b uint = 13   /* 13 = 0000 1101 */
  var c uint = 0      

  c = a &amp; b    */\* 12 = 0000 1100 \*/*
  fmt.Printf(&#34;第一行 - c 的值为 %d**\n**&#34;, c )

  c = a | b    */\* 61 = 0011 1101 \*/*
  fmt.Printf(&#34;第二行 - c 的值为 %d**\n**&#34;, c )

  c = a ^ b    */\* 49 = 0011 0001 \*/*
  fmt.Printf(&#34;第三行 - c 的值为 %d**\n**&#34;, c )

  c = a &lt;&lt; 2   */\* 240 = 1111 0000 \*/*
  fmt.Printf(&#34;第四行 - c 的值为 %d**\n**&#34;, c )

  c = a &gt;&gt; 2   */\* 15 = 0000 1111 \*/*
  fmt.Printf(&#34;第五行 - c 的值为 %d**\n**&#34;, c )
}

/*
第一行 - c 的值为 12
第二行 - c 的值为 61
第三行 - c 的值为 49
第四行 - c 的值为 240
第五行 - c 的值为 15
*/
```



### 赋值运算符

下表列出了所有Go语言的赋值运算符。

| 运算符 | 描述                                           | 实例                                  |
| :----- | :--------------------------------------------- | :------------------------------------ |
| =      | 简单的赋值运算符，将一个表达式的值赋给一个左值 | C = A &#43; B 将 A &#43; B 表达式结果赋值给 C |
| &#43;=     | 相加后再赋值                                   | C &#43;= A 等于 C = C &#43; A                 |
| -=     | 相减后再赋值                                   | C -= A 等于 C = C - A                 |
| *=     | 相乘后再赋值                                   | C *= A 等于 C = C * A                 |
| /=     | 相除后再赋值                                   | C /= A 等于 C = C / A                 |
| %=     | 求余后再赋值                                   | C %= A 等于 C = C % A                 |
| &lt;&lt;=    | 左移后赋值                                     | C &lt;&lt;= 2 等于 C = C &lt;&lt; 2               |
| &gt;&gt;=    | 右移后赋值                                     | C &gt;&gt;= 2 等于 C = C &gt;&gt; 2               |
| &amp;=     | 按位与后赋值                                   | C &amp;= 2 等于 C = C &amp; 2                 |
| ^=     | 按位异或后赋值                                 | C ^= 2 等于 C = C ^ 2                 |
| \|=    | 按位或后赋值                                   | C \|= 2 等于 C = C \| 2               |



### 其他运算符

下表列出了Go语言的其他运算符。

| 运算符 | 描述             | 实例                       |
| :----- | :--------------- | :------------------------- |
| &amp;      | 返回变量存储地址 | &amp;a; 将给出变量的实际地址。 |
| *      | 指针变量。       | *a; 是一个指针变量         |

```go
package main

import &#34;fmt&#34;

func main() {
   var a int = 4
   var b int32
   var c float32
   var ptr *int

   /* 运算符实例 */
   fmt.Printf(&#34;第 1 行 - a 变量类型为 = %T\n&#34;, a );
   fmt.Printf(&#34;第 2 行 - b 变量类型为 = %T\n&#34;, b );
   fmt.Printf(&#34;第 3 行 - c 变量类型为 = %T\n&#34;, c );

   /*  &amp; 和 * 运算符实例 */
   ptr = &amp;a     /* &#39;ptr&#39; 包含了 &#39;a&#39; 变量的地址 */
   fmt.Printf(&#34;a 的值为  %d\n&#34;, a);
   fmt.Printf(&#34;*ptr 为 %d\n&#34;, *ptr);
}
```



### 运算符优先级

有些运算符拥有较高的优先级，二元运算符的运算方向均是从左至右。下表列出了所有运算符以及它们的优先级，由上至下代表优先级由高到低：

| 优先级 | 运算符           |
| :----- | :--------------- |
| 5      | * / % &lt;&lt; &gt;&gt; &amp; &amp;^ |
| 4      | &#43; - \| ^         |
| 3      | == != &lt; &lt;= &gt; &gt;=  |
| 2      | &amp;&amp;               |
| 1      | \|\|             |

## Golang代码执行步骤

![image-20210329145018401](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/image-20210329145018401.png)

---

> 作者: [Fred](https://github.com/ipfred)  
> URL: https://ipfred.github.io/lang/go/go_base/20250515174514/  

