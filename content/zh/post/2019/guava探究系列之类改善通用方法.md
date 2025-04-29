+++
title = "Guava探究系列之三：改善通用方法"
date = 2019-07-25T22:33:00-07:00
lastmod = 2025-01-09T18:45:48-08:00
tags = ["java"]
categories = ["Guava探究"]
draft = false
toc = true
showQuote = true
+++

## <span class="section-num">1</span> 前言 {#前言}

Java 是一门集大成的面向对象语言, 在Java的世界里, 一切皆对象, 而=Object=类就是所有对象的默认父类. Guava
提供了若干个工具方法来扩展=Object=类的通用能力.


## <span class="section-num">2</span> equals {#equals}

在Java的编程世界, 比较两个对象是个很常见的操作, `Object=类也提供了一个=equals=方法来判断对象是否相等.
  但是=Object=使用的=equals=方法有诸多不便, 最痛苦的是无处不在的=NullPointerException`, 例如:

```java
public testEqueal(Object input){
    this.equals(input);
}
```

但当 =this=指针指向一个空对象的时候,
就会出现=null.testEqueal(input)=的情形, 就会抛出NPE.
为了让=equals=方法更易用,
Guava提供了一个=Objects.equal(Object a, Object b)=方法来判断两个对象是否相等.
用法如下:

```java
Objects.equal("a", "a"); // returns true
Objects.equal(null, "a"); // returns false
Objects.equal("a", null); // returns false
Objects.equal(null, null); // returns true
```

可能是Java语言的开发者也意识到=Object.equals=方法的不便,
所以在JDK7的时候,
官方也提供了=Objects.equals(Object a, Object b)=的方法,
Guava的竞品自然也没了用武之处。

不过, 说实话, 无论是JDK的=Objects.equals=, `Object.equal=还是Guava的=Object.equal()`,
在日常的开发中也用的不多, 用的最多的是Apache Common库的各种Utils工具,
比较String类型用的是=StringUtils.equals()=,
比较容器(Collection)用的=CollectionUtils.isEqualCollection()=,
毕竟这些工具要更高级和完善(有趣的是, JDK的方法名是=equals=,
Guava的方法名是=equal=, 下文提到的JDK的hash方法名叫=hash=,
Guava叫=hashCode=).


### <span class="section-num">2.1</span> hashCode {#hashcode}

在《Effective Java》的条款9中说到:

> Always override `hashCode` when you override `equals`

就是说在你重写=equals=方法的时候, 记得重写=hashCode=方法,
因为按照Java的约定, 如果两个对象通过调用=equals=方法判断是相等的话,
它们调用=hashCode()=方法的返回结果也是一样的.

《Effective Java》 给出的重写建议是把一个对象的所有字段进行计算取得一个hash值,
示例代码如下:

```java
private volatile int hashCode;
public class User {
    private String name;
    private int age;
    private String passport;
    @Override
    public int hashCode() {
        int result = hashCode;
        if(result == 0){
            result = 17; // Aribtrary number.
            result = 31 * result + name.hashCode();
            result = 31 * result + age; // 31 is an odd prime
            result = 31 * result + passport.hashCode();
            hashCode = result;
        }
        return hashCode;
    }
}
```

