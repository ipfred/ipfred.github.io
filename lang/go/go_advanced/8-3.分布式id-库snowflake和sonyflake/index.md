# 

## 分布式ID的特点

- 全局唯一性：不能出现有重复的ID标识，这是基本要求。
- 递增性：确保生成ID对于用户或业务是递增的。
- 高可用性：确保任何时候都能生成正确的D。
- 高性能性：在高并发的环境下依然表现良好。

&gt;  不仅仅是用于用户ID，实际互联网中有很多场景需要能够生成类似MySQL自增ID这样不断增大，同时又不会重复的id。以支持业务中的高并发场景。比较典型的场景有：
&gt;
&gt; - 电商促销时短时间内会有大量的订单涌入到系统，比如每秒10w&#43;；
&gt; - 明星出轨时微博短时间内会产生大量的相关微博转发和评论消息。
&gt;
&gt; 在这些业务场景下将数据插入数据库之前，我们需要给这些订单和消息先分配一个唯一ID，然后再保存到数据库中。对这个id的要求是希望其中能带有一些时间信息，这样即使我们后端的系统对消息进行了分库分表，也能够以时间顺序对这些消息进行排序。

## snowflake

### 1. 原理

- Twitter 的 Snowflake 算法规范

  - 整个ID是存储在int64中的63位整数
  - 41位用于存储收到请求的时间戳 - 单位毫秒
  - 10位用于存储节点 - 范围从0到1023
  - 12位用于存储序列号 - 范围从0到4095

  ![image-20220427111710044](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/image-20220427111710044.png)

1. 由于最高位是标识位，为1表示为负数，因此最高位不使用。
2. 41bit 保存时间戳，精确到毫秒。也就是说最大可以使用的年限是69年。
3. 10bit 的机器位，能部属在1024台机器节点来生成ID。
4. 12bit 的序列号，一毫秒最大生成惟一ID的数量为4096个。

&gt; - 1 bit：不用，为啥呢？因为二进制里第一个bit为如果是1，那么都是负数，但是我们生成的id都是正数，所以第一个bit统一都是0
&gt;
&gt; - 41 bit：41 bit可以表示的数字多达2^41 - 1，也就是可以标识2 ^ 41 - 1个毫秒值，换算成年就是表示69年的时间。
&gt;
&gt; - 10 bit：代表的是这个服务最多可以部署在2^10台机器上哪，也就是1024台机器。但是10 bit里5个bit代表机房id，5个bit代表机器id。意思就是最多代表2 ^ 5个机房（32个机房），每个机房里可以代表2 ^ 5个机器（32台机器）。
&gt;
&gt; - 12 bit：这个是用来记录同一个毫秒内产生的不同id，12 bit可以代表的最大正整数是2 ^ 12 - 1 = 4096，也就是说可以用这个12bit代表的数字来区分同一个毫秒内的4096个不同的id
&gt;
&gt; *同一台机器上，同一毫秒内可以产生4096个id，一秒共400w个id，理论完全够用。*

### 2. demo

- 推特开源库: https://github.com/bwmarrin/snowflake

```go
package main

import (
	&#34;fmt&#34;
	&#34;github.com/bwmarrin/snowflake&#34;
	&#34;log&#34;
	&#34;time&#34;
)

func BuildSnowFlakeId(starTime string, machineID int64) (id int64, err error) {
	var st time.Time
	st, err = time.Parse(&#34;2006-01-02&#34;, starTime)
	if err != nil {
		return 0, err
	}
	snowflake.Epoch = st.UnixNano() / 1000000
	node, err := snowflake.NewNode(machineID)
	if err != nil {
		return 0, err
	}
	id = node.Generate().Int64()
	return id, err
}

func main() {
	startTime := time.Now().Format(&#34;2006-01-02&#34;)
	snowFlakeId, err := BuildSnowFlakeId(startTime, 1)
	if err != nil {
		log.Fatalln(err)
	}
	fmt.Println(snowFlakeId)

}

```



## sonyflake

### 1. 原理

- `Snowflake`算法是相当灵活的，我们可以根据自己的业务需要，对63 bit的的各个部分进行增减。索尼公司的[Sonyflake](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fsony%2Fsonyflake)对原生的`Snowflake`进行改进，重新分配了各部分的bit位:

  ![image-20220426204201273](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/image-20220426204201273.png)

1. 由于最高位是标识位，为1表示为负数，因此最高位不使用.
2. 39bit 来保存时间戳，与原生的Snowflake不同的地方是，Sonyflake是以10毫秒为单位来保存时间的。这样的话，可以使用的年限为 **174年** 比Snowflake长太多了。
3. 8bit 做为序列号，每10毫最大生成256个，1秒最多生成25600个，比原生的Snowflake少好多，如果感觉不够用，目前的解决方案是跑多个实例生成同一业务的ID来弥补。
4. 16bit 做为机器号，默认的是当前机器的私有IP的最后两位

### 2. demo

```go
package main

import (
	&#34;fmt&#34;
	&#34;time&#34;

	&#34;github.com/sony/sonyflake&#34;
)

func BuildSonyFlakeID(starTime string, machineID uint16) (id int64, err error) {
	var sf *sonyflake.Sonyflake
	var st sonyflake.Settings
	st.MachineID = func() (uint16, error) {
		return machineID, nil
	}
	starT, err := time.Parse(&#34;2006-01-02&#34;, starTime)
	if err != nil {
		return 0, err
	}
	st.StartTime = starT
	sf = sonyflake.NewSonyflake(st)
	if sf == nil {
		panic(&#34;sonyflake not created&#34;)
	}
	idUint64, err := sf.NextID()
	if err != nil {
		return 0, err
	}
	return int64(idUint64), err
}

func main() {
	id, err := BuildSonyFlakeID(time.Now().Format(&#34;2006-01-02&#34;), 1)
	if err != nil {
		panic(err)
	}
	fmt.Println(id)
}

```



---

> 作者:   
> URL: http://localhost:1313/lang/go/go_advanced/8-3.%E5%88%86%E5%B8%83%E5%BC%8Fid-%E5%BA%93snowflake%E5%92%8Csonyflake/  

