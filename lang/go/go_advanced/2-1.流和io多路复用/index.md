# 

# 一. 流, I/O操作, 阻塞

## 1. 流

- 可以进行I/O操作的内核对象
- 文件、管道、套接字……
- 流的入口：文件描述符(fd)

## 2. I/O操作

- 所有对流的读写操作，我们都可以称之为IO操作。

- 当一个流中， 在没有数据read的时候，或者说在流中已经写满了数据，再write，我们的IO操作就会出现一种现象，就是阻塞现象，如下图。

&lt;img src=&#34;https://img.kancloud.cn/18/de/18de4271dbfbd3c5cf0193fb60e8c5b7_832x210.png&#34; alt=&#34;img&#34; style=&#34;zoom:50%;&#34; /&gt;

&lt;img src=&#34;https://img.kancloud.cn/a5/58/a558f826d4c1e0872aaa888306cf05f0_854x244.png&#34; alt=&#34;img&#34; style=&#34;zoom:50%;&#34; /&gt;

## 3. 阻塞

&lt;img src=&#34;https://img.kancloud.cn/8b/74/8b74ba71a1e5cdae8a994712b8a85e99_755x564.png&#34; alt=&#34;img&#34; style=&#34;zoom:33%;&#34; /&gt;

 **阻塞场景**: 你有一份快递，家里有个座机，快递到了主动给你打电话，期间你可以休息。
&lt;img src=&#34;https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/af25125ccf72dc6a288c6cb5e06f240c_757x562-20211203110017276.png&#34; alt=&#34;img&#34; style=&#34;zoom:33%;&#34; /&gt;

**非阻塞，忙轮询场景**: 你性子比较急躁， 每分钟就要打电话询问快递小哥一次， 到底有没有到，快递员接你电话要停止运输，这样很耽误快递小哥的运输速度。

- 阻塞等待

空出大脑可以安心睡觉, 不影响快递员工作（不占用CPU宝贵的时间片）。

- 非阻塞，忙轮询

浪费时间，浪费电话费，占用快递员时间（占用CPU，系统资源）。

很明显，阻塞等待这种方式，对于通信上是有明显优势的， 那么它有哪些弊端呢？

# 二. IO多路复用

## 1. 阻塞IO

- 就像 socket 通信一样, 连接的客户端一直不发数据，那么服务端线程将会一直阻塞在 read 函数上不返回，也无法接受其他客户端连接。就会发生IO阻塞

  &lt;img src=&#34;https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/1638502588418.gif&#34; alt=&#34;1638502588418&#34; style=&#34;zoom: 67%;&#34; /&gt;

## 2. 非阻塞 IO

- **恳请操作系统为我们提供一个非阻塞的 read 函数**。通过轮询或者回调的方法实现

  &lt;img src=&#34;https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/1638502577323.gif&#34; alt=&#34;1638502577323&#34; style=&#34;zoom: 67%;&#34; /&gt;

## 3. IO多路复用

&gt; doc: https://mp.weixin.qq.com/s/kebjG5UosHmXa7AKCatSrA



- 实现IO多路复用的方法:
  - select : 通过遍历(系统遍历)的方式实现, select 只能监听 1024 个文件描述符. windows,linux
  - poll: 通过遍历(系统遍历)的方式实现, 不再有文件描述符数量限制. linux
  - epoll: 通过回调的方式实现IO多路复用, 没有文件描述符限制, 效率最高. linux

### 3.1 select

&gt;1. 系统遍历
&gt;2. select 调用需要传入 fd 数组，需要拷贝一份到内核，高并发场景下这样的拷贝消耗的资源是惊人的。（可优化为不复制）
&gt;
&gt;2. select 在内核层仍然是通过遍历的方式检查文件描述符的就绪状态，是个同步过程，只不过无系统调用切换上下文的开销。（内核层可优化为异步事件通知）
&gt;
&gt;3. select 仅仅返回可读文件描述符的个数，具体哪个可读还是要用户自己遍历。（可优化为只返回给用户就绪的文件描述符，无需用户做无效的遍历）

&lt;img src=&#34;https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/1638502585301.gif&#34; alt=&#34;1638502585301&#34; style=&#34;zoom:50%;&#34; /&gt;



