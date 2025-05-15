# 9. Socket编程

# 计算机网络基础

- 常见的两种架构:
  - C/S 	微信,qq,迅雷等需要安装客户端的应用.
    - client 客户端
    - serve 服务端
  - B/S   百度,知乎,博客园登不需要客户端,通过一个浏览器即可实现相关服务
    - browser 浏览器
    - server 服务端

# 协议

- server和client得到的内容都是二进制,所以每一位代表什么就需要事先规定好,再按照约定进行发送和解析,这个约定就是协议.

- **arp协议(重点)**
  - 地址解析协议，即ARP（Address Resolution Protocol），是根据IP地址获取物理地址的一个TCP/IP协议。
  - 由交换机完成:交换机**先广播再单播**完成通讯
  - arp协议:通过ip地址获取mac地址
  - 交换机通过arp协议识别一台机器

-  **IP协议**
   -  规定网络地址的协议叫ip协议
   -  规定网络地址的协议叫ip协议，它定义的地址称之为ip地址，广泛采用的v4版本即ipv4，它规定网络地址32位2进制表示范围0.0.0.0-255.255.255.255
      一个ip地址通常写成四段十进制数，例：172.16.10.1
   -  IP协议的作用主要有两个，一个是为每一台计算机分配IP地址，另一个是确定哪些地址在同一个子网络。

### TCP协议

- 可靠,速度慢,全双工通信

- 建立连接**三次握手**,断开连接**四次挥手**

- 建立起链接之后,发送每条消息都有回执,为了保证数据的完整性,还有重传机制

- 数据传输:有收必有发,收发必相等

- 长连接:会一直占用对方端口

- IO操作(input/output),IO操作的输入输出时相对内存来说

  - write-send (输出ouput)
  - read-recv (输入input)

  ```python
  #三次握手
  TCP是因特网中的传输层协议，使用三次握手协议建立连接。当主动方发出SYN连接请求后，等待对方回答SYN&#43;ACK[1]，并最终对对方的 SYN 执行 ACK 确认。这种建立连接的方法可以防止产生错误的连接。[1] 
  TCP三次握手的过程如下：
  客户端发送SYN（SEQ=x）报文给服务器端，进入SYN_SEND状态。
  服务器端收到SYN报文，回应一个SYN （SEQ=y）ACK(ACK=x&#43;1）报文，进入SYN_RECV状态。
  客户端收到服务器端的SYN报文，回应一个ACK(ACK=y&#43;1）报文，进入Established状态。
  三次握手完成，TCP客户端和服务器端成功地建立连接，可以开始传输数据了。
                          
  #四次挥手
  (1) 某个应用进程首先调用close，称该端执行“主动关闭”（active close）。该端的TCP于是发送一个FIN分节，表示数据发送完毕。
  (2) 接收到这个FIN的对端执行 “被动关闭”（passive close），这个FIN由TCP确认。
  注意：FIN的接收也作为一个文件结束符（end-of-file）传递给接收端应用进程，放在已排队等候该应用进程接收的任何其他数据之后，因为，FIN的接收意味着接收端应用进程在相应连接上再无额外数据可接收。
  (3) 一段时间后，接收到这个文件结束符的应用进程将调用close关闭它的套接字。这导致它的TCP也发送一个FIN。
  (4) 接收这个最终FIN的原发送端TCP（即执行主动关闭的那一端）确认这个FIN。[1] 
  既然每个方向都需要一个FIN和一个ACK，因此通常需要4个分节。
  ```

  ![image-20220706221143179](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/20220706221143.png)

  ![image-20210324215259267](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/image-20210324215259267.png)

### UDP协议

- 不需要建立连接,速度特别快,可能会丢消息.

### 小结(TCP/UDP重点)

- 应用场景
  - TCP:文件上传下载(邮件,网盘)
  - UDP:即时通讯(微信,qq)

- 传输文件长度:
  - TCP 长度无限
  - UDP 能够传输的数据航都是有限的,根据数据传递设备的设置有关系

# osi七层模型

- &#39;应表会传网数物&#39;

  ```也叫osi五层模型,专业七层,开发人员掌握五层模型,表示层会话层了解```

  - **应用层**:python代码
  - 表示层
  - 会话层
  - **传输层**:tcp协议 udp协议 端口
  - **网络层**:ipv4/ipv6协议
  - **数据链路层**:mac地址 arp协议
  - **物理层**:

  ![](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/20210324215659.png)

