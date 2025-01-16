+++
title = "为Java瘦身 – Lombok"
description = "An introduction about lombok"
date = 2017-05-24T00:00:00+08:00
lastmod = 2022-02-24T15:34:18+08:00
tags = ["java"]
categories = ["java"]
draft = false
toc = true
+++

## <span class="section-num">1</span> 前言 {#前言}

几天前 Goolge 在 I/O 大会上宣布了 Android 将官方支持 Kotlin, 这意味着 Android开发者可以更好地使用 Kotlin 开发 Android.

我虽不是 Android 开发者，但是也为 Android 开发者多了一个选择而感到高兴，略显意外的是，接下来到处可以看到 "Java已死，Kotlin 当立" 之类的言论。

一群人围在一起诉说被 Java "折磨" 的血泪史，然后为Kotlin 的到来欢欣鼓舞。我学过挺多的语言，也并不是一个 "Java 卫道士".

但是看到很多人都说 "Java 的语法啰嗦，每次都要编写一大段 "Setter/Getter" 这类的模板代码，还有各种的 Bean; 真的好累" 我就觉得其实很多人都是人云亦云，他们也并没有对 Java 有多少关注。

其实 Java8 发布以后，使用 Java8 的函数式进行编码已经可以减少很多代码了；其次，一个新颖的类库也可以帮 Java 的代码进行瘦身 -- Lombok


## <span class="section-num">2</span> Lombok {#lombok}


### <span class="section-num">2.1</span> 简介 {#简介}

很多开发者都对模板代码嗤之以鼻，但是 Java 中就有很多非常类似且改动很少的样板 代码。

这问题一方面是由于类库的设计决定，另外一方面也是 Java 的自身语言的特性。 而 Lombok 这个项目就是希望通过注解来减少模板代码。

就注解而言，大多是各类框架用于生成代码 (典型的就是 Spring 和 Hibernate 了)，而很少直接使用注解生成的代
码。

因为如果想要直接在程序中使用注解生成的代码，就意味着在代码进行编译之前，注解就要进行相应的处理。这似乎是没可能发生的事情: 在编译代码之前使用编译后生成的代码。

但是 Lombok 在 IDE 的配合下就真的做到在开发的时候就插入相应的代码。


### <span class="section-num">2.2</span> @Getter and @Setter {#getter-and-setter}

百闻不如一见，还是直接看例子吧。

对于使用过 Java 的开发者而言，我相信他们最熟悉的肯定是 Java 无处不在的封装以及对应的 Getter/Setter. 现在就来看一下如何 为最常见的模板代码瘦身。

未使用 Lombok 的代码：

```java
private boolean employed = true;
private String name;

public boolean isEmployed() {
    return employed;
}

public void setEmployed(final boolean employed) {
    this.employed = employed;
}

protected void setName(final String name) {
    this.name = name;
}
```

并没有上面特点，还是 "旧把式". 那么现在来看一下使用了 Lombok 的同等作用的代
码：

```java
@Getter @Setter private boolean employed = true;
@Setter(AccessLevel.PROTECTED) private String name;
```

除了必要的变量定义以及 `@Getter`, `@Setter` 注解以外，没有了其他的东西的。

但是使用 Lombok 的代码比原生的 Java 代码少了很多行，定义的类的属性越多，减少的代码数就越可观。

而 @Getter 和 @Setter 注解的作用就是为一个类的属性生成 getter 和 setter 方法，而这些生成的方法跟我们自己编写的代码是一样的。


### <span class="section-num">2.3</span> @NonNull {#nonnull}

相信每一个使用过 Java 的开发者都不会对空指针这个异常陌生吧，因为 NullPointException 导致了各种 Bug, 以至于它的发明者 Tony Hoare 都自嘲到他创 造了价值十亿的错误 ("Null Reference: The Billion Dollar Mistake").

因此在Java 的代码中，出于安全性的考虑，对于可能出现空指针的地方，都需要进行空指针 检查，自然无可避免地产生了很多的模板代码。

而 Lombok 引入的 @NonNull 注解可以让需要进行空指针检查的代码 fast-fail; 这样就无需显示添加空指针检查了。

当为类 的属性添加了 @NonNull 注解以后，在对应的 setter 函数，Lombok 也会生成对应的 空指针检查。例子：使用 Lombok 对 `Family` 类添加注解：

```java
@Getter @Setter @NonNull
private List<Person> members;
```

对应的相同作用的原生代码：

```java
@NonNull
private List<Person> members;

public Family(@NonNull final List<Person> members) {
    if (members == null) throw new java.lang.NullPointerException("members");
    this.members = members;
}

@NonNull
public List<Person> getMembers() {
    return members;
}

public void setMembers(@NonNull final List<Person> members) {
    if (members == null) throw new java.lang.NullPointerException("members");
    this.members = members;
}
```

不得不感慨，使用 Lombok, 敲击键盘的次数都成指数级下降.


### <span class="section-num">2.4</span> @EqualsAndHashCode {#equalsandhashcode}

因为 Java 的 Object 类存在用于比较的 `equals()` 以及对应的 `hashCode()` 方法，而很多类都经常需要重写这两个方法来实现比较操作。

比较的操作大多是逐一比较子类的属性，而计算 hash 值的函数也基本是逐一取各个属性的 hash 值，然后与固定值相乘在相加. 这样的操作并不需要复杂算法，完成的都是重复性的 "体力活".

幸运的是， Lombok 也提供了相应的注解来减少这些模板代码。类级别的 `@EqualsAndHashCode` 注解可以为指定的属性生成 `equals()` 方法和 `hashCode()` 方法。

