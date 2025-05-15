# 1-2.Go垃圾回收机制

[TOC]

# 一. 栈和堆

&gt;- **栈内存:** 栈内存首先是一片内存区域，存储的都是局部变量，凡是定义在方法中的都是局部变量（方法外的是全局变量），for循环内部定义的也是局部变量，是先加载函数才能进行局部变量的定义，所以方法先进栈，然后再定义变量，变量有自己的作用域，一旦离开作用域，变量就会被释放。栈内存的更新速度很快，因为局部变量的生命周期都很短。但是可分配的内存有限
&gt;
&gt;- **堆内存:** 存储的是数组和对象（其实数组就是对象），凡是new建立的都是在堆中，堆中存放的都是实体（对象），实体用于封装数据，而且是封装多个（实体的多个属性），如果一个数据消失，这个实体也没有消失，还可以用，所以堆是不会随时释放的，但是栈不一样，栈里存放的都是单个变量，变量被释放了，那就没有了。堆里的实体虽然不会被释放，但是会被当成垃圾通过GC回收.

&lt;img src=&#34;https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/image-20211202100041939.png&#34; alt=&#34;image-20211202100041939&#34; style=&#34;zoom:33%;&#34; /&gt;

## 1.原理

- 在main函数中` var arr = make([]int,9)`的定义流程:

  1. 主函数先进栈，在栈中定义一个变量arr,接下来为arr赋值，但是右边不是一个具体值，是一个实体。实体创建在堆里，在堆里首先通过new关键字开辟一个空间，内存在存储数据的时候都是通过地址来体现的，地址是一块连续的二进制，然后给这个实体分配一个内存地址。数组都是有一个索引，数组这个实体在堆内存中产生之后每一个空间都会进行默认的初始化（**这是堆内存的特点，未初始化的数据是不能用的**），不同的类型初始化的值不一样。所以堆和栈里就创建了变量和实体：

     ![image-20211202095343255](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/image-20211202095343255.png)

  2. 给堆分配了一个地址，把堆的地址赋给arr，arr就通过地址指向了数组。所以arr想操纵数组时，就通过地址，而不是直接把实体都赋给它。这种我们不再叫他基本数据类型，而叫引用数据类型。称为arr引用了堆内存当中的实体。(指针)

     ![image-20211202095324746](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/image-20211202095324746.png)

- 如果当int [] arr=null;arr不做任何指向，null的作用就是取消引用数据类型的指向。

## 2.区别

1. 栈内存存储的是局部变量而堆内存存储的是实体；

2. 栈内存的更新速度要快于堆内存，因为局部变量的生命周期很短；

3. 栈内存存放的变量生命周期一旦结束就会被释放，而堆内存存放的实体会被垃圾回收机制不定时的回收。      

# 二. python 垃圾回收

&gt; 书籍: 《python源码剖析》

## 1. python内存管理

- python中将所有的数据类型分为了两种；分别是:由多个元素组成的和单个元素组成的；以利用不同的结构体去区分，
  分别是：pyobject结构体（_ PyObject_HEAD_EXTRA 双向链表;ob_refcnt;引用计数器；_typeobject *ob_type 表示对象类型），
  一个是pyvarobject结构体(PyObject ob_base 内部包含pyobject结构体； obsize；此对象有多少元素组成 )
- 在pytho代码中，如果创建对象或者是对对象赋值，内存中会对对象做两种操作：将对象加入双向链表，引用计数加1；
- 如果执行对象删除操作，也会进行两步操作：引用计数器减一；如果引用计数为0，就将对象从链表中剔除；

## 2. python 垃圾回收

- 垃圾回收机制是以引用计数为主，以分代回收和标记清楚为辅；

  - 引用计数

    ```python
    在pytho代码中，如果创建对象或者是对对象赋值，内存中会对对象做两种操作：将对象加入双向链表，引用计数加1；如果执行对象删除操作，也会进行两步操作：引用计数器减一；如果引用计数为0，就将对象从链表中剔除；
    ```

  - **标记清除：引用计数可以满足基本的内存管理和垃圾回收，但是无法解决&#34;循环引用&#34;的问题,所以存在标记清除;**只有多个元素组成的才会产生循环引用；

    ```python
    # 循环引用
    v1= [1,2]
    v2=[3,4]
    v1.append(v2)
    v2.append(v1)
    # 在python内部维护了两个双向链表，一个单个元素组成，一个是多个元素组成的；在垃圾回收机制（GC）中会定期扫描由多个元素组成的链表，如果发现有循环引用存在，那么引用计数分别减一；
    ```

  - 分代回收：

    ```python
    分代回收：在python中为由多个元素的组成的类型（可能存在循环引用），为这些元素维护了三个双向链表，分别称为0代1代2代；python中为这三个链表设置了一个阈值，分别是700，10，10；（参数说明：如果第0代的链表中有700个对象时，进行一次扫描；0代扫描十次，一代扫描一次；一代扫描十次，二代扫描一次；极大的减少了扫描元素的个数）
    0代链表中item的个数达到700时进行十次扫描，标记清除，引用计数为0的从双向链表中删除；
    ```

    

