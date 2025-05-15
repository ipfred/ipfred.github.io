# 

## 什么是接口型函数？

&gt; 某一函数类型实现了接口，该函数类型调用接口实现方法时再调用该函数类型本体，这种函数叫做接口型函数。

- 示例代码

  ```go
  // 调用器接口
  type Invoker interface {
  	Call(interface{}) error
  }
  
  // 定义函数为类型
  type FuncCaller func(interface{}) error
  
  // 接口实现
  func (f FuncCaller) Call(i interface{}) error {
  	return f(i)  // 调用函数f本体
  }
  ```

  

## 优点

1. 可以完美使用接口优点，不必将某个接口函数附在某个type上面
2. 可以直接调用函数或者使用该接口，非常灵活

- code demo

  ```go
  // interface_func project main.go 接口型函数基本使用
  package main
  
  import &#34;fmt&#34;
  
  type Handler interface {
      Do(k, v interface{})
  }
  
  type HandlerFunc func(k, v interface{})
  
  func (f HandlerFunc) Do(k, v interface{}) {
      f(k, v)
  }
  
  func Each(m map[interface{}]interface{}, h Handler) {
      if m != nil &amp;&amp; len(m) &gt; 0 {
          for k, v := range m {
              h.Do(k, v)
          }
      }
  }
  
  func SelfInfo(k, v interface{}) {
      fmt.Printf(&#34;我是%s,今年%d岁了\n&#34;, k, v)
  }
  
  func EachFunc(m map[interface{}]interface{}, f func(k, v interface{})) {
      Each(m, HandlerFunc(f))
  }
  
  func main() {
      SelfInfo(&#34;chaozhou&#34;, 23) //单独调用
      SelfInfo(&#34;lisi&#34;, 24)     //单独调用
      person := make(map[interface{}]interface{})
      person[&#34;chaozhou&#34;] = 23
      person[&#34;lisi&#34;] = 24
      EachFunc(person, SelfInfo) //函数接口参数调用
  }
  ```

  

## 应用场景

- `net/http` 库中`handler`和`handlerFunc`

---

> 作者:   
> URL: http://localhost:1313/lang/go/go_advanced/13-2.-go-%E6%8E%A5%E5%8F%A3%E5%9E%8B%E5%87%BD%E6%95%B0/  

