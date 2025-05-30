# 3-3. 数组和切片

# 数组

- 数组是具有相同唯一类型的一组已编号且长度固定的数据项序列，这种类型可以是任意的原始类型例如整型、字符串或者自定义类型。
- **声明数组的时候必须声明长度或者用[...], 不然就是切片.**

- 初始化数组:

```go
var balance = [5]float32{1000.0, 2.0, 3.4, 7.0, 50.0}
```

- 如果数组长度不确定，可以使用 **...** 代替数组的长度，编译器会根据元素个数自行推断数组的长度：

```go
balance := [...]float32{1000.0, 2.0, 3.4, 7.0, 50.0}
```

- 如果设置了数组的长度，我们还可以通过指定下标来初始化元素：

```go
//  将索引为 1 和 3 的元素初始化
balance := [5]float32{1:2.0,3:7.0}
```



# 切片(slice)

- Go 语言切片是对数组的抽象。也可以被看做为&#34;动态数组&#34;,数组的长度不可改变,切片长度可变

- 在做函数调用时，slice 按引用传递，array 按值传递：

- **切片是对数组的引用,切片本身并不包含任何元素** 

- 切片的结构包括三个部分:

  - **地址**: 切片的地址一般指切片中的第一个元素所指向的内存地址, 用十六进制表示;
  - **长度**: 切片实际存在元素的个数;
  - **容量**: **从切片的起始元素开始到其底层数组中最后一个元素的个数**;

- **切片的长度和容量都不是固定的,追加元素会使切片的长度和容量都增大**
- 切片如果是从其他数组或者切片中来的话, 切片容量增加但是所引用数组容量不变
  - 切片如果是从其他数组或者切片中来的话, 当切片长度大于多引用的数组容量时; 切片容量会以 **切片新容量=2*切片当前容量** 的速度扩容
  
- 切片如果是从其他数组或者切片中来的话, 当前片长量大于所引用数组的容量时, 切片中的第一个元素所指向的内存地址会发生改变;

**定义切片**

- 声明一个未指定大小的数组来定义切片(**切片不需要说明长度**)：

```go
var identifier []type
```

- 使用 **make()** 函数来创建切片:

```go
var slice1 []type = make([]type, len)

也可以简写为

slice1 := make([]type, len)
```

- 也可以指定容量，其中 **capacity** 为可选参数。

```go
make([]T, length, capacity)
//这里 len 是数组的长度并且也是切片的初始长度。
```

- 特殊

  ```go
  a := []int{2: 1}  // 声明切片索引为2的写入1
  ```

  

### 切片函数

- len()  获取长度

- cap()  获取切片容量,即最大长度

- append()  往切片尾部添加一个元素

- copy()   *拷贝 numbers 的内容到 numbers1*

  ```go
  /* 拷贝 numbers 的内容到 numbers1 */
  copy(numbers1,numbers)
  ```

- 合并多个数组：

  ```go
  package main
  import &#34;fmt&#34;
  
  func main() {
      var arr1 = []int{1,2,3}
      var arr2 = []int{4,5,6}
      var arr3 = []int{7,8,9}
      var s1 = append(append(arr1, arr2...), arr3...)
      fmt.Printf(&#34;s1: %v\n&#34;, s1)
  }
  // s1: [1 2 3 4 5 6 7 8 9]
  ```

- 从切片中删除元素

  ```go
  package main
  
  import &#34;fmt&#34;
  
  func main() {
  	sli1:=[]int{1,2,3}
    c:=append(sli1[0:1],sli1[2:]...)
  	fmt.Println(c)
  }
  // [1 3]
  ```

  

- 插入切片头部

  ```go
  a := []int{1, 2}
  a :=append([]int{1},a...)
  ```

  

- 插入切片任意位置

  ```go
  a := []int{1, 2, 3, 4, 5}
  a := append(a, 0) //先把原来的切片长度&#43;1
  index := 2 //要把新元素插入到第二个位置
  copy(a[index&#43;1:], a[index:])
  a[index] = 0 //新元素的值是0
  ```

  

![](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/bigox.top-20220225175824520.svg)


---

> 作者: [Fred](https://github.com/ipfred)  
> URL: https://ipfred.github.io/lang/go/go_base/20250515175051/  

