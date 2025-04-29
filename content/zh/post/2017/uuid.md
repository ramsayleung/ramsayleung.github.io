+++
title = "Java UUID 源码剖析"
description = "a introduction about uuid"
date = 2017-05-18T00:00:00+08:00
lastmod = 2022-02-23T22:34:25+08:00
tags = ["java"]
categories = ["java"]
draft = false
toc = true
showQuote = true
+++

笔者近来闲来无事，又因为有需要构造全局唯一 ID 的需求，所以就去看了 UUID 这个提供稳定的系统唯一标识符的类的源码


## <span class="section-num">1</span> UUID variant {#uuid-variant}

事实上是存在很多中 UID 的不同实现的的，但是 UUID 里面默认是使用 "加盐"(Leach-Salz)实现，但是也可以使用其他的实现。


## <span class="section-num">2</span> Layout of variant2(Leach-Salz) UUID {#layout-of-variant2--leach-salz--uuid}

加盐的 UUID 的结构布局如下：最高位 (most significant) 的64 位长整型值由下面的的无符号位组成：

-   0xFFFFFFFF00000000 time_low //时间的低位值
-   0x00000000FFFF0000 time_mid //时间的中位值
-   0x000000000000F000 version // 说明 UUID 的类型，1,2,3,4 分别代表 基于时间，基于 DEC，基于命名，和随机产生的 UUID
-   0x0000000000000FFF time_hi //时间的高位值

最低位 (least significant) 的 64 位长整型由以下的无符号位组成：

-   0xC000000000000000 variant //说明UUID 的结构布局，并且只有在类型 2 (加盐类型), 结构布局才有效
-   0x3FFF000000000000 clock_seq
-   0x0000FFFFFFFFFFFF node


## <span class="section-num">3</span> UUID constructor {#uuid-constructor}

UUID 类有两个构造函数，分别是 public 和 private 修饰的构造函数


### <span class="section-num">3.1</span> private UUID {#private-uuid}

private 类型的构造函数以一个 byte 数组为构造参数：

```java

/*
 * Private constructor which uses a byte array to construct the new UUID.
 */
private UUID(byte[] data) {
    long msb = 0;
    long lsb = 0;
    assert data.length == 16 : "data must be 16 bytes in length";
    for (int i=0; i<8; i++)
	msb = (msb << 8) | (data[i] & 0xff);
    for (int i=8; i<16; i++)
	lsb = (lsb << 8) | (data[i] & 0xff);
    this.mostSigBits = msb;
    this.leastSigBits = lsb;
}
```

private 构造器完成的工作主要是通过左移位，与运算和或运算对 mostSigBit 和 leastSigBit 赋值。 private的构造函数只能在类本身被调用, 该构造器的用法会在接下来阐述。


### <span class="section-num">3.2</span> public UUID {#public-uuid}

public 类型的构造器接受两个 `long` 类型的参数，即上面提到的最高位和最低位：

```java
/**
 * Constructs a new {@code UUID} using the specified data.  {@code
 * mostSigBits} is used for the most significant 64 bits of the {@code
 * UUID} and {@code leastSigBits} becomes the least significant 64 bits of
 * the {@code UUID}.
 *
 * @param  mostSigBits
 *         The most significant bits of the {@code UUID}
 *
 * @param  leastSigBits
 *         The least significant bits of the {@code UUID}
 */
public UUID(long mostSigBits, long leastSigBits) {
    this.mostSigBits = mostSigBits;
    this.leastSigBits = leastSigBits;
}
```

使用最高位和最低位的值来构造 UUID, 而最高位和最低位的赋值是在 private 的构造器里面完成的。


## <span class="section-num">4</span> UUID type {#uuid-type}


### <span class="section-num">4.1</span> type 4 -- randomly generated UUID {#type-4-randomly-generated-uuid}

现在就看看使用频率最高的 UUID 类型 -- 随机的 UUID 以及随机生成 UUID 的函数：
`randomUUID()`

```java
/**
 * Static factory to retrieve a type 4 (pseudo randomly generated) UUID.
 *
 * The {@code UUID} is generated using a cryptographically strong pseudo
 * random number generator.
 *
 * @return  A randomly generated {@code UUID}
 */
public static UUID randomUUID() {
    SecureRandom ng = Holder.numberGenerator;

    byte[] randomBytes = new byte[16];
    ng.nextBytes(randomBytes);
    randomBytes[6]  &= 0x0f;  /* clear version        */
    randomBytes[6]  |= 0x40;  /* set to version 4     */
    randomBytes[8]  &= 0x3f;  /* clear variant        */
    randomBytes[8]  |= 0x80;  /* set to IETF variant  */
    return new UUID(randomBytes);
}
```