# 三. Golang 垃圾回收



## 1. Go-v1.3 标记清除

- 标记-清除(mark and sweep)算法

  1. **第一步**, 标记

     暂停程序业务逻辑, 分类出可达和不可达的对象，然后做上标记。如下如, 目前程序可达对象仅1,2,3,4,7五个对象, 对这五个对象进行标记(mark)

     ![image-20220630100704197](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/20220630100709.png)

     

  2. **第二步**,需要程序暂停, 然后清除`对象5`和`对象6`, 程序会暂定停止任何工作，卡在那等待回收执行完毕。

  3. **第三部**, 停止暂停，让程序继续跑。然后循环重复这个过程，直到process程序生命周期结束。

### 1.1 缺点

- STW，stop the world；让程序暂停，程序出现卡顿 **(重要问题)**；
- 标记需要扫描整个heap；
- 清除数据会产生heap碎片。

![image-20211201171747329](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/image-20211201171747329.png)

## 2. Go-v1.5 三色标记&#43;屏障机制

- 标记清除最大的缺点就是存在 STW需要程序暂停才能进行垃圾回收, 造成性能问题. v1.5使用三色标记&#43;屏障机制 解决STW问题

### 2.1 三色标记

##### 2.1.1 原理

1. **第一步** , 每次新创建的对象，默认的颜色都是标记为“白色”，如图所示。

   ![image-20211201173055909](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/image-20211201173055909.png)

2. **第二步**, 每次GC回收开始, 会从根节点开始遍历所有对象，把遍历到的对象从白色集合放入“灰色”集合如图所示。

   &gt;要注意的是，本次遍历是一次遍历，非递归形式，是从程序抽次可抵达的对象遍历一层，如上图所示，当前可抵达的对象是对象1和对象4，那么自然本轮遍历结束，对象1和对象4就会被标记为灰色，灰色标记表就会多出这两个对象。

   &lt;img src=&#34;https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/image-20211201173158570.png&#34; alt=&#34;image-20211201173158570&#34; style=&#34;zoom:50%;&#34; /&gt;

3. **第三步**, 遍历灰色集合，将灰色对象引用的对象从白色集合放入灰色集合，之后将此灰色对象放入黑色集合，如图所示。

   &lt;img src=&#34;https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/image-20211201173250172.png&#34; alt=&#34;image-20211201173250172&#34; style=&#34;zoom:45%;&#34; /&gt;

4. **第四步**, 重复**第三步**, 直到灰色中无任何对象，如图所示。

   ![image-20211201173355805](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/image-20211201173355805.png)

   &gt;当我们全部的可达对象都遍历完后，灰色标记表将不再存在灰色对象，目前全部内存的数据只有两种颜色，黑色和白色。那么黑色对象就是我们程序逻辑可达（需要的）对象，这些数据是目前支撑程序正常业务运行的，是合法的有用数据，不可删除，白色的对象是全部不可达对象，目前程序逻辑并不依赖他们，那么白色对象就是内存中目前的垃圾数据，需要被清除。

5. **第五步**: 回收所有的白色标记表的对象. 也就是回收垃圾，如图所示。将全部的白色对象进行删除回收，剩下的就是全部依赖的黑色对象。

   &lt;img src=&#34;https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/image-20211201173458193.png&#34; alt=&#34;image-20211201173458193&#34; style=&#34;zoom:50%;&#34; /&gt;

##### 2.1.2 缺点

- 为了保证数据安全, 仍然存在 STW 缺陷

&gt; 如果没有STW的三色标记法

1. 第一轮扫描，目前黑色的有对象1和对象4， 灰色的有对象2和对象7，其他的为白色对象，且对象2是通过指针p指向对象3的，如图所示。

   &lt;img src=&#34;https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/image-20211201174103798.png&#34; alt=&#34;image-20211201174103798&#34; style=&#34;zoom:33%;&#34; /&gt;