&lt;img src=&#34;https://img.kancloud.cn/ee/43/ee430296183245bb677144388a458f5e_675x410.png&#34; alt=&#34;img&#34; style=&#34;zoom: 50%;&#34; /&gt;

- 我们可以开设一个代收网点，让快递员全部送到代收点。这个网店管理员叫select。这样我们就可以在家休息了，麻烦的事交给select就好了。当有快递的时候，select负责给我们打电话，期间在家休息睡觉就好了。
- 但select 代收员比较懒，她记不住快递员的单号，还有快递货物的数量。她只会告诉你快递到了，但是是谁到的，你需要挨个快递员问一遍。

### 3.2 poll

&gt;- 系统遍历
&gt;
&gt;- poll 也是操作系统提供的系统调用函数。
&gt;- 它和 select 的主要区别就是，去掉了 select 只能监听 1024 个文件描述符的限制。

### 3.2 epoll

&gt;1. 回调(异步 IO 事件唤醒)
&gt;2. 内核中保存一份文件描述符集合，无需用户每次都重新传入，只需告诉内核修改的部分即可。
&gt;
&gt;2. 内核不再通过轮询的方式找到就绪的文件描述符，而是通过异步 IO 事件唤醒。
&gt;
&gt;3. 内核仅会将有 IO 事件的文件描述符返回给用户，用户也无需遍历整个文件描述符集合。

&lt;img src=&#34;https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/1638502586963.gif&#34; alt=&#34;1638502586963&#34; style=&#34;zoom:67%;&#34; /&gt;



# 三.底层原理

## 1 单进程单socket的弊端

- 服务端需要管理多个客户端连接，而 `recv()` 系统调用只能监视单个socket。
- 这种情况下，如果要管理多个客户端连接，就需要多开进程或线程，每个进程维护一个socket套接字，没有网络数据时进程阻塞在recv()系统调用上，当网络数据到达时，操作系统环境对应socket等待队列上的进程。
  此时面临的问题是维护进程或线程带来的系统开销（每个线程的栈空间8M，由于系统的内存资源有限，1K个线程就已经需要消耗8G内存，不可能无限制的多开线程，且进程、线程间的频繁切换也会带来较大的开销）。

## 2 select的设计思想

- 最先想到的办法是使用一个进程监视多个socket，预先传入一个socket列表，如果列表中的socket都没有数据，则进程继续挂起；直到有一个或以上的socket接收到网络数据，再唤醒进程。这种方法很直接，这也是select的设计思想。

### 2.1 select的执行流程

- 如下图，假设进程A同时监听 sock1、sock2、sock3（通过fd_set传入），那么，在调用select之后，操作系统会把进程A分别加入到这三个socket的等待队列中：

![请添加图片描述](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/20220728200711.jpeg)

- 当任何一个socket上收到数据时，中断程序将唤起进程。所谓唤起进程，就是将其从所有的socket对象的等待队列中移除，并插入到就绪队列中。

  ![请添加图片描述](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/20220728200747.jpeg)

  ![请添加图片描述](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/20220728200753.jpeg)

### 2.2 select的缺点

1. 每次调用select都需要将进程加入到所有socket对象的等待队列中，每次唤醒进程又要将进程从所有socket对象的等待队列中移除。这里涉及到对socket列表的两次遍历，而且每次都要将整个fds列表传递给内核，有一定的开销。正因为遍历操作开销大，出于效率的考量，才会规定select的最大监视数量，默认只能监视1024个socket（强行修改也是可以的）；
2. 进程被唤醒后，程序并不知道socket列表中的那些socket上收到数据，因此在用户空间内需要对socket列表再做一次遍历。

## 3 epoll的设计思想

### 3.1 epoll的实现原理和流程

1. **创建epoll对象：**

   - 当进程调用 `epoll_create` 方法时，内核会创建一个 `eventpoll` 对象，也就是应用程序中的 epfd 所代表的对象。`eventpoll` 对象也是文件系统中的一员（Linux中一切设备皆文件），和socket一样也拥有一个“等待队列”。

   ![请添加图片描述](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/20220728200953.jpeg)