关于调用到的 `Holder` 变量的定义：

```java
/*
 * The random number generator used by this class to create random
 * based UUIDs. In a holder class to defer initialization until needed.
 */
private static class Holder {
    static final SecureRandom numberGenerator = new SecureRandom();
}
```

上面用到 `java.security.SecureRandom` 类来生成字节数组， `SecureRandom` 是被认为是达到了加密强度 (cryptographically strong) 并且因为不同的 JVM 而有不同的实现的。所以可以保证产生足够 "随机"的随机数以保证 UUID 的唯一性。

然后在即将用来构造的 UUID 的字节数组重置和添加关于 UUID 的相关信息，例如版本，类型信息等，然后把处理好的字节数组传到 private 的构造器以构造 UUID。这里的`randomUUID` 静态方法就是通过静态工厂的方式构造 UUID.


### <span class="section-num">4.2</span> type 3 -- name-based UUID {#type-3-name-based-uuid}

在上面关于 UUID 结构布局的时候提到，UUID 有四种类型的实现，而类型3 就是基于命名的实现：

```java
/**
 * Static factory to retrieve a type 3 (name based) {@code UUID} based on
 * the specified byte array.
 *
 * @param  name
 *         A byte array to be used to construct a {@code UUID}
 *
 * @return  A {@code UUID} generated from the specified array
 */
public static UUID nameUUIDFromBytes(byte[] name) {
    MessageDigest md;
    try {
	md = MessageDigest.getInstance("MD5");
    } catch (NoSuchAlgorithmException nsae) {
	throw new InternalError("MD5 not supported", nsae);
    }
    byte[] md5Bytes = md.digest(name);
    md5Bytes[6]  &= 0x0f;  /* clear version        */
    md5Bytes[6]  |= 0x30;  /* set to version 3     */
    md5Bytes[8]  &= 0x3f;  /* clear variant        */
    md5Bytes[8]  |= 0x80;  /* set to IETF variant  */
    return new UUID(md5Bytes);
}

```

`MessageDigest` 是 JDK 提供用来计算散列值的类，使用的散列算法包括Sha-1,Sha-256 或者是 MD5 等等。

`nameUUIDFromBytes` 使用 MD5 算法计算传进来的参数 name 的散列值，然后在散列值重置，添加 UUID 信息，然后再使用生成的散列值 (字节数组)传递给 private 构造器以构造 UUID.

这里的 `nameUUIDFromBytes` 静态方法也是通过静态工厂的方式构造 UUID.


### <span class="section-num">4.3</span> type 2 -- DEC security {#type-2-dec-security}

在 JDK 的 UUID 类中并未提供 基于 DEC 类型的 UUID 的实现。


### <span class="section-num">4.4</span> type 1 -- time-based UUID {#type-1-time-based-uuid}

与基于命名和随机生成的 UUID 都有一个静态工厂方法不一样， 基于时间的 UUID 并不存在静态工厂方法，time-based UUID 是基于一系列相关的方法的：


#### <span class="section-num">4.4.1</span> timestamp {#timestamp}

```java
public long timestamp() {
    if (version() != 1) {
	throw new UnsupportedOperationException("Not a time-based UUID");
    }

    return (mostSigBits & 0x0FFFL) << 48
	| ((mostSigBits >> 16) & 0x0FFFFL) << 32
	| mostSigBits >>> 32;
}
```

60 个bit长的时间戳是由上面提到的 `time_low` `time_mid` `time_hi` 构造而成的。

而时间的计算是从 UTC 时间的 1582 年 10月 15 的凌晨开始算起，结果的值域在 100-nanosecond 之间。

但是这个时间戳的值只是对基于时间的 UUID 有效的，对于其他类型的 UUID, `timestamp()` 方法会抛出`UnsuportedOperationException`异常。


#### <span class="section-num">4.4.2</span> clockSequence() {#clocksequence}

```java
public int clockSequence() {
    if (version() != 1) {
	throw new UnsupportedOperationException("Not a time-based UUID");
    }

    return (int)((leastSigBits & 0x3FFF000000000000L) >>> 48);
}
```

14 个 bit 长的时钟序列值是从 该UUID 的时钟序列域构造出来的(clock sequence filed).

