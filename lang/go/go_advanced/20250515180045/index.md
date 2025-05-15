# 1-8.Go和Python中的深浅拷贝

## 1. Golang

- go 深拷贝， 就是拷贝值
- go 浅拷贝， 拷贝引用
- go中赋值就能实现拷贝，**针对引用类型（slice，map，channel）是浅拷贝，对值类型是深拷贝**

1、深拷贝（Deep Copy）：

拷贝的是数据本身，创造一个样的新对象，新创建的对象与原对象不共享内存，新创建的对象在内存中开辟一个新的内存地址，新对象值修改时不会影响原对象值。既然内存地址不同，释放内存地址时，可分别释放。

值类型的数据，默认全部都是深复制，**Array、Int、String、Struct、Float，Bool**。

2、浅拷贝（Shallow Copy）：

拷贝的是数据地址，只复制指向的对象的指针，此时新对象和老对象指向的内存地址是一样的，新对象值修改时老对象也会变化。释放内存地址时，同时释放内存地址。

引用类型的数据，默认全部都是浅复制，Slice，Map,  channel。

二、本质区别：
是否真正获取（复制）对象实体，而不是引用。

三、如何理解？
这里举个例子，比如P2复制了P1，修改P1属性的时候，观察P2的属性是否会产生变化

1、P2的属性变化了，说明这是浅拷贝，堆中内存还是同一个值。

p2=&amp;p1 // 浅拷贝，p2为指针，p1和p2共用一个内存地址
2、P2的属性没变化，说明这是深拷贝，堆中内存是不同的值了。

p2=p1 // 深拷贝，生成两个内存地址
四、演示示例：
深拷贝示例：

```golang
package main

import (
   &#34;fmt&#34;
)

// 定义一个Robot结构体
type Robot struct {
   Name  string
   Color string
   Model string
}

func main() {
   fmt.Println(&#34;深拷贝 内容一样，改变其中一个对象的值时，另一个不会变化。&#34;)
   robot1 := Robot{
      Name:  &#34;小白-X型-V1.0&#34;,
      Color: &#34;白色&#34;,
      Model: &#34;小型&#34;,
   }
   robot2 := robot1
   fmt.Printf(&#34;Robot 1：%s\t内存地址：%p \n&#34;, robot1, &amp;robot1)
   fmt.Printf(&#34;Robot 2：%s\t内存地址：%p \n&#34;, robot2, &amp;robot2)

   fmt.Println(&#34;修改Robot1的Name属性值&#34;)
   robot1.Name = &#34;小白-X型-V1.1&#34;

   fmt.Printf(&#34;Robot 1：%s\t内存地址：%p \n&#34;, robot1, &amp;robot1)
   fmt.Printf(&#34;Robot 2：%s\t内存地址：%p \n&#34;, robot2, &amp;robot2)

}
```

运行结果：

```
深拷贝 内容一样，改变其中一个对象的值时，另一个不会变化。
Robot 1：{小白-X型-V1.0 白色 小型}      内存地址：0xc000072330
Robot 2：{小白-X型-V1.0 白色 小型}      内存地址：0xc000072360
修改Robot1的Name属性值
Robot 1：{小白-X型-V1.1 白色 小型}      内存地址：0xc000072330
Robot 2：{小白-X型-V1.0 白色 小型}      内存地址：0xc000072360
```

深拷贝中，我们可以看到Robot1号的地址与Robot2号的内存地址是不同的，修改Robot1号的Name属性时，Robot2号不会变化。

浅拷贝我们用两种方式来介绍。

浅拷贝示例1：

```go
package main

import (
   &#34;fmt&#34;
)

// 定义一个Robot结构体
type Robot struct {
   Name  string
   Color string
   Model string
}

func main() {

   fmt.Println(&#34;浅拷贝 内容和内存地址一样，改变其中一个对象的值时，另一个同时变化。&#34;)
   robot1 := Robot{
      Name:  &#34;小白-X型-V1.0&#34;,
      Color: &#34;白色&#34;,
      Model: &#34;小型&#34;,
   }
   robot2 := &amp;robot1
   fmt.Printf(&#34;Robot 1：%s\t内存地址：%p \n&#34;, robot1, &amp;robot1)
   fmt.Printf(&#34;Robot 2：%s\t内存地址：%p \n&#34;, robot2, robot2)

   fmt.Println(&#34;在这里面修改Robot1的Name和Color属性&#34;)
   robot1.Name = &#34;小黑-X型-V1.1&#34;
   robot1.Color = &#34;黑色&#34;

   fmt.Printf(&#34;Robot 1：%s\t内存地址：%p \n&#34;, robot1, &amp;robot1)
   fmt.Printf(&#34;Robot 2：%s\t内存地址：%p \n&#34;, robot2, robot2)

}
```