2. **维护监视列表：**

   - 创建epoll对象 `eventpoll` 之后，可以使用 `epoll_ctl` 添加或者删除所要监听的socket。以添加socket为例，如果要对sock1、sock2、sock3进行监视，内核会将`eventpoll`添加到这三个socket的等待队列中。

   - 当socket收到数据后，中断回调程序会操作eventpoll对象，而不是直接操作进程。

   ![请添加图片描述](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/20220728200946.jpeg)

3. **接收数据：**

   - select的低效原因之一在于应用程序不知道哪些socket收到数据，只能一个个的遍历。如果内核维护一个“就绪列表”，在就绪列表中引用收到数据的socket，就能避免遍历。

   - 在 `eventpoll` 对象中就实现了这样的一个“就绪列表” ---- `rdlist`。

   - 当socket收到数据，中断回调程序会给`eventpoll`的“就绪列表”添加socket的引用，如下图所示：

   ![请添加图片描述](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/20220728201026.jpeg)

   - eventpoll对象相当于是socket和进程之间的中介，socket的数据接收并不直接影响进程，而是通过改变eventpoll的就绪列表来改变进程状态。

   - epoll_wait的返回条件也是根据rdlist的状态进行判断：
     1. 如果rdlist已经引用了socket，那么epoll_wait直接返回；
     2. 如果rdlist为空，阻塞进程。

4. 阻塞和唤醒进程：

   - 如下图，假设当进程A运行到epoll_wait()时，操作系统会将进程A放入到eventpoll对象的等待队列中，阻塞进程。（对于epoll，操作系统只需要将进程A放入eventpoll这一个对象的等待队列中；而对于select，操作系统则需要将进程A放入到socket列表中的所有socket对象的等待队列中。）

     ![请添加图片描述](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/20220728201646.jpeg)

   - 当socket接收到数据时，中断回调程序一方面修改rdlist“就绪列表”，另一方面唤醒eventpoll等待队列中的进程A。
     也因为rdlist就绪列表的存在，进程A可以在重新进入运行态后准确知道哪些socket上发生了变化。

     ![请添加图片描述](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/20220728201755.jpeg)



### 3.2 epoll高效的原因

- 相较于select，epoll实现高效主要基于以下两点：
  1. 功能分离；
  2. 就绪列表。

- select低效的原因之一是将“维护等待队列”和“阻塞进程”两个步骤合二为一。
  - 每次调用select都需要这两步操作，然而大多数应用场景中，需要监视的socket相对固定，并不需要每次修改。

- epoll将这两个操作分开，先用 `epoll_ctl` 维护等待队列，再调用 `epoll_wait` 阻塞进程，以此来提高效率。

&lt;img src=&#34;https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/20220728202010.jpeg&#34; alt=&#34;请添加图片描述&#34; style=&#34;zoom:33%;&#34; /&gt;

- select低效的另一个原因在于程序不知道哪些socket收到数据，只能一个一个的遍历。如果内核维护一个“就绪列表”，引用收到的数据的socket，就能避免遍历。

  ![请添加图片描述](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/20220728202139.jpeg)



### 3.3 epoll的实现细节

#### a. eventpoll对象的数据结构

- eventpoll对象包含了：lock、mtx、wq（等待队列）、rdlist 等成员。

&lt;img src=&#34;https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/20220728202225.jpeg&#34; alt=&#34;请添加图片描述&#34; style=&#34;zoom: 67%;&#34; /&gt;

#### b. rdlist就绪队列的数据结构：

- epoll使用双向链表来实现就绪队列rdlist。

#### c. 索引结构：

- epoll使用红黑树作为索引结构，以便于快速的插入和删除要监视的socket套接字。
- 红黑树时一种自平衡的二叉查找树，搜索、插入、删除的时间复杂度都是O(logN)。

# 四. CPU密集型和IO密集型

## **CPU密集型**

CPU密集型也叫计算密集型，指的是系统的硬盘、内存性能相对CPU要好很多，此时，系统运作CPU读写IO(硬盘/内存)时，IO可以在很短的时间内完成，而CPU还有许多运算要处理，因此，CPU负载很高。

