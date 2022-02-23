+++
title = "Lambda与Java Collection有感"
description = "An Discussion abount Java Collection with Lambda Expression"
date = 2017-03-04T00:00:00+08:00
lastmod = 2022-02-23T21:28:09+08:00
tags = ["java"]
categories = ["java"]
draft = false
toc = true
+++

我平时也有浏览各类博客的习惯，毕竟三人行则必有我师嘛。今天在浏览关于Java的一个博客的时候，对博主的观点有一些不同的开发，但是困于没法在博客下评论，内容如下：
[![](/ox-hugo/argument.png)](/ox-hugo/argument.png)
所以打算聊聊Java 中Collection 这个话题。(BTW,窃以为博主对Java8 新引进的Lambda, 应该了解不足)


## <span class="section-num">1</span> Java函数式编程 {#java函数式编程}

Java8 引进了函数式编程的新特性，让Java的开发人员也可以享受函数式编程的美妙，已经有很多的文章介绍函数式了，珠玉在前，我就不赘言了。

来说说Java 的Lambda吧：Java8 对核心类库进行了改进，只要包括集合类的API和新引入的流(Stream), 流可以让开发者站在更高的 抽象层次对集合进行操作


## <span class="section-num">2</span> 流的常用操作 {#流的常用操作}


### <span class="section-num">2.1</span> collect(toList()) {#collect--tolist}

`collect(toList())` 方法可以由Stream 值生成一个List,而Stream 的of方法可以使用初始值生成新的Stream.

```java
List<String> collected= Stream.of("this","is","a","list").collect(Collectors.toList());
```


### <span class="section-num">2.2</span> map {#map}

如果有一个函数可以将一种类型的值转换成另外一种类型，map 操作就可以使用该函数，将一个流中的值转换成一个新的流
[![](/ox-hugo/java_map.png)](/ox-hugo/java_map.png)


#### <span class="section-num">2.2.1</span> 例子 {#例子}

将字符变成大写格式如果用没有Lambda 时的模式编程

```java
List<String> oldStyle=new ArrayList<>();
for(String string : Arrays.asList("a","b","c")){
    String uppercaseString=string.toUpperCase();
    oldStyle.add(uppercaseString);
}
```

但是如果你有了Lambda

```java
List<String> lambdaStyle=Stream.of("a","b","c").map(string -> string.toUpperCase())
    .collect(Collectors.toList());
```

真的有种说不出的优雅


### <span class="section-num">2.3</span> filter {#filter}

遍历数据并检查其中的元素

{{< figure src="/ox-hugo/java_filter.png" link="/ox-hugo/java_filter.png" >}}


#### <span class="section-num">2.3.1</span> 例子 {#例子}

你有一个User类，然后你想找出年龄大于30岁的用户

```java
private static List<User> users = Arrays.asList(
						new User(1, "Steve", "Vai", 40),
						new User(4, "Joe", "Smith", 32),
						new User(3, "Steve", "Johnson", 57),
						new User(9, "Mike", "Stevens", 18),
						new User(10, "George", "Armstrong", 24),
						new User(2, "Jim", "Smith", 40),
						new User(8, "Chuck", "Schneider", 34),
						new User(5, "Jorje", "Gonzales", 22),
						new User(6, "Jane", "Michaels", 47),
						new User(7, "Kim", "Berlie", 60)
						);
```

非函数式编程(旧式):

```java
List<User> olderUsers = new ArrayList<User>();
for (User u : users) {
    if (u.age > 30) {
	olderUsers.add(u);
    }
}
```

函数式编程：

```java
List<User> olderUsers = users.stream().filter(u -> u.age > 30).collect(Collectors.toList());
```


### <span class="section-num">2.4</span> flatMap {#flatmap}

flatMap 方法可用Stream 替换值，然后将多个Stream 连接成一个Stream
[![](/ox-hugo/java_flatmap.png)](/ox-hugo/java_flatmap.png)


#### <span class="section-num">2.4.1</span> 例子 {#例子}

假设有一个包含多个列表的流，希望得到所有数字的序列

```java
List<Integer> together=Stream.of(Arrays.asList(1,2),Arrays.asList(3,4))
    .flatMap(numbers->numbers.stream()).collect(Collectors.toList());
```

---

还有其他常用的操作，我就不一一列举了，官方[Quick Start](http://www.oracle.com/webfolder/technetwork/tutorials/obe/java/Lambda-QuickStart/index.html)有更详细的介绍。

但是就我谈到的几种操作，应该可以对那位博主朋友的博文做出回应了，最有效优雅过滤一个Collection 的方法，我觉得是Stream 的filter

```java
List<String> filterExample=Stream.of("a","b","c").filter(string->string.equals("a")).collect(Collectors.toList());
```


### <span class="section-num">2.5</span> 小结 {#小结}

Java Lambda的特性如果不经常使用，很容易又忘了，本文就当是对Java Lambda 的一次review吧

不过，函数式的引用的确让Java 焕发出新的活力，记得之前一位前辈吐嘈Java语法太啰嗦，现在前辈应该会用得舒心一点吧

---

备注：上面的图都是来自《Java8 函数式编程》 一书

参考：Java8 函数式编程