- 每层运行常见协议/物理设备

| tcp/ip五层  | 每层运行常见协议                | 每层运行常见物理设备   |
| ----------- | ------------------------------- | ---------------------- |
| 5应用层     | python代码/http/https/ftp/smtp/ |                        |
| 4传输层     | tcp/udp协议 端口                | 四层交换机/四层路由器  |
| 3网络层     | ipv4/ipv6协议                   | 三层路由器/三层交换机  |
| 2数据链路层 | mac地址/arp协议                 | 网卡/交换机/二层交换机 |
| 1物理层     |                                 |                        |

# socket

- 中文名字:套接字
- **Socket是应用层与传输层中间的抽象层，Socket帮助去组织拼接信息数据，以符合指定的协议。**
- 用来描述IP地址和端口,是一个通信的句柄,用来实现不同计算机之间的通信
- socket对于程序员来说,已经是网络操作的底层了

# 粘包

### 粘包概念:

- TCP粘包是指发送方发送的若干包数据到接收方接收时粘成一包，从接收缓冲区看，后一包数据的头紧接着前一包数据的尾。
- 粘包可能由发送方造成，也可能由接收方造成。
- 只有TCP有粘包现象，UDP永远不会粘包
- 粘包不一定会发生

### 粘包原因:

```所谓粘包问题主要还是因为接收方不知道消息之间的界限，不知道一次性提取多少字节的数据所造成的。```

- 发送端原因:  由于TCP协议本身的机制（面向连接的可靠地协议-三次握手机制）客户端与服务器会**维持一个连接**（Channel），数据在连接不断开的情况下，可以**持续不断地将多个数据包发往服务器**，但是如果发送的网络数据包太小，那么他本身会启用Nagle算法（可配置是否启用）**对较小的数据包进行合并**（基于此，TCP的网络延迟要UDP的高些）然后再发送（超时或者包大小足够）。那么这样的话，服务器在接收到消息（数据流）的时候就**无法区分哪些数据包是客户端自己分开发送的**，这样产生了粘包.
- 接收端原因:  服务器在接收到数据库后，**放到缓冲区**中，如果**消息没有被及时从缓存区取走**，下次在取数据的时候可能就会出现**一次取出多个数据包**的情况，造成粘包现象。

# tcp粘包解决办法

### python版

- 在每次使用tcp协议发送数据流时,在开头标记一个数据流长度信息,并固定该报文长度(自定义协议).在客户端接收数据时先接收该长度字节数据,判断客户端发送数据流长度,并只接收该长度字节数据,就可以实现拆包,完美解决tcp粘包问题.

```python
#struct模块
#该模块可以把一个类型，如数字，转成固定长度为4的bytes类型
import struct
res = struct.pack(&#39;i&#39;,12345)	#i表示整数int
print(res,len(res),type(res))  #长度是4

res2 = struct.pack(&#39;i&#39;,12345111)
print(res,len(res),type(res2))  #长度也是4

unpack_res =struct.unpack(&#39;i&#39;,res2)
print(unpack_res)  #(12345111,)
print(unpack_res[0]) #12345111

```

```python
###################客户端client###################
#!/usr/bin/env python
# -*- coding:utf-8 -*-
import socket
import struct
sock=socket.socket()	
sock.connect((&#39;127.0.0.1&#39;, 13459))	

content1=&#39;我好&#39;.encode(&#39;utf-8&#39;)		#要发送消息
content2=&#39;他也好&#39;.encode(&#39;utf-8&#39;)

con1_len=struct.pack(&#39;i&#39;,len(content1))	# 计算要发送消息(字节)的长度,并使用struct模块转化为长度为4的字节b&#39;\x06\x00\x00\x00&#39;
sock.send(con1_len)						#先把这个4字节的报文发送
sock.send(content1)						#发送内容

con2_len=struct.pack(&#39;i&#39;,len(content2))
sock.send(con2_len)
sock.send(content2)


sock.close()
```