2. 现在如何三色标记过程不启动STW，那么在GC扫描过程中，任意的对象均可能发生读写操作，如图所示，在还没有扫描到对象2的时候，已经标记为黑色的对象4，此时创建指针q，并且指向白色的对象3。

   &lt;img src=&#34;https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/image-20211201174127576.png&#34; alt=&#34;image-20211201174127576&#34; style=&#34;zoom:33%;&#34; /&gt;

3. 与此同时灰色的对象2将指针p移除，那么白色的对象3实则就是被挂在了已经扫描完成的黑色的对象4下，如图所示。

   ![image-20211201174209570](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/image-20211201174209570.png)

4. 然后我们正常指向三色标记的算法逻辑，将所有灰色的对象标记为黑色，那么对象2和对象7就被标记成了黑色，如图所示。`对象3`只能等待被清除, 产生错误! 最后本来是对象4合法引用的对象3，却被GC给“误杀”回收掉了。

   ![image-20211201174230117](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/image-20211201174230117.png)



### 2.2 屏障机制

##### 2.2.1 强弱三色不变式

- 强三色不变式: 不存在黑色对象引用到白色对象的指针。

  &gt; 强三色不变式实际上是强制性的不允许黑色对象引用白色对象，这样就不会出现有白色对象被误删的情况。

  ![image-20211201174718596](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/image-20211201174718596.png)

- 若三色不变式:  所有被黑色对象引用的白色对象都处于灰色保护状态。

  &gt;弱三色不变式强调，黑色对象可以引用白色对象，但是这个白色对象必须存在其他灰色对象对它的引用，或者可达它的链路上游存在灰色对象。 这样实则是黑色对象引用白色对象，白色对象处于一个危险被删除的状态，但是上游灰色对象的引用，可以保护该白色对象，使其安全。

  ![image-20211201174930624](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/image-20211201174930624.png)

##### 2.2.2 屏障机制

&gt; 为了遵循上述的强弱三色不变式，GC算法演进到两种屏障方式，他们“插入屏障”, “删除屏障”。GC算法演进到两种屏障方式，他们“插入屏障”, “删除屏障”。

- **插入屏障**

  `具体操作`: 在`黑色A对象`引用`白色B对象`的时候，B对象被标记为灰色。(将B挂在A下游，B必须被标记为灰色)

  `满足`: **强三色不变式**. (不存在黑色对象引用白色对象的情况了， 因为白色会强制变成灰色)

  

- **删除屏障**

  `具体操作`: 被破坏的对象，如果自身为灰色或者白色，那么被标记为灰色。

  `满足`: **弱三色不变式**. (保护灰色对象到白色对象的路径不会断)

##### 2.2.3 缺点

- 插入写屏障：结束时需要STW来重新扫描栈，标记栈上引用的白色对象的存活；
- 删除写屏障：回收精度低，GC开始时STW扫描堆栈来记录初始快照，这个过程会保护开始时刻的所有存活对象。

## 3. Go-v1.8&#43; 三色标记 &#43; 混合写屏障

&gt; Go V1.8版本引入了混合写屏障机制（hybrid write barrier），避免了对栈re-scan的过程，极大的减少了STW的时间。结合了两者的优点。
&gt;
&gt; 注意混合写屏障是Gc的一种屏障机制，所以只是当程序执行GC的时候，才会触发这种机制。

### 3.1 混合写屏障原理

&gt; 注意: 混合写屏障是Gc的一种屏障机制，所以只是当程序执行GC的时候，才会触发这种机制。
&gt;

1. GC开始将`栈上的可达对象`全部扫描并标记为黑色(之后不再进行第二次重复扫描，无需STW)，栈上不再启用屏障, 只有堆启用屏障.

2. GC期间，任何在栈上创建的新对象，均为黑色。

3. 栈上被删除的对象标记为灰色。

4. 栈上被添加的对象标记为灰色。

`满足`: 变形的**强弱三色不变式**.结合了`插入/删除屏障`的优点

### 3.2 混合写屏障的具体场景

- GC开始：扫描栈区，将可达对象全部标记为黑

  ![image-20211202100713544](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/image-20211202100713544.png)

##### 3.2.1 场景一

&gt;  对象被一个堆对象删除引用，成为栈对象的下游(A引用B, A就是B的下游)

- 前提：

  堆对象4-&gt;对象7 = 对象7；  //对象7 被 对象4引用
  栈对象1-&gt;对象7 = 堆对象7；  //将堆对象7 挂在 栈对象1 下游
  堆对象4-&gt;对象7 = null；    //对象4 删除引用 对象7

