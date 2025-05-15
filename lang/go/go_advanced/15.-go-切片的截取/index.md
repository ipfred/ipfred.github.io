# 

## 截取容量问题

&gt; 切片截取子切片时，会造成临时内存泄露， 主要原因有两个
&gt;
&gt; 1. 切片截取时，新旧切片会共用一个底层数组
&gt; 2. 切片的底层结构体指向数组的指针只是一个头指针

- demo

  ```go
  package main
  
  import &#34;fmt&#34;
  
  func main() {
  	a := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
  	c := a[1:2]
  	fmt.Println(len(c), cap(c))     // 1,9   c的数组头指针执行索引1，所以容量为9
  }
  ```

- 解决办法

  1. 使用copy，不过要注意copy时的长度和容量问题

  2. 使用`slice [1:2:3]` 两个冒号语法截取：`[startIndex:endIndex:max]`, **其中 max 的值一定要大于 endIndex**

     - 新切片的容量就是`max - startIndex`,
     - 实际引用的数组时从数组`startIndex`索引开始到`max`索引为止，但不包括max索引处的元素,
     - 新切片的长度就是`endIndex - startIndex `

     ```go
     package main
     
     import &#34;fmt&#34;
     
     func main() {
     	a := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
     	b := a[2:5:6]
       fmt.Println(len(b), cap(b))  // 3 ,4    len(b) = 5-2,  cap(b) = 6-2
     }
     ```

     

---

> 作者:   
> URL: http://localhost:1313/lang/go/go_advanced/15.-go-%E5%88%87%E7%89%87%E7%9A%84%E6%88%AA%E5%8F%96/  