CPU密集表示该任务需要大量的运算，而没有阻塞，CPU一直全速运行。CPU密集任务只有在真正的多核CPU上才可能得到加速（通过多线程），而在单核CPU上，无论你开几个模拟的多线程该任务都不可能得到加速，因为CPU总的运算能力就只有这么多。

CPU使用率较高（例如:计算圆周率、对视频进行高清解码、矩阵运算等情况）的情况下，通常，线程数只需要设置为CPU核心数的线程个数就可以了。 这一情况多出现在一些业务复杂的计算和逻辑处理过程中。比如说，现在的一些机器学习和深度学习的模型训练和推理任务，包含了大量的矩阵运算。

## **IO密集型**

IO密集型指的是系统的CPU性能相对硬盘、内存要好很多，此时，系统运作，大部分的状况是CPU在等IO (硬盘/内存) 的读写操作，因此，CPU负载并不高。

密集型的程序一般在达到性能极限时，CPU占用率仍然较低。这可能是因为任务本身需要大量I/O操作，而程序的逻辑做得不是很好，没有充分利用处理器能力。

CPU 使用率较低，程序中会存在大量的 I/O 操作占用时间，导致线程空余时间很多，通常就需要开CPU核心数数倍的线程。

其计算公式为：**IO密集型核心线程数 = CPU核数 / （1-阻塞系数）**。

当线程进行 I/O 操作 CPU 空闲时，启用其他线程继续使用 CPU，以提高 CPU 的使用率。例如：数据库交互，文件上传下载，网络传输等。

## **CPU密集型与IO密集型任务的使用说明**

- 当线程等待时间所占比例越高，需要越多线程，启用其他线程继续使用CPU，以此提高CPU的利用率；
- 当线程CPU时间所占比例越高，需要越少的线程，通常线程数和CPU核数一致即可，这一类型在开发中主要出现在一些计算业务频繁的逻辑中。

## **CPU密集型任务与IO密集型任务的区别**

计算密集型任务的特点是要进行大量的计算，消耗CPU资源，全靠CPU的运算能力。这种计算密集型任务虽然也可以用多任务完成，但是任务越多，花在任务切换的时间就越多，CPU执行任务的效率就越低，所以，要最高效地利用CPU，计算密集型任务同时进行的数量应当等于CPU的核心数，避免线程或进程的切换。

&gt; 计算密集型任务由于主要消耗CPU资源，因此，代码运行效率至关重要。Python这样的脚本语言运行效率很低，完全不适合计算密集型任务。对于计算密集型任务，最好用C语言编写。

IO密集型任务的特点是CPU消耗很少，任务的大部分时间都在等待IO操作完成（因为IO的速度远远低于CPU和内存的速度）。涉及到网络、磁盘IO的任务都是IO密集型任务，

对于IO密集型任务，线程数越多，CPU效率越高，但也有一个限度。

## go适合的IO密集型

- 推荐博文：https://mp.weixin.qq.com/s/eSlv61fR61SQTt3iTgzbPQ

##  **总结**

1. 一个计算为主的应用程序（CPU密集型程序），多线程或多进程跑的时候，可以充分利用起所有的 CPU 核心数，比如说16核的CPU ，开16个线程的时候，可以同时跑16个线程的运算任务，此时是最大效率。但是如果线程数/进程数远远超出 CPU 核心数量，反而会使得任务效率下降，因为**频繁的切换线程或进程**也是要消耗时间的。因此对于 CPU 密集型的任务来说，线程数/进程数等于 CPU 数是最好的了。
2. 如果是一个磁盘或网络为主的应用程序（IO密集型程序），一个线程处在 IO 等待的时候，另一个线程还可以在 CPU 里面跑，有时候 CPU 闲着没事干，所有的线程都在等着 IO，这时候他们就是同时的了，而单线程的话，此时还是在一个一个等待的。我们都知道IO的速度比起 CPU 来是很慢的。此时线程数可以是CPU核心数的数倍（视情况而定）。



---

> 作者:   
> URL: http://localhost:1313/lang/go/go_advanced/2-1.%E6%B5%81%E5%92%8Cio%E5%A4%9A%E8%B7%AF%E5%A4%8D%E7%94%A8/  