这样的计算方式虽然有效, 但是未免过于烦琐, 还要手动计算每个字段. 为此,
Guava 提供了一个 =Objects.hashCode(field1, field2, ..., fieldn=的方法,
用于对所有的字段计算hash值, 用法如下:

```java
public class User {
    public int hashCode() {
        return Objects.hashCode(name, age, passport);
    }
}
```

看起来简洁多了。然后在Java7的时候,
JDK也推出了一个=Objects.hash(field1, field2,...,fieldn)=的方法,
而Guava的竞品很快就被废弃了. 我都在想JDK是不是在吸收Guava的精华,
毕竟实现都一样！(￣▽￣)


## <span class="section-num">3</span> toString {#tostring}

《Effective Java》的条款10说到:

> Always override toString

也就是说, 《Effective Java》建议所有的类都重写=toString()=方法.
其实=toString()=方法不是给程序看的, 而是给开发者自己看的.

据说, 好看的=toString()=方法的输出结果可以让程序员更愉悦, 可见颜值处处都有用.
比较常见的重写=toString()=的方式是把所有的字段拼接输出,
只不过手动拼接有点累.

省心的是, Intellij Idea 为开发者提供了生成=toString()=的快捷方式, 如下图:

{{< figure src="/ox-hugo/Idea_generate_to_string.gif" caption="<span class=\"figure-number\">Figure 1: </span>Idea生成toString" link="/ox-hugo/Idea_generate_to_string.gif" >}}

如果觉得Idea生成的=toString()=有太多的拼接字符串,
还可以试试Guava提供的=toString()=工具方法: `MoreObjects.toStringHelper`,
具体用法如下:

```java
@Test
public void test() {
    System.out.println(MoreObjects.toStringHelper(this)
                       .add("name", "Linus")
                       .toString());

    System.out.println(MoreObjects.toStringHelper("TestToStringHelper")
                       .add("method", "toStringHelper")
                       .toString());
}
// 结果如下:
// ToStringTest{name=Linus}
// TestToStringHelper{method=toStringHelper}
```

使用方法也是很明了, 就不过多赘述.


## <span class="section-num">4</span> compare/compareTo {#compare-compareto}

既然前面提到了《Effective Java》, 那么基于前后呼应的原则,
最后也免不了要再引用一下《Effective Java》:

> 条款12: Consider implementing Comparable

不像前文介绍过的方法, =compareTo=方法并不是=Object=类的方法,
而是=Comparable=接口的方法. 这个方法和前文提到的=equals=方法类似,
只不过用法不一样. =compareTo=通常用于排序,
如下面的代码就是对实现了Comparable接口的Person对象的列表进行排序:

```java
class Person implements Comparable<Person> {
    private String lastName;
    private String firstName;
    private int zipCode;

    public int compareTo(Person other) {
        int cmp = lastName.compareTo(other.lastName);
        if (cmp != 0) {
            return cmp;
        }
        cmp = firstName.compareTo(other.firstName);
        if (cmp != 0) {
            return cmp;
        }
        return Integer.compare(zipCode, other.zipCode);
    }
}

public class CompareTest {
    @Test
    public void testSort(){
        Person[] persons = new Person[2];
        persons[0] = new Person("Ma", "Jack", 12345);
        persons[1] = new Person("Ma", "Pony", 65432);
        Arrays.sort(persons);
        Arrays.stream(persons).forEach(person -> System.out.println(person.getFirstName()));
    }
}
```

上面的代码的逻辑就是先比较=Person.lastName=,
如果相等再比较=Person.firstName=, 如果前面的条件还是相等,
就再比较=Person.zipCode=.

代码的含义相当清晰, 只是有不少的模板代码, 如果能减少这些模板代码, 那就更好了. 幸运的是, Guava 提供了一个
=ComparisonChain=来处理这些模板逻辑, 应用=ComparisonChain=之后的代码如下:

```java
public class ComparisonChainPerson implements Comparable<ComparisonChainPerson> {
    private String lastName;
    private String firstName;
    private int zipCode;

    @Override
    public int compareTo(ComparisonChainPerson that) {
        return ComparisonChain.start()
            .compare(this.lastName, that.lastName)
            .compare(this.firstName, that.firstName)
            .compare(this.zipCode, that.zipCode)
            .result();
    }
}
```

确实简洁了许多~~


## <span class="section-num">5</span> 总结 {#总结}

到本文为止, Guava提供的基本工具类就已经介绍完了，暂时告一段落了,
接下来就要介绍Guava最常用的工具之一: 各种容器(Collections)

<div center class="qr-container">
<img src="/ox-hugo/qrcode_gh_e06d750e626f_1.jpg" alt="qrcode_gh_e06d750e626f_1.jpg" width="160px" height="160px" center="t" class="qr-container" />
公号同步更新，欢迎关注👻
</div>

