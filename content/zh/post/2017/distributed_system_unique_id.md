+++
title = "关于分布式系统唯一ID的探究"
description = "An discussion about unique id in distributed system"
date = 2017-05-23T00:00:00+08:00
keywords = ["uuid", "java"]
lastmod = 2022-02-23T19:03:46+08:00
tags = ["java", "algorithm", "distributed_system", "design"]
categories = ["algorithm"]
draft = false
toc = true
+++

最近我需要为运行的分布式系统某部分模块构造系统唯一的ID, 而 ID 需要是数字的形式，并应该尽量的短。不得不说，这是一个有趣的问题


## <span class="section-num">1</span> 若干实现策略 {#若干实现策略}

查阅完相关的资料，发现为分布式系统生成唯一 ID 方法挺多的，例如：

-   UUID
-   使用一个 ticket server, 即中央的服务器，各个节点都从中央服务器取 ID
-   Twitter 的 Snowflake 算法
-   Boundary 的 flake 算法

其中 UUID 生成的 ID 是字符串＋数字，不适用； ticket server 的做法略麻烦，笔者
并不想为了个 ID 还要去访问中央服务器；剩下就是 Snowflake 和 flake 算法，
flake 算法生成的是 128 位的 ID, 略长；所以最后笔者选择了 Snowflake 算法。


## <span class="section-num">2</span> Snowflake 算法实现 {#snowflake-算法实现}