运行结果1：

```
浅拷贝 内容和内存地址一样，改变其中一个对象的值时，另一个同时变化。
Robot 1：{小白-X型-V1.0 白色 小型}      内存地址：0xc000062330
Robot 2：&amp;{小白-X型-V1.0 白色 小型}     内存地址：0xc000062330
在这里面修改Robot1的Name和Color属性
Robot 1：{小黑-X型-V1.1 黑色 小型}      内存地址：0xc000062330
Robot 2：&amp;{小黑-X型-V1.1 黑色 小型}     内存地址：0xc000062330
```

浅拷贝中，我们可以看到Robot1和Robot2的内存地址是相同的，修改其中一个对象的属性时，另一个也会产生变化。

浅拷贝示例2：

```go
package main

import (
   &#34;fmt&#34;
)

// 定义一个Robot结构体
type Robot struct {
   Name  string
   Color string
   Model string
}

func main() {

   fmt.Println(&#34;浅拷贝 使用new方式&#34;)
   robot1 := new(Robot)
   robot1.Name = &#34;小白-X型-V1.0&#34;
   robot1.Color = &#34;白色&#34;
   robot1.Model = &#34;小型&#34;

   robot2 := robot1
   fmt.Printf(&#34;Robot 1：%s\t内存地址：%p \n&#34;, robot1, robot1)
   fmt.Printf(&#34;Robot 2：%s\t内存地址：%p \n&#34;, robot2, robot2)

   fmt.Println(&#34;在这里面修改Robot1的Name和Color属性&#34;)
   robot1.Name = &#34;小蓝-X型-V1.2&#34;
   robot1.Color = &#34;蓝色&#34;

   fmt.Printf(&#34;Robot 1：%s\t内存地址：%p \n&#34;, robot1, robot1)
   fmt.Printf(&#34;Robot 2：%s\t内存地址：%p \n&#34;, robot2, robot2)
}
```

运行结果：

```
浅拷贝 使用new方式
Robot 1：&amp;{小白-X型-V1.0 白色 小型}     内存地址：0xc000068330
Robot 2：&amp;{小白-X型-V1.0 白色 小型}     内存地址：0xc000068330
在这里面修改Robot1的Name和Color属性
Robot 1：&amp;{小黑-X型-V1.2 黑色 小型}     内存地址：0xc000068330
Robot 2：&amp;{小黑-X型-V1.2 黑色 小型}     内存地址：0xc000068330
```

new操作，robot2 := robot1，看上去是深拷贝，其实是浅拷贝，robot2和robot1两个指针共用同一个内存地址。

## 2. Python

```python
对于不可变数据类型来说都一样,对于可变数据类型:
# 浅拷贝:
只拷贝数据的第一层
# 深拷贝:
对于可变的数据类型,拷贝拷贝嵌套层级中的所有可变类型
----------------------------------------------------------
id(查看内存地址),赋值更改内存地址,内部变更改变量的值
# 内存地址
a = 1
b = 1
id(a) = id(b)
#按理说a与b的id不该一样,但是在python中,为了提高运算性能,对某些特殊情况进行了缓存.(小数据池)缓存
对象:
1. 整型： -5 ~ 256
2. 字符串：&#34;alex&#34;,&#39;asfasd asdf asdf d_asdf &#39; ----&#34;f_*&#34; * 3 - 重新开辟内存。
== 与is 区别
==比较的是值是否一致
is 比较内存地址是否一致
```

&lt;img src=&#34;https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/700-20211222003346107.png&#34; alt=&#34;img&#34; style=&#34;zoom:50%;&#34; /&gt;

&lt;img src=&#34;https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/702.png&#34; alt=&#34;img&#34; style=&#34;zoom:50%;&#34; /&gt;

## 3. 总结

- go深拷贝， 就是拷贝值
- go浅拷贝， 拷贝引用
- go中赋值就能实现拷贝，针对引用类型（slice，map，channel）是浅拷贝，对值类型是深拷贝
- python **深浅拷贝针对可变类型的**
- python 深拷贝，拷贝是所有层引用
- python 浅拷贝，拷贝是第一层引用







---

> 作者: [Fred](https://github.com/ipfred)  
> URL: https://ipfred.github.io/lang/go/go_advanced/20250515180045/  