```python
###################服务端server###################
#!/usr/bin/env python
# -*- coding:utf-8 -*-
import struct
import socket

sock = socket.socket()		#买手机
sock.bind((&#39;127.0.0.1&#39;, 13459))		#插卡
sock.listen(10)  	#开机(同时最大连接10)

conn, addr = sock.accept()		#(受)与cilent端connect(攻)对应.
msg = conn.recv(4)				#首先接收4个字节(4个字节由client端struct模块转化)
len_msg= struct.unpack(&#39;i&#39;,msg)	#struct模块读取报文,判断跟随数据长度.返回值是一个元祖(6,)
size_msg=len_msg[0]					#取值判断跟随数据长度
msg = conn.recv(size_msg)			#接收报文读取长度字节
print(msg.decode(&#39;utf-8&#39;))			#解码输出

msg=conn.recv(4)
len_msg=struct.unpack(&#39;i&#39;,msg)
size_msg=len_msg[0]
msg = conn.recv(size_msg)
print(msg.decode(&#39;utf-8&#39;))


conn.close()
sock.close()

```

**!重要struct模块转化与读取都是对字节进行操作**!

### Go

将消息长度转为`int32(len(msg))`,4个字节

- 协议代码

  ```go
  package proto
  
  import (
  	&#34;bufio&#34;
  	&#34;bytes&#34;
  	&#34;encoding/binary&#34;
  )
  
  func Encode(message string) ([]byte, error) {
  	// 读取消息的长度转换成int32类型（4字节）
  	var length = int32(len(message))
  	var pkg = new(bytes.Buffer)
  	// 写入消息头
  	err := binary.Write(pkg, binary.LittleEndian, length)
  	if err != nil {
  		return nil, err
  	}
  	// 写入包体
  	err = binary.Write(pkg, binary.LittleEndian, []byte(message))
  	if err != nil {
  		return nil, err
  	}
  	return pkg.Bytes(), nil
  }
  
  // 解码
  func Decode(reader *bufio.Reader) (string, error) {
  	// 读消息长度
  	lengthByte, _ := reader.Peek(4)
  	lengthBuff := bytes.NewBuffer(lengthByte)
  	var length int32
  	err := binary.Read(lengthBuff, binary.LittleEndian, &amp;length)
  	if err != nil {
  		return &#34;&#34;, err
  	}
  	// buffer返回缓冲中现有的可读的字节数
  	if int32(reader.Buffered()) &lt; length&#43;4 {
  		return &#34;&#34;, err
  	}
  	// 读取真正的数据
  	pack := make([]byte, int(4&#43;length))
  	_, err = reader.Read(pack)
  	if err != nil {
  		return &#34;&#34;, err
  	}
  	return string(pack[4:]), nil
  }
  
  
  
  ```

- server code

  ```go
  package main
  
  import (
  	&#34;20_tcp/03_粘包/proto&#34;
  	&#34;bufio&#34;
  	&#34;fmt&#34;
  	&#34;io&#34;
  	&#34;net&#34;
  )
  
  func main() {
  	// 本地端口启动服务
  	listener, err := net.Listen(&#34;tcp&#34;, &#34;localhost:20000&#34;)
  	if err != nil {
  		fmt.Println(&#34;服务器启动失败....&#34;, err)
  		return
  	}
  	fmt.Println(&#34;监听成功...&#34;)
  	// 等待连接
  	for {
  		conn, err := listener.Accept()
  		if err != nil {
  			fmt.Println(&#34;连接建立失败...&#34;, err)
  			break
  		}
  		fmt.Println(&#34;连接成功...&#34;)
  		go Process(conn)
  	}
  	// 通信
  }
  
  func Process(conn net.Conn) {
  	defer conn.Close()
  	reader := bufio.NewReader(conn)
  	for {
  		msg, err := proto.Decode(reader)
  
  		fmt.Println(&#34;收到消息：&#34;, msg)
  		if err == io.EOF {
  			return
  		}
  		if err != nil {
  			fmt.Println(&#34;decode失败，err:&#34;, err)
  		}
  	}
  }
  
  
  ```