![img](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/64c76eea3706c37f160b8345b7b3742c_1920x1080.jpeg)

![img](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/4d6728d276d2786017cde37b824333aa_1920x1080.jpeg)

##### 3.2.2 场景二

&gt; 对象被一个栈对象删除引用，成为另一个栈对象的下游

- new 栈对象9；
  对象8-&gt;对象3 = 对象3；      //将栈对象3 挂在 栈对象9 下游
  对象2-&gt;对象3 = null；      //对象2 删除引用 对象3

![img](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/beedb81ec3cd5a4813aaa5bce1341949_1920x1080.jpeg)

![img](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/48569d6dfb8ac6f1b0d6238a9d8150b3_1920x1080-20211202110233896.jpeg)

![img](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/46e6be62e880e0f5796bc1e6f050b512_1920x1080.jpeg)

# 四. GC触发时机

1. 主动调用`runtime.GC`

2. 当距离上一个 GC 周期的时间超过一定时间时，将会触发。时间周期以runtime.forcegcperiod 变量为准，默认 2 分钟

3. 当所分配的堆大小达到阈值（由控制器计算的触发堆的大小）时，将会触发。

   - **申请内存触发 runtime.mallocgc**

     Go运行时会将堆上的对象按大小分成微对象、小对象和大对象三类，这三类对象的创建都可能会触发新的GC

     1. 当前线程的内存管理单元中不存在空闲空间时，创建微对象 `(noscan&amp;&amp;size&lt;maxTinySize)`和小对象需要调用 `runtime.mcache.nextFree`从中心缓存或者页堆中获取新的管理单元，这时如果span满了就会导致返回的 `shouldhelpgc=true`，就可能触发垃圾收集；
     2. 当用户程序申请分配 32KB 以上的大对象时，一定会构建 `runtime.gcTrigger`结构体尝试触发垃圾收集；

     ```javascript
     // src/runtime/mgc.go
     func (t gcTrigger) test() bool {
     	if !memstats.enablegc || panicking != 0 || gcphase != _GCoff {
     		return false
     	}
     	switch t.kind {
     	case gcTriggerHeap:
     		// Non-atomic access to gcController.heapLive for performance. If
     		// we are going to trigger on this, this thread just
     		// atomically wrote gcController.heapLive anyway and we&#39;ll see our
     		// own write.
     		return gcController.heapLive &gt;= gcController.trigger  // 是否触发gc
     	case gcTriggerTime:
     		if gcController.gcPercent &lt; 0 {
     			return false
     		}
     		lastgc := int64(atomic.Load64(&amp;memstats.last_gc_nanotime))
     		return lastgc != 0 &amp;&amp; t.now-lastgc &gt; forcegcperiod
     	case gcTriggerCycle:
     		// t.n &gt; work.cycles, but accounting for wraparound.
     		return int32(t.n-work.cycles) &gt; 0
     	}
     	return true
     }
     ```

     这个时候调用 `t.test()`执行的是 `gcTriggerHeap`情况，只需要判断 `gcController.heapLive&gt;=gcController.trigger`的真假就可以了。`heapLive` 表示垃圾收集中存活对象字节数， `trigger`表示触发标记的堆内存大小的；当内存中存活的对象字节数大于触发垃圾收集的堆大小时，新一轮的垃圾收集就会开始。

     1. `heapLive` — 为了减少锁竞争，运行时只会在中心缓存分配或者释放内存管理单元以及在堆上分配大对象时才会更新；
     2. `trigger` — 在标记终止阶段调用 `runtime.gcSetTriggerRatio` 更新触发下一次垃圾收集的堆大小，它能够决定触发垃圾收集的时间以及用户程序和后台处理的标记任务的多少，利用反馈控制的算法根据堆的增长情况和垃圾收集 CPU 利用率确定触发垃圾收集的时机。

# 五. Golang GC总结

- v1.3 使用标记清除, 存在STW(stop the word)机制, 垃圾回收时程序暂停, 效率低下
- v1.5 三色标记法 &#43; 删除/插入屏障  : 栈空间不动, 堆空间全部重新扫描,存在STW(stop the word)机制, 效率低下.
- vGoV1.8-三色标记法，混合写屏障机制， 栈空间不启动，堆空间启动。整个过程几乎不需要STW，效率较高。

---

> 作者: [Fred](https://github.com/ipfred)  
> URL: https://ipfred.github.io/lang/go/go_advanced/20250515175202/  