默认情况下，所有非静态或者没被标注成 `transient` 的属性都会被 `equals()` 和 `hashCode()` 方 法包含在内。当然，你也可以使用 `exclude` 声明不需要被包含的属性。

例子：使用了 `@EqualAndHashCode` 注解的代码

```java
@EqualsAndHashCode(callSuper=true,exclude={"address","city","state","zip"})
public class Person extends SentientBeing {
    enum Gender { Male, Female }

    @NonNull private String name;
    @NonNull private Gender gender;

    private String ssn;
    private String address;
    private String city;
    private String state;
    private String zip;
}
```

对应的相同作用的原生代码：

```java
public class Person extends SentientBeing {

    enum Gender {
	/*public static final*/ Male /* = new Gender() */,
	/*public static final*/ Female /* = new Gender() */;
    }
    @NonNull
    private String name;
    @NonNull
    private Gender gender;
    private String ssn;
    private String address;
    private String city;
    private String state;
    private String zip;

    @java.lang.Override
	public boolean equals(final java.lang.Object o) {
	if (o == this) return true;
	if (o == null) return false;
	if (o.getClass() != this.getClass()) return false;
	if (!super.equals(o)) return false;
	final Person other = (Person)o;
	if (this.name == null ?
	    other.name != null : !this.name.equals(other.name)) return false;
	if (this.gender == null ?
	    other.gender != null : !this.gender.equals(other.gender)) return false;
	if (this.ssn == null ?
	    other.ssn != null : !this.ssn.equals(other.ssn)) return false;
	return true;
    }

    @java.lang.Override
	public int hashCode() {
	final int PRIME = 31;
	int result = 1;
	result = result * PRIME + super.hashCode();
	result = result * PRIME + (this.name == null ?
				   0 : this.name.hashCode());
	result = result * PRIME + (this.gender == null ?
				   0 : this.gender.hashCode());
	result = result * PRIME + (this.ssn == null ?
				   0 : this.ssn.hashCode());
	return result;
    }
}

```

模板代码和使用了 Lombok 的代码简洁程度而言，差距越来越大了


### <span class="section-num">2.5</span> @Data {#data}

下面我就来介绍一下在我项目中使用最频繁的注解 `@Data`.

使用 `@Data` 相当于同时在类级别使用 `@EqualAndHashCode` 注解以及我未曾提及的 `@ToString` 注解 (这个应该可以从注解名字猜出注解的作用), 以及为每一个类的属性添加上 `@Setter` 和 `@Getter` 注解。

在一个类使用 `@Data`, Lombok 还会为该类生成构造函数。

例子：使用 了 `@Data` 注解的函数：

```java
@Data(staticConstructor="of")
public class Company {
    private final Person founder;
    private String name;
    private List<Person> employees;
}
```

同等作用的原生 Java 代码：

```java
public class Company {
    private final Person founder;
    private String name;
    private List<Person> employees;

    private Company(final Person founder) {
	this.founder = founder;
    }

    public static Company of(final Person founder) {
	return new Company(founder);
    }

    public Person getFounder() {
	return founder;
    }

    public String getName() {
	return name;
    }

    public void setName(final String name) {
	this.name = name;
    }

    public List<Person> getEmployees() {
	return employees;
    }

    public void setEmployees(final List<Person> employees) {
	this.employees = employees;
    }

    @java.lang.Override
	public boolean equals(final java.lang.Object o) {
	if (o == this) {
	    return true;
	}
	if (o == null) {
	    return false;
	}
	if (o.getClass() != this.getClass()) {
	    return false;
	}
	final Company other = (Company) o;
	if (this.founder == null ?
	    other.founder != null : !this.founder.equals(other.founder)) {
	    return false;
	}
	if (this.name == null ?
	    other.name != null : !this.name.equals(other.name)) {
	    return false;
	}
	if (this.employees == null ?
	    other.employees != null : !this.employees.equals(other.employees)) {
	    return false;
	}
	return true;
    }

    @java.lang.Override
	public int hashCode() {
	final int PRIME = 31;
	int result = 1;
	result = result * PRIME + (this.founder == null ?
				   0 : this.founder.hashCode());
	result = result * PRIME + (this.name == null ?
				   0 : this.name.hashCode());
	result = result * PRIME + (this.employees == null ?
				   0 : this.employees.hashCode());
	return result;
    }

    @java.lang.Override
	public java.lang.String toString() {
	return "Company(founder=" + founder + ", name=" + name + "," +
	    " employees=" + employees + ")";
    }
}

```

差别更加显而易见了


## <span class="section-num">3</span> 小结 {#小结}

我在日常的开发中，使用的开发语言主要是 Java, 我也学习过其他的语言，所以 Java 和其他语言相比的优缺点也了然于心。

Java 绝佳的工程性，优秀的 OOP 范式，以及大量的类库，框架 (例如 Spring "全家桶"), 以及 JIT 带来的接近 C++ 的性能，但是 Java 语法实在啰嗦，需要编写很多的模板代码，以至于经常出现将小项目写成中项目，中项目写成大项目的烦恼，更被戏称为 "搬砖".

现在看来，Lombok 为 Java 减少的模板代码实在算是造福 Java 开发者，让开发者在获得 Java 优势的时候，还可以尽量少地打字，可谓是来的及时。

---


## <span class="section-num">4</span> 参考 {#参考}

-   <https://projectlombok.org/index.html>

<div center class="qr-container">
<img src="/ox-hugo/qrcode_gh_e06d750e626f_1.jpg" alt="qrcode_gh_e06d750e626f_1.jpg" width="160px" height="160px" center="t" class="qr-container" />
公号同步更新，欢迎关注👻
</div>

