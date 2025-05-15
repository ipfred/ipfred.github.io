# 12. 正则表达式

## 编译函数

&gt; Compile函数或MustCompile函数是将正则表达式进行编译，返回优化的 Regexp 结构体

1. `Compile`: 返回两个参数*Regexp,error类型
1. `MustCompile`: 只返回*Regexp类型

## 正则匹配

### 1. MatchString

&gt; 返回的**第一个参数是bool类型**即匹配结果，**第二个参数是error类型**

- demo

  ```go
  sourceStr := `my email is gerrylon@163.com`
  matched, _ := regexp.MatchString(`[\w-]&#43;@[\w]&#43;(?:\.[\w]&#43;)&#43;`, sourceStr)
  fmt.Printf(&#34;%v&#34;, matched) // true
  ```

### 2. FindString

&gt; **返回一个字符串**，该字符串具有最左边匹配的文本。如果找不到匹配项，则返回空字符串。

- demo

  ```go
  r,_:=regexp.Compile(&#34;p([a-z]&#43;)ch&#34;)
  //查找匹配的字符串
  fmt.Println(r.FindString(&#34;peach punch&#34;))  //打印结果：peach
  ```

  

### 3. FindStringIndex

&gt; 查找第一个匹配**字符串开始**和**结束位置的索引**，而不是匹配内容

- demo

  ```go
  r,_:=regexp.Compile(&#34;p([a-z]&#43;)ch&#34;)
  // 查找匹配字符串开始和结束位置的索引，而不是匹配内容[0 5]
  fmt.Println(r.FindStringIndex(&#34;peach punch&#34;))  //打印结果： [0 5]
  ```

  

### 4. FindStringSubmatch

&gt; 查找第一个， 返回完全匹配和局部匹配的字符串

- dmeo

  ```go
  r,_:=regexp.Compile(&#34;p([a-z]&#43;)ch&#34;)
  //返回完全匹配和局部匹配的字符串，例如，这里会返回  p([a-z]&#43;)ch 和 `([a-z]&#43;) 的信息
  fmt.Println(r.FindStringSubmatch(&#34;peach punch&#34;))   //打印结果：[peach ea]
  ```

### 5. FindAllString

&gt;查找字符串中所有符合规则的，可以指定个数

- demo

  ```go
  r, _ := regexp.Compile(&#34;p([a-z]&#43;)ch&#34;)
  fmt.Println(r.FindAllString(&#34;aapeach punch pqwerch&#34;, 2)) //打印结果： [peach punch]
  ```

### 6. FindStringIndex

&gt; 查找全部 匹配**字符串开始**和**结束位置的索引**，而不是匹配内容

- demo

  ```go
  	r, _ := regexp.Compile(&#34;p([a-z]&#43;)ch&#34;)
  	fmt.Println(r.FindAllStringIndex(&#34;aapeach punch pqwerch&#34;, 2)) //打印结果： [[2 7] [8 13]]
  ```

  

### 7. FindAllStringSubmatch

&gt; 返回全部的 完全匹配和局部匹配的字符串，可以指定个数

- demo

  ```go
  	r, _ := regexp.Compile(&#34;p([a-z]&#43;)ch&#34;)
  	fmt.Println(r.FindAllStringSubmatch(&#34;aapeach punch pqwerch&#34;, 2)) //打印结果： [[peach ea] [punch un]]
  ```

  

### 7. ReplaceAllString

&gt; 将匹配的结果，替换成新输入的结果, 没有匹配到返回原string

- demo

  ```go
  	r, _ := regexp.Compile(&#34;p([a-z]&#43;)ch&#34;)
  	fmt.Println(r.ReplaceAllString(&#34;aapeach punch pqwerch&#34;, &#34;-&#34;)) //打印结果： aa- - -
  ```

  

### 8. ReplaceAllFunc

- demo

  ```go
  //Func 变量允许传递匹配内容到一个给定的函数中，
  in:=[]byte(&#34;a peach&#34;)
  out:=r1.ReplaceAllFunc(in,bytes.ToUpper)
  fmt.Printf(string(out)) //打印结果：   a PEACH
  ```

  

---

> 作者: [Fred](https://github.com/ipfred)  
> URL: https://ipfred.github.io/lang/go/go_base/20250515175142/  