- client

  ```go
  package main
  
  import (
  	&#34;20_tcp/03_粘包/proto&#34;
  	&#34;fmt&#34;
  	&#34;net&#34;
  )
  
  func main() {
  	conn, err := net.Dial(&#34;tcp&#34;, &#34;127.0.0.1:20000&#34;)
  	if err != nil {
  		fmt.Println(&#34;连接失败,err:&#34;, err)
  		return
  	}
  	defer conn.Close()
  	msg := &#34;hello socket&#34;
  	for i := 0; i &lt; 20; i&#43;&#43; {
  		// 调用协议编码协议
  		b, err := proto.Encode(msg)
  		if err != nil {
  			fmt.Println(&#34;Encode失败，err:&#34;, err)
  		}
  		conn.Write(b)
  		fmt.Println(&#34;发送成功...,msg：&#34;, b)
  	}
  }
  
  
  ```

  

# TCP代码实现

- golang中使用net包实现socker编程; net常用的函数:
  - Dial   拨号
  - Listen   监听
  - Accept   接受

### TCP客户端

- tcp客户端

- ```go
  package main
  
  import (
  	&#34;fmt&#34;
  	&#34;log&#34;
  	&#34;net&#34;
  )
  
  func main() {
  	// 尝试连接百度服务器
  	conn,err:= net.Dial(&#34;tcp&#34;,&#34;www.baidu.com:80&#34;)
  	if err!= nil {
  		fmt.Println(err)
  	}
  	defer conn.Close()
  	log.Println(&#34;连接成功&#34;)
  	
  	// 发送数据
  	conn.Write([]byte(&#34;test\n&#34;))
  	// 接收数据
  	buf:= make([]byte,10)
  	conn.Read(buf)
  	
  }
  ```

  

### TCP服务端

- 服务端

  ```go
  package main
  
  import (
  	&#34;fmt&#34;
  	&#34;log&#34;
  	&#34;net&#34;
  	&#34;time&#34;
  )
  
  func main() {
  	listener,err:= net.Listen(&#34;tcp&#34;,&#34;:80&#34;)
  	if err!=nil {
  		fmt.Println(err)
  	}
  	defer listener.Close()
  	log.Println(&#34;启动成功...&#34;)
  	// 阻塞等待客户端连接
  	conn,err:= listener.Accept()
  	// 设置连接超时时间
  	conn.SetDeadline(time.Now().Add(time.Second))
  	// 设置读取超时时间
  	conn.SetReadDeadline(time.Now().Add(time.Second))
  	// 设置写入超时时间
  	conn.SetWriteDeadline(time.Now().Add(time.Second))
  }
  
  ```

  

# UDP代码实现

### UDP客户端

- 与tcp类似; 将Dial第一个参数改为udp

  ```go
  package main
  
  import (
  	&#34;fmt&#34;
  	&#34;log&#34;
  	&#34;net&#34;
  )
  
  func main() {
  	// 尝试连接百度服务器
  	conn,err:= net.Dial(&#34;udp&#34;,&#34;www.baidu.com:80&#34;)
  	if err!= nil {
  		fmt.Println(err)
  	}
  	defer conn.Close()
  	log.Println(&#34;连接成功&#34;)
  	
  	// 发送数据
  	conn.Write([]byte(&#34;test\n&#34;))
  	// 接收数据
  	buf:= make([]byte,10)
  	conn.Read(buf)
    log.Println(string(buf))
  
  	
  }
  ```

  

### UDP服务端

- ```go
  package main
  
  import (
  	&#34;fmt&#34;
  	&#34;log&#34;
  	&#34;net&#34;
  )
  
  func checkErr(err error)  {
  	if err!=nil {
  		log.Println(err)
  	}
  }
  
  func main() {
  	// 创建一个UDP地址
  	udpaddr, err:= net.ResolveUDPAddr(&#34;udp4&#34;,&#34;:1234&#34;)
  	checkErr(err)
  	// 创建udp服务
  	conn,err:= net.ListenUDP(&#34;udp&#34;,udpaddr)
  	checkErr(err)
  	defer conn.Close()
  	fmt.Println(&#34;UDP服务创建成功&#34;)
  
  	buf:= make([]byte,1024)
  	conn.Read(buf)
  	log.Println(string(buf))
  	
  	_,raddr,err:= conn.ReadFromUDP(buf)
  	
  	conn.Write([]byte(&#34;hello word\r\n&#34;))
  	conn.WriteToUDP([]byte(&#34;hello word\r\n&#34;),raddr)
  
  }
  
  ```

  

---

> 作者: [Fred](https://github.com/ipfred)  
> URL: https://ipfred.github.io/lang/go/go_base/20250515175134/  

