+++
title = "Java读写文件小结"
description = "An post about write and read file with java"
date = 2017-03-06T00:00:00+08:00
keywords = ["javaio"]
lastmod = 2022-02-23T21:27:49+08:00
tags = ["java"]
categories = ["java"]
draft = false
toc = true
+++

今天在完成《算法》上的练习的时候，要对文件进行读写，而书上的例子是直接通过 Linux/Unix的重定向来实现的，我要把它修改成直接读取文件。

此外，个人一直觉得Java IO 很容易混淆，因为有太多的选择(但是这也是Java 的强大之处)，现在Java8 又新增了文件的API,所以我就对文件IO作了个小结


## <span class="section-num">1</span> Read {#read}

我今天的需求是要逐行读写文本文件，我就以此为例子了；测试文件是 `/tmp/test.txt`

```text
test
this is a test
this is another test
```


### <span class="section-num">1.1</span> BufferedReader {#bufferedreader}

虽然已经有了Java8的 `Stream`, 但是经典的东西总是历久弥新的；例如 `BufferedReader`就是JDK1.1就发布了的文件读API (对可能出现的IOException,使用更优雅try-with-resource并免去编写大量手动关闭资源的模板代码的麻烦)

```java
public  void testBufferedReader(){
    String filePath="/tmp/test.txt";
    try(BufferedReader bufferedReader=new BufferedReader(new FileReader(filePath))){
	String line;
	while((line=bufferedReader.readLine())!=null){
	    System.out.println(line);
	}
    }catch (IOException ex){
	ex.printStackTrace();
	//do something
    }
```


### <span class="section-num">1.2</span> Scanner {#scanner}

对发布于JDK1.5的Scanner,大部份Java 程序员都是相当熟悉的，因为总是用它来读取标准输入的数据。 现在只要把从标准输入变为从文件读取数据就可以了

```java
public  void testScanner(){
    String filePath="/tmp/test.txt";
    try(Scanner scanner=new Scanner(new File(filePath))){
	while(scanner.hasNextLine()){
	    System.out.println(scanner.nextLine());
	}
    }catch (IOException ex){
	ex.printStackTrace();
	//do something
    }
}
```


### <span class="section-num">1.3</span> BufferedReader+Stream {#bufferedreader-plus-stream}

`Files` 类作为Java NIO 的一部分在Java 7被引入，该类提供了一系列操作文件的方法，而在Java8 又引入了另外有用的特性让Java 开发者可以更方便地操作文件。

例如 `lines()` 方法，可以让 `BufferedReader` 可以把文件内容以 `Stream` 的形式返回；读取文件， 并把文件内容存储到 `ArrayList`.

```java
public void testBufferedReaderAndStream(){
    String filePath="/tmp/test.txt";
    List<String> list=new ArrayList<>();
    try(BufferedReader bufferedReader= Files.newBufferedReader(Paths.get(filePath))){
	list=bufferedReader.lines().collect(Collectors.toList());
    }catch (IOException ex){
	ex.printStackTrace();
	//do something
    }
}
```

得益于强大的 `Stream` 你可以在读取文件是进行更多的操作；例如只存储含有 `this` 字符的行并且删除结尾的空白符

```java
public  void testBufferedReaderAndStream(){
    String filePath="/tmp/test.txt";
    List<String> list=new ArrayList<>();
    try(BufferedReader bufferedReader= Files.newBufferedReader(Paths.get(filePath))){
	bufferedReader.lines().filter(line->line.contains("this")).map(String::trim)
	    .forEach(System.out::println);
    }catch (IOException ex){
	ex.printStackTrace();
	//do something
    }
}
```


### <span class="section-num">1.4</span> lines+Stream {#lines-plus-stream}

也可以直接使用 `lines` 方法来逐行读取文本文件，只是对比 `newBufferedReader` + `Stream`, 前者颗粒度更细；

```java
public void testlinesAndStream(){
    String filePath="/tmp/test.txt";
    List<String> list=new ArrayList<>();
    try(Stream<String> stringStream=Files.lines(Paths.get(filePath))){
	stringStream.filter(line->line.contains("test")).forEach(System.out::println);
    }catch (IOException ex ){
	ex.printStackTrace();
	//do something
    }
}
```

如果你是读取不是很大的文件的时候，你可以一次就把文件都进内存； `Files` 已经为你提供这样的方法

```java
public static  void testReadAllLines(){
    String filePath="/tmp/test.txt";
    List<String> lists= null;
    try {
	lists = Files.readAllLines(Paths.get(filePath));
    } catch (IOException e) {
	e.printStackTrace();
    }
    for (String list : lists) {
	System.out.println(list);
    }
}
```

需要注意的是 `try-with-resource` 是不支持 `readAllLines` .此外大文件请慎重使用 `readAllLines`,因为你可能出现 `OutOfMemoryException`

不得不说，新加入的API的确更加优雅

---


## <span class="section-num">2</span> Write {#write}

我就把测试文件重新写到一个新的文件，实现复制的功能，因为我的文件很小，所以我直接把测试独的文件加载到内存


### <span class="section-num">2.1</span> BufferedWriter {#bufferedwriter}

与 `BufferedReader` 对应，对文件进行写

```java
public void testBufferedWriter() {
    String readFilePath = "/tmp/test.txt";
    String writeFilePath = "/tmp/test1.txt";
    try {
	List<String> lines = Files.readAllLines(Paths.get(readFilePath));
	try (BufferedWriter bufferedWriter = new BufferedWriter(new FileWriter(writeFilePath))) {
	    for (String line : lines) {
		bufferedWriter.write(line+"\n");
	    }
	} catch (IOException ex) {
	    ex.printStackTrace();
	    //do something
	}
    } catch (IOException ex) {
	ex.printStackTrace();
	//do something
    }
}
```

你也可以将 `BufferedReader` 和 `Files` 结合

```java
public static void testBufferedWriterAndFiles() {
    String readFilePath = "/tmp/test.txt";
    String writeFilePath = "/tmp/test1.txt";
    try {
	List<String> lines = Files.readAllLines(Paths.get(readFilePath));
	try (BufferedWriter bufferedWriter = Files.newBufferedWriter(Paths.get(writeFilePath))) {
	    for (String line : lines) {
		bufferedWriter.write(line + "\n");
	    }
	} catch (IOException ex) {
	    ex.printStackTrace();
	    //do something
	}
    } catch (IOException ex) {
	ex.printStackTrace();
	//do something
    }
}
```


### <span class="section-num">2.2</span> Files.write {#files-dot-write}

使用 `Files.write()` 也可以写出相当优雅的代码

```java
public  void testFilesWrite() {
    String readFilePath = "/tmp/test.txt";
    String writeFilePath = "/tmp/test1.txt";
    try {
	List<String> lines = Files.readAllLines(Paths.get(readFilePath));
	Files.write(Paths.get(writeFilePath), lines);
    } catch (IOException ex) {
	ex.printStackTrace();
	//do something
    }
}
```

---

这就是各种对文本文件进行读写的方法；不知道为什么，我觉得似乎写文件的方法似乎比读文件的方法少，例如读文件有 `Scanner` , 而写文件似乎没有 `Printer` :(

不应该是匹配的么，或许我是不知道？

Enjoy Java :)


## <span class="section-num">3</span> 参考 {#参考}

-   <http://winterbe.com/posts/2015/03/25/java8-examples-string-number-math-files/>
-   <http://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#lines-java.nio.file.Path->
-   <https://docs.oracle.com/javase/8/docs/api/java/util/stream/package-summary.html>
