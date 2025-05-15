# 4-3. 空结构体

## 空结构体

- 空结构体不包含任何字段`struct{}{}`

- 空结构体不占内存空间

  ```go
  package main
  
  import (
  	&#34;fmt&#34;
  	&#34;unsafe&#34;
  )
  
  func main() {
  	fmt.Println(unsafe.Sizeof(struct{}{}))  // 0
  }
  ```

  

## 空结构的作用

### 1. 与map结合实现set

- Go 语言标准库没有提供 Set 的实现，通常使用 map 来代替。事实上，对于集合来说，只需要 map 的键，而不需要值。即使是将值设置为 bool 类型，也会多占据 1 个字节，那假设 map 中有一百万条数据，就会浪费 1MB 的空间
- 将 map 作为集合(Set)使用时，可以将值类型定义为空结构体，仅作为占位符使用即可。

```go
type Set map[string]struct{}

func (s Set) Has(key string) bool {
	_, ok := s[key]
	return ok
}

func (s Set) Add(key string) {
	s[key] = struct{}{}
}

func (s Set) Delete(key string) {
	delete(s, key)
}

func main() {
	s := make(Set)
	s.Add(&#34;Tom&#34;)
	s.Add(&#34;Sam&#34;)
	fmt.Println(s.Has(&#34;Tom&#34;))
	fmt.Println(s.Has(&#34;Jack&#34;))
}
```



### 2. 制造伪迭代器

```go
for range make([]struct{}, 100) {
  fmt.Println(&#34;迭代器&#34;)
}
```



### 3. 不发送数据的channel

```go
func worker(ch chan struct{}) {
	&lt;-ch
	fmt.Println(&#34;do something&#34;)
	close(ch)
}

func main() {
	ch := make(chan struct{})
	go worker(ch)
	ch &lt;- struct{}{}
}
```



### 4. 仅包含方法的结构体

```go
type Door struct{}

func (d Door) Open() {
	fmt.Println(&#34;Open the door&#34;)
}

func (d Door) Close() {
	fmt.Println(&#34;Close the door&#34;)
}
```



---

> 作者: [Fred](https://github.com/ipfred)  
> URL: https://ipfred.github.io/lang/go/go_base/20250515175106/  

