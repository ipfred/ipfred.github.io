# 

## 什么是netgo

&gt; Go语言的网络库是基于操作系统提供的系统调用(syscall)实现的。在大多数现代操作系统上，这些系统调用都是由C语言实现的，经过高度优化，性能非常好。但在某些特殊的架构或操作系统上，系统网络调用可能无法正常工作，或者效率较低。
&gt;
&gt; 为了解决这个可移植性问题，Go语言还提供了一个纯Go实现的网络库。这个网络库不依赖于操作系统的系统调用，而是使用Go语言的运行时(runtime)来处理网络通信。由于Go运行时本身就是跨平台的，因此纯Go网络库也就获得了良好的可移植性。

- `netgo`是Go编译器的一个命令行标志，它控制编译产生的可执行文件所使用的网络库实现。Go语言的网络库默认是基于操作系统提供的系统调用(syscall)实现的，但也提供了一个纯Go语言实现的网络库作为可选方案，`netgo`标志就是用来选择使用哪种网络库实现。
- 不使用`netgo`标志时，编译器会链接并使用操作系统提供的系统网络调用库，这通常可以获得更好的网络性能。
- 使用`netgo`标志(`go build -netgo`)时，编译器会链接纯Go实现的网络库，虽然性能可能会略低，但提高了可移植性。

## netgo使用场景

- 一般来说，只有在以下情况下才需要使用`-netgo`标志:
  1. **目标系统不支持Go默认网络库**:某些特殊的架构或操作系统可能无法正常支持Go语言默认使用的基于系统调用的网络库实现，此时需要使用纯Go网络库来获得可移植性。比如我们项目从arm架构适配申威架构，出现网络调用程序panic，需要使用netgo；
  2. **需要跨平台可移植性**:如果你需要在多个操作系统平台上运行你的Go程序，而这些平台的系统网络调用存在不兼容的情况，使用纯Go网络库就可以提高可移植性。

- 真实生产环境panic报错部分信息，最后通过使用netgo方式解决

  &gt; 系统架构 sw64 linux

  ```
  fatal error: unexpected signal during runtime execution
  [signal SIGSEGV: segmentation violation code=0x1 addr=0x46 pc=0x416263ca3ea4]
  
  runtime stack:
  runtime.throw({0x121da53a3, 0x2a})
          /go/src/runtime/panic.go:992 &#43;0x94
  runtime.sigpanic()
          /go/src/runtime/signal_unix.go:802 &#43;0x4e0
  
  goroutine 1233 [syscall]:
  runtime.cgocall(0x1218d90b0, 0xc00be65c8)
          /go/src/runtime/cgocall.go:157 &#43;0x60 fp=0xc00be6598 sp=0xc00be6568 pc=0x120005190
  net._C2func_getaddrinfo(0xc003cc870, 0x0, 0xc00c457a0, 0xc01686180)
          _cgo_gotypes.go:94 &#43;0x8c fp=0xc00be65c0 sp=0xc00be6598 pc=0x1202c825c
  net.cgoLookupIPCNAME.func1({0xc003cc870, 0x2f, 0x2f}, 0xc00c457a0, 0xc01686180)
          /go/src/net/cgo_unix.go:160 &#43;0x140 fp=0xc00be6600 sp=0xc00be65c0 pc=0x1202caee0
  net.cgoLookupIPCNAME({0x121d544ba, 0x3}, {0xc004be540, 0x2e})
          /go/src/net/cgo_unix.go:160 &#43;0x210 fp=0xc00be6710 sp=0xc00be6600 pc=0x1202ca420
  net.cgoIPLookup(0xc02494720, {0x121d544ba, 0x3}, {0xc004be540, 0x2e})
          /go/src/net/cgo_unix.go:217 &#43;0x8c fp=0xc00be67a8 sp=0xc00be6710 pc=0x1202cafcc
  net.cgoLookupIP.func1()
          /go/src/net/cgo_unix.go:227 &#43;0xac fp=0xc00be67d8 sp=0xc00be67a8 pc=0x1202cb62c
  runtime.goexit()
          /go/src/runtime/asm_sw64.s:363 &#43;0x4 fp=0xc00be67d8 sp=0xc00be67d8 pc=0x1200b9c54
  created by net.cgoLookupIP
          /go/src/net/cgo_unix.go:227 &#43;0x208
  ```

  

## netgo使用方式

- 在Go命令中使用`-netgo`标志非常简单，只需在`go build`或`go install`等编译命令中加上该标志; 加上`-netgo`标志后，编译器会自动链接纯Go实现的网络库，而不是默认的基于系统调用的网络库实现

  ```sh
  go build -netgo  xxx
  ```

---

> 作者:   
> URL: http://localhost:1313/lang/go/go_advanced/21.go%E7%BC%96%E8%AF%91%E5%8F%82%E6%95%B0netgo/  

