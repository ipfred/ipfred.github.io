# 1-0. 环境配置


# Golang - 环境配置

## 认识go语言

-  go语言（或 Golang）是Google开发的开源编程语言，诞生于2006年1月2日下午15点4分5秒，于2009年11月开源，2012年发布go稳定版

- go是非常年轻的一门语言，它的主要目标是“兼具Python 等动态语言的开发速度和C/C&#43;&#43;等编译型语言的性能与安全性

- 可以粗略的认为go = c &#43; python

  1. 从C语言中继承了很多理念，尤其是指针的运用

  ```go
  func testPtr(num *int) {
    *num = 20
  }
  ```

  1. 引入包的概念，用于组织程序结构。单独的go文件必须存在于package中
  2. 垃圾回收制度，内存自动回收。
  3. 语言层面天然支持高并发
     使用了`goroutine`的语法，轻量级线程，高效利用多核
  4. 吸收了管道通信机制，有管道的写法，`channel`的写法，通过管道实现不同goroute之间的相互通信。

## go的优势

- 做高并发有巨大的优势

- 垃圾自动回收

- 开发简单，开发效率堪比python

- 运行效率高，很适合用作中央服务器的系统编程语言

- 是项目转型的首选语言，很多公司在用go重构代码

- 提供了海量并行的支持，很适合处理游戏相关数据

## 环境准备

1. 安装golang jdk --中文在线文档：https://studygolang.com/pkgdoc
2. 选择安装,一路next....

3. 配置环境变量(可选)

4. 安装开发工具

## 创建第一个项目

- GOPATH：go的代码存这里面
  - src: 存放源代码
  - pkg:存一些中间文件
  - bin:存放二进制文件的目录

```go
// 单行注释
/*多行
注释*/

 // 每个go源代码文件开头必须package声明，代表属于哪个包
 // 声明main，代表可以编译
 // 下面会对应有且仅有一个main函数，主函数
package main

// 引入的包，必须用，不然报错
// 自动导包
import (
   &#34;fmt&#34;
)

// 左括号不能单起一行，会报错
// 同一个包中，只有一个main
func main() {
   fmt.Println(&#34;hello word&#34;)
}
```

## 常见的go命令

```go
go run test.go // 执行go文件，相当于 先编译生成可执行文件后接着执行它

go build test.go //编译生成可执行文件
//注意：windows平台生成exe,但是mac就是一个test,无后缀，使用方式./test即可

go run 运行当个.go文件
go install 在编译源代码之后还安装到指定的目录
go build 加上可编译的go源文件可以得到一个可执行文件
go get = git clone &#43; go install 从指定源上面下载或者更新指定的代码和依赖，并对他们进行编译和安装
```

两者的三个主要区别：

1. go bulid编译后生成的可执行文件，可以直接拿到同平台下但是没有go开发环境的机器下运行。
2. go run不会编译生成可执行文件，不能在没有go开发环境下的其它机器上运行。
3. .go的源文件虽然很小，但是编译后会将源代码所依赖的库文件编译进来，导致生成的可执行文件很大。

关于编译的几个参数：

1. 编译时可以改变生成的可执行文件的名字
   `go build -o hello-new hello.go` 可执行文件就会叫hello-new

## 在一个平台下生成多个平台运行包

&gt; 编译之后直接可执行，使用起来非常方便

1. Mac
   Mac下编译Linux, Windows平台的64位可执行程序：

   CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build test.go
   CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build test.go

2. Linux
   Linux下编译Mac, Windows平台的64位可执行程序：

   CGO_ENABLED=0 GOOS=darwin GOARCH=amd64 go build test.go
   CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build test.go

3. Windows
   Windows下编译Mac, Linux平台的64位可执行程序：

   ```sh
   SET CGO_ENABLED=0
   SET GOOS=darwin3
   SET GOARCH=amd64
   go build main.go
   SET CGO_ENABLED=0
   SET GOOS=linux
   SET GOARCH=amd64
   go build main.go
   ```

   

&gt; GOOS：目标可执行程序运行操作系统，支持 darwin，freebsd，linux，windows
&gt; GOARCH：目标可执行程序操作系统构架，包括 386，amd64，arm

## Golang的项目目录结构

- 个人

![image-20210228114456941](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/image-20210228114456941.png)

- 公司

  ![image-20210228114609916](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/image-20210228114609916.png)

---

> 作者: [Fred](https://github.com/ipfred)  
> URL: https://ipfred.github.io/lang/go/go_base/20250515174347/  