而时钟序列域通常是用来保证基于时间的 UUID 的唯一性。跟 `timestamp()` 函数一样， `clockSequence()` 函数也只对基于时间的 UUID 有效。 对于其他类型的 UUID, 它会抛出`UnsuportedOperationException`异常。


#### <span class="section-num">4.4.3</span> node() {#node}

48 个 bit 长的节点值是从该 UUID 的节点域 (node filed) 构造出来的。节点域通过保存运行 JVM 机器的局域网地址 (IEEE 802) 来保证该机器生成 UUID 的空间唯一性。

和上述方法一样， `node()` 方法只对基于时间的 UUID 有效，对于其他类型的 UUID 该方法会抛出`UnsuportedOperationException`异常。

---

对应 field 的图示

```nil
0                   1                   2                   3
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                          time_low                             |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|       time_mid                |         time_hi_and_version   |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|clk_seq_hi_res |  clk_seq_low  |         node (0-1)            |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                         node (2-5)                            |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```


## <span class="section-num">5</span> FromString()/ToString() {#fromstring-tostring}


### <span class="section-num">5.1</span> toString() {#tostring}

以字符串的形式表示 UUID, 格式说明：

```nil
hexDigit               =
	"0" | "1" | "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9"
	| "a" | "b" | "c" | "d" | "e" | "f"
	| "A" | "B" | "C" | "D" | "E" | "F"
hexOctet               = <hexDigit><hexDigit>
time_low               = 4*<hexOctet>
time_mid               = 2*<hexOctet>
time_high_and_version  = 2*<hexOctet>
variant_and_sequence   = 2*<hexOctet>
node                   = 6*<hexOctet>

UUID = <time_low> "-" <time_mid> "-" <time_high_and_version> "-" "variant_and_sequence" "-" <node>
```

而关于这些不同 field 的大小，之前的内容已经有图示，需要的可以去回顾。

```java
/** Returns val represented by the specified number of hex digits. */
private static String digits(long val, int digits) {
    long hi = 1L << (digits * 4);
    return Long.toHexString(hi | (val & (hi - 1))).substring(1);
}

public String toString() {
    return (digits(mostSigBits >> 32, 8) + "-" +
	    digits(mostSigBits >> 16, 4) + "-" +
	    digits(mostSigBits, 4) + "-" +
	    digits(leastSigBits >> 48, 4) + "-" +
	    digits(leastSigBits, 12));
}
```


### <span class="section-num">5.2</span> fromString() {#fromstring}

与 `toString()` 函数功能相反， `fromString()` 函数的作用就是将字符串形式的对象解码成 UUID 对象：

```java
public static UUID fromString(String name) {
    String[] components = name.split("-");
    if (components.length != 5)
	throw new IllegalArgumentException("Invalid UUID string: "+name);
    for (int i=0; i<5; i++)
	components[i] = "0x"+components[i];

    long mostSigBits = Long.decode(components[0]).longValue();
    mostSigBits <<= 16;
    mostSigBits |= Long.decode(components[1]).longValue();
    mostSigBits <<= 16;
    mostSigBits |= Long.decode(components[2]).longValue();

    long leastSigBits = Long.decode(components[3]).longValue();
    leastSigBits <<= 48;
    leastSigBits |= Long.decode(components[4]).longValue();

    return new UUID(mostSigBits, leastSigBits);
}
```


## <span class="section-num">6</span> 使用场景 {#使用场景}

UUID 一般用来生成全局唯一标识符，那么 UUID 是否能保证唯一呢？以`UUID.randomUUID()` 生成的 UUID 为例，从上面的源码，除了 version 和 variant是固定值之外，另外的 14 byte 都是足够随机的.

如果你生成的是 128 bit 长的 UUID 的话，理论上是 2的14x8=114次方才会有一次重复。这是个什么概念的呢？ 即你每秒能 生成 10 亿个 UUID, 在100年以后，你就有 50%的可能性产生一个重复的 UUID了，是不是很开心呢？

即使你使用 `UUID.randomUUID.getLeastSignificant()` 生成长整型的ID, 你理论上需要生成 2的56次方个 ID 后才会产生一个重复的 ID, 所以你可以放心地使用 UUID 了 :)

<div center class="qr-container">
<img src="/ox-hugo/qrcode_gh_e06d750e626f_1.jpg" alt="qrcode_gh_e06d750e626f_1.jpg" width="160px" height="160px" center="t" class="qr-container" />
公号同步更新，欢迎关注👻
</div>

