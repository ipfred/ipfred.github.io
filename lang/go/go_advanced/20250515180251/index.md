# 14.位运算

## itoa 位运算

- code-1

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

  - const 声明第一个常量必须指定一个表达式，后续的常量如果没有表达式，则继承上面的表达式。

  - iota 表示从 0 开始自动加 1，所以 **i=1&lt;&lt;0**, **j=3&lt;&lt;1**（**&lt;&lt;** 表示左移的意思），即：i=1, j=6，这没问题，关键在 k 和 l，从输出结果看 **k=3&lt;&lt;2**，**l=3&lt;&lt;3**。

    - 简单表述:
      - **i=1**：左移 0 位,不变仍为 1;
      - **j=3**：左移 1 位,变为二进制 110, 即 6;

      - **k=3**：左移 2 位,变为二进制 1100, 即 12;

      - **l=3**：左移 3 位,变为二进制 11000,即 24;

    注：**&lt;&lt;n==\*(2^n)**。

    ```go
    // 左移运算符 &lt;&lt; 是双目运算符。左移 n 位就是乘以 2 的 n 次方。 其功能把 &lt;&lt; 左边的运算数的各二进位全部左移若干位，由 &lt;&lt; 右边的数指定移动的位数，高位丢弃，低位补 0。
    
    //右移运算符 &gt;&gt; 是双目运算符。右移 n 位就是除以 2 的 n 次方。 其功能是把 &gt;&gt; 左边的运算数的各二进位全部右移若干位， &gt;&gt; 右边的数指定移动的位数。
    ```

- code-2

  ```go
  const (
      bit0, mask0 = 1 &lt;&lt; iota, 1&lt;&lt;iota - 1   //const声明第0行，即iota==0
      bit1, mask1                            //const声明第1行，即iota==1, 表达式继承上面的语句
      _, _                                   //const声明第2行，即iota==2
      bit3, mask3                            //const声明第3行，即iota==3
  )
  ```

  - 第0行的表达式展开即`bit0, mask0 = 1 &lt;&lt; 0, 1&lt;&lt;0 - 1`，所以bit0 == 1，mask0 == 0；
  - 第1行没有指定表达式继承第一行，即`bit1, mask1 = 1 &lt;&lt; 1, 1&lt;&lt;1 - 1`，所以bit1 == 2，mask1 == 1；
  - 第2行没有定义常量
  - 第3行没有指定表达式继承第一行，即`bit3, mask3 = 1 &lt;&lt; 3, 1&lt;&lt;3 - 1`，所以bit0 == 8，mask0 == 7；

## 简单位运算

### 1. 左偏移

- 左偏移就是数字乘以2的偏移量次方

  ```go
  3 &lt;&lt; 4   ==&gt;   3*math.Pow(2,4)  ==&gt; 48
  ```

  

### 2. 右偏移

- 右偏移就是数字除以2的偏移量次方

  ```go
  16 &gt;&gt; 3  ==&gt;   16/math.Pow(2,3)  ==&gt;2
  ```

  

---

> 作者: [Fred](https://github.com/ipfred)  
> URL: https://ipfred.github.io/lang/go/go_advanced/20250515180251/  

