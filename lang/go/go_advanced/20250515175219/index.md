# 1-7.Go中的Defer

# Defer知识点

&gt;Defer 是在函数结束之前,return之前 执行一些逻辑, **return之后的语句先执行，defer后的语句后执行**

## 1. defer的执行顺序

- 多个defer出现的时候，**它是一个“栈”的关系，也就是先进后出**。

## 2. defer与return谁先谁后

- Defer 是在函数结束之前,return之前 执行一些逻辑

- **return之后的语句先执行，defer后的语句后执行**, 除非defer后函数的参数中包含子函数

  &lt;img src=&#34;https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/b86b39eef4df72656bf6b2d713bbbb29_1920x1080.jpeg&#34; alt=&#34;img&#34; style=&#34;zoom:33%;&#34; /&gt;

  ```go
  package main
  
  import &#34;fmt&#34;
  
  func d() string{
  	fmt.Println(&#34;d &#34;)
  	return &#34;derfer return&#34;
  }
  func f()  string{
  	fmt.Println(&#34;f &#34;)
  
  	return &#34;func func&#34;
  }
  
  func do() string  {
  	defer d()
  	return f()
  }
  
  func main() {
  	fmt.Println(do())
  }
  /*
  f
  d
  func func
  */
  
  ```

- defer下的**函数参数**包含子函数, 该子函数会先执行, 其他情况defer后的函数都不会先执行(其他情况如:defer内部包含自执行函数,defer后面跟自执行函数, ) 然后 return后的函数, 然后再执行defer后的函数

  

## 3. 函数的返回值初始化

&gt;**只要声明函数的返回值变量名称，就会在函数初始化时候为之赋值为0，而且在函数体作用域可见**。

- 该知识点不属于defer本身，但是调用的场景却与defer有联系，所以也算是defer必备了解的知识点之一。

- 如 ： `func DeferFunc1(i int) (t int) {}`其中返回值`t int`，这个`t`会在函数起始处被初始化为对应类型的零值并且作用域为整个函数。

  ![img](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/126393f6f0295f9b6bb876c22eb39626_1580x812.png)

## 4. 有名函数返回值遇见defer情况

&gt;通过**知识点2**得知，先return，再defer，所以在执行完return之后，还要再执行defer里的语句，依然可以修改本应该返回的结果。

- 在没有defer的情况下，其实函数的返回就是与return一致的，但是有了defer就不一样了。

  ```go
  package main
  
  import &#34;fmt&#34;
  
  func returnButDefer() (t int) {  //t初始化0， 并且作用域为该函数全域
  
      defer func() {
          t = t * 10
      }()
  
      return 1
  }
  
  func main() {
      fmt.Println(returnButDefer())
  }
  // 输出: 10
  /*
  该returnButDefer()本应的返回值是1，但是在return之后，又被defer的匿名func函数执行，所以t=t*10被执行，最后returnButDefer()返回给上层main()的结果为10
  */
  ```

  

## 5. defer遇见panic

&gt;**defer 最大的功能是 panic 后依然有效**
&gt;所以defer可以保证你的一些资源一定会被关闭，从而避免一些异常出现的问题。

&lt;img src=&#34;https://img.kancloud.cn/d3/00/d300e0e7c008ed40baa6354424569b07_1920x1080.jpeg&#34; alt=&#34;img&#34; style=&#34;zoom:33%;&#34; /&gt;

### 5.1 defer 没有recover

- panic 触发之前, 先进后出执行已经入栈的defer,  然后panic,**程序异常结束**, panic之后的不再执行.

```go
package main

import (
    &#34;fmt&#34;
)

func main() {
    defer_call()

    fmt.Println(&#34;main 正常结束&#34;)
}

func defer_call() {
    defer func() { fmt.Println(&#34;defer: panic 之前1&#34;) }()
    defer func() { fmt.Println(&#34;defer: panic 之前2&#34;) }()

    panic(&#34;异常内容&#34;)  //触发defer出栈

	defer func() { fmt.Println(&#34;defer: panic 之后，永远执行不到&#34;) }()
}
/*
defer: panic 之前2
defer: panic 之前1
panic: 异常内容

goroutine 1 [running]:
main.defer_call()
	/Users/liusaisai/workspace/goProject/src/picturePro/main.go:17 &#43;0x6c
main.main()
	/Users/liusaisai/workspace/goProject/src/picturePro/main.go:8 &#43;0x20
*/
```