本来 Twitter 的算法是有相应实现的，不过后来删除了；笔者就只好自己卷起袖子自己 实现了:(

虽说 Twitter 没有了相应的实现，但是 Snowflake 算法原理很简单，实现起来并不难.


### <span class="section-num">2.1</span> Snowflake 算法 {#snowflake-算法}

Snowflake 算法生成 64 位的 ID, ID 的格式是 41 位的时间戳 + 10 位的截断的 mac 地址 + 12 位递增序列：

```python
"""id format =>
timestamp | machineId|sequence
41        | 10       |12
"""
```


### <span class="section-num">2.2</span> Snowflake 实现 {#snowflake-实现}


#### <span class="section-num">2.2.1</span> 生成时间戳 {#生成时间戳}

Java 内置了生成精确到毫秒的时间戳的方法，非常便利：

```java
long timestamp = System.currentTimeMillis();
```


#### <span class="section-num">2.2.2</span> 生成递增序列 {#生成递增序列}

12 bits 的最大值是 2\*\*12=4096, 所以生成递增序列也非常简单：

```java
private final long sequenceMax = 4096; //2**12
private volatile long sequence = 0L;
public void generateId(){
    //do something
    sequence = (sequence + 1) % sequenceMax;
    //do something
}
```


#### <span class="section-num">2.2.3</span> 获取 Mac 地址 {#获取-mac-地址}

我们通过获取当前机器的 IP 地址以获取对应的物理地址：

```java
protected long getMachineId() throws GetHardwareIdFailedException {

    try {
	InetAddress ip = InetAddress.getLocalHost();
	NetworkInterface network = NetworkInterface.getByInetAddress(ip);
	long id;
	if (network == null) {
	    id = 1;
	} else {
	    byte[] mac = network.getHardwareAddress();
	    id =
		((0x000000FF & (long) mac[mac.length - 1]) |
		 (0x0000FF00 & (((long) mac[mac.length - 2]) << 8))) >> 6;
	}
	return id;
    } catch (SocketException e) {
	throw new GetHardwareIdFailedException(e);
    } catch (UnknownHostException e) {
	throw new GetHardwareIdFailedException(e);
    }
}
```

又因为 Mac 地址是6 个字节 (48 bits)，而需要的只是 10 bit, 所以需要取最低位的 2个字节 (16 bits)，然后右移 6 bits 以获取 10 个 bits 的 Mac地址


### <span class="section-num">2.3</span> Snowflake 完整代码 {#snowflake-完整代码}

下面是 Snowflake 的 Java 实现：

```java
public class IdGenerator {

    //   id format  =>
    //   timestamp |datacenter | sequence
    //   41        |10         |  12
    private final long sequenceBits = 12;
    private final long machineIdBits = 10L;
    private final long MaxMachineId = -1L ^ (-1L << machineIdBits);

    private final long machineIdShift = sequenceBits;
    private final long timestampLeftShift = sequenceBits + machineIdBits;
    private static final Object lock=new Object();
    private final long twepoch = 1288834974657L;
    private final long machineId;
    private final long sequenceMax = 4096; //2**12

    private volatile long lastTimestamp = -1L;
    private volatile long sequence = 0L;

    private static volatile IdGenerator  instance;

    public static IdGenerator getInstance() throws Exception {
	IdGenerator generator=instance;
	if (instance == null) {
	    synchronized(lock){
		generator=instance;
		if(generator==null){
		    generator=new IdGenerator();
		    instance=generator;
		}
	    }
	}
	return generator;
    }

    private IdGenerator() throws Exception {
	machineId = getMachineId();
	if (machineId > MaxMachineId || machineId < 0) {
	    throw new Exception("machineId > MaxMachineId");
	}
    }

    public synchronized Long generateLongId() throws Exception {
	long timestamp = System.currentTimeMillis();
	if (timestamp < lastTimestamp) {
	    throw new Exception(
				"Clock moved backwards.  Refusing to generate id for " + (
											  lastTimestamp - timestamp) + " milliseconds.");
	}
	if (lastTimestamp == timestamp) {
	    sequence = (sequence + 1) % sequenceMax;
	    if (sequence == 0) {
		timestamp = tillNextMillis(lastTimestamp);
	    }
	} else {
	    sequence = 0;
	}
	lastTimestamp = timestamp;
	Long id = ((timestamp - twepoch) << timestampLeftShift) |
	    (machineId << machineIdShift) |
	    sequence;
	return id;
    }

    protected long tillNextMillis(long lastTimestamp) {
	long timestamp = System.currentTimeMillis();
	while (timestamp <= lastTimestamp) {
	    timestamp = System.currentTimeMillis();
	}
	return timestamp;
    }

    protected long getMachineId() throws GetHardwareIdFailedException {

	try {
	    InetAddress ip = InetAddress.getLocalHost();
	    NetworkInterface network = NetworkInterface.getByInetAddress(ip);
	    long id;
	    if (network == null) {
		id = 1;
	    } else {
		byte[] mac = network.getHardwareAddress();
		id =
		    ((0x000000FF & (long) mac[mac.length - 1]) |
		     (0x0000FF00 & (((long) mac[mac.length - 2]) << 8))) >> 6;
	    }
	    return id;
	} catch (SocketException e) {
	    throw new GetHardwareIdFailedException(e);
	} catch (UnknownHostException e) {
	    throw new GetHardwareIdFailedException(e);
	}
    }
}
```

正如笔者所言，算法并不难，就是分别获取时间戳， mac 地址，和递增序列号，然后移位得到 ID. 但是在具体的实现中还是有一些需要注意的细节的。


#### <span class="section-num">2.3.1</span> 线程同步 {#线程同步}

因为算法中使用到递增的序列号来生成 ID,而在实际的开发或者生产环境中很可能不止一个线程在使用 IdGenerator 这个类，如果这样就很容易出现不同线程的竞争问题，所以我使用了单例模式来生成 ID, 一方面更符合生成器的设计，另一方面因为对生成 ID的方法进行了同步，就保证了不会出现竞争问题。


#### <span class="section-num">2.3.2</span> 同一毫秒生成多个 ID {#同一毫秒生成多个-id}

因为序列号长度是 12个 bit, 那么序列号最大值就是2\*\*12=4096了，此外时间戳是精确到毫秒的，这就是意味着，当一毫秒内，产生超过 4096 个 ID 的时候就会出现重复的ID.

这样的情况并不是不可能发生，所以要对此进行处理；所以在 _generateId()_ 函数中：

```java
if (lastTimestamp == timestamp) {
    sequence = (sequence + 1) % sequenceMax;
    if (sequence == 0) {
	timestamp = tillNextMillis(lastTimestamp);
    }
}
```

有以上的一段代码。当现在的时间戳与之前的时间戳一致，那么就意味着还是同一毫秒，如果序列号为 0, 就说明已经产生了 4096 个 ID了，继续产生 ID,就会出现重复 ID, 所以要等待一毫秒，这个就是 _tillNextMills()_ 函数的作用了。


## <span class="section-num">3</span> 小结 {#小结}

算法虽然简单，但是在找到 Snowflake 算法之前，笔者尝试了挺多的算法，但是都是因为不符合要求而被一一否决， 而 Snowflake 算法虽然简单，但是胜在实用。最后附上我写的 snowflake 算法的 Python 实现： [Snowfloke](https://github.com/samrayleung/snowflake)