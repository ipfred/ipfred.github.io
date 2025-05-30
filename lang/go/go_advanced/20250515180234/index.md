# 12-1. unsafe.Pointer和 uintptr

## 区别

- `unsafe.Pointer`只是单纯的通用指针类型，用于转换不同类型指针，它不可以参与指针运算；
- 而`uintptr`是用于指针运算的，GC 不把 uintptr 当指针，也就是说 uintptr 无法持有对象， uintptr 类型的目标会被回收；
- `unsafe.Pointer` 可以和 普通指针 进行相互转换；
- `unsafe.Pointer` 可以和 uintptr 进行相互转换。

## 示例

- 通过一个例子加深理解，接下来尝试用指针的方式给结构体赋值。

```go
package main

import (
 &#34;fmt&#34;
 &#34;unsafe&#34;
)

type W struct {
 b int32
 c int64
}

func main() {
 var w *W = new(W)
 //这时w的变量打印出来都是默认值0，0
 fmt.Println(w.b,w.c)

 //现在我们通过指针运算给b变量赋值为10
 b := unsafe.Pointer(uintptr(unsafe.Pointer(w)) &#43; unsafe.Offsetof(w.b))
 *((*int)(b)) = 10
 //此时结果就变成了10，0
 fmt.Println(w.b,w.c)
}
```

- `uintptr(unsafe.Pointer(w))` 获取了 `w` 的指针`起始值`
- `unsafe.Offsetof(w.b)` 获取 `b` 变量的`偏移量`
- 两个`相加`就得到了 `b` 的`地址值`，将通用指针 `Pointer` 转换成具体指针 `((*int)(b))`，通过 `*` 符号取值，然后赋值。`*((*int)(b))` 相当于把 `(*int)(b)` 转换成 `int` 了，最后对变量重新赋值成 `10`，这样指针运算就完成了。

---

> 作者: [Fred](https://github.com/ipfred)  
> URL: https://ipfred.github.io/lang/go/go_advanced/20250515180234/  