### 5.2 defer 中存在recover

- panic 触发之前, 先进后出执行已经入栈的defer,  recover 正常执行, 然后panic,**程序正常结束**, 子函数中panic之后的不再执行,main函数正常执行.

```go
package main

import (
    &#34;fmt&#34;
)

func main() {
    defer_call()

    fmt.Println(&#34;main 正常结束&#34;)
}

func defer_call() {

    defer func() {
        fmt.Println(&#34;defer: panic 之前1, 捕获异常&#34;)
        if err := recover(); err != nil {
            fmt.Println(err)
        }
    }()

    defer func() { fmt.Println(&#34;defer: panic 之前2, 不捕获&#34;) }()

    panic(&#34;异常内容&#34;)  //触发defer出栈

		defer func() { fmt.Println(&#34;defer: panic 之后, 永远执行不到&#34;) }()
}

/*
defer: panic 之前2, 不捕获
defer: panic 之前1, 捕获异常
异常内容
main 正常结束
*/
```

## 6. defer中包含panic

&gt;**panic仅有最后一个可以被revover捕获**。

- 触发`panic(&#34;panic&#34;)`后defer顺序出栈执行，第一个被执行的defer中 会有`panic(&#34;defer panic&#34;)`异常语句，这个异常将会覆盖掉main中的异常`panic(&#34;panic&#34;)`，最后这个异常被第二个执行的defer捕获到。

```go
package main

import (
    &#34;fmt&#34;
)

func main()  {

    defer func() {
       if err := recover(); err != nil{
           fmt.Println(err)
       }else {
           fmt.Println(&#34;fatal&#34;)
       }
    }()

    defer func() {
        panic(&#34;defer panic&#34;)
    }()

    panic(&#34;panic&#34;)
}

// &#34;defer panic&#34;
```

## 7. defer下的函数参数包含子函数

&gt;defer下的**函数参数**包含子函数, 该子函数会先执行, 其他情况defer后的函数都不会先执行(其他情况如:defer内部包含自执行函数,defer后面跟自执行函数, ) 

- defer压栈function1，压栈函数地址、形参1、形参2(调用function3) --&gt; 打印3
- defer压栈function2，压栈函数地址、形参1、形参2(调用function4) --&gt; 打印4
- defer出栈function2, 调用function2 --&gt; 打印2
- defer出栈function1, 调用function1--&gt; 打印1

```go
package main

import &#34;fmt&#34;

func function(index int, value int) int {

    fmt.Println(index)

    return index
}

func main() {
    defer function(1, function(3, 0))
    defer function(2, function(4, 0))
}
```

# 经典练习(一定要看)

- 题解: https://www.kancloud.cn/aceld/golang/1958310#3_92

1. ```go
   package main
   
   import &#34;fmt&#34;
   
   func DeferFunc1(i int) (t int) {
       t = i
       defer func() {
           t &#43;= 3
       }()
       return t  // 返回的是t
   }
   
   func DeferFunc2(i int) int {
       t := i
       defer func() {
           t &#43;= 3
       }()
       return t // 将t赋值给返回值
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
           fmt.Println(t)  // 闭包引入外部变量t
       }(t)  // t的值被压入栈, 此时为0
       t = 1
       return 2
   }
   
   func main() {
       fmt.Println(DeferFunc1(1)) 
       fmt.Println(DeferFunc2(1))
       fmt.Println(DeferFunc3(1)) 
       DeferFunc4() 
   }
   ```

---

> 作者: [Fred](https://github.com/ipfred)  
> URL: https://ipfred.github.io/lang/go/go_advanced/20250515175219/  

