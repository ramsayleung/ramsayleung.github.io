+++
title = "重新造轮子系列(六)：构建工具"
date = 2025-04-20T18:18:00-07:00
lastmod = 2025-04-21T22:53:44-07:00
tags = ["reinvent"]
categories = ["ReInvent: 重新造轮子系列"]
draft = false
toc = true
+++

项目 GitHub 地址: [Build Manager](https://github.com/ramsayleung/reinvent/tree/master/build_manager)


## <span class="section-num">1</span> 前言 {#前言}

以 C 语言为例，一个程序通常由多个源文件 `.c` 组成, 每个源文件需要先编译成目标文件 `.o`, 再链接成最终的可执行文件。

{{< figure src="/ox-hugo/c_program_build_phase.jpg" >}}

如果只改动了其中一个源文件的内容，理想情况只需要重新编译并重新链接改动文件，而非从头构建整个项目(所谓的增量编译)。

但是如果手动管理这些依赖关系，随着源文件的增多，很容易就会变得无法维护。

类似的问题在软件开发中比比皆是，以我们此前实现的[「模板引擎」]({{< relref "reinvent_page_template" >}})为例，如果我们正使用静态网站生成器来构建个人博客(Hugo 或者 Jekyll),
当我们修改了某篇文章时，系统只需要重新生成该页面，而不必重新编译整个网站。

但如果我们调整了调整了网站的模板，那么所有依赖该模板的页面都需要重新渲染，而手动处理这些依赖关系既繁琐又容易出错。

因此我们需要一个构建工具(build tool或者叫 build maanger).


## <span class="section-num">2</span> 需求 {#需求}

构建工具的核心理念就是自动化上述的操作：

1.  定义依赖关系, 比如 `main.o` 依赖于 `main.c` 和 `header.h`
2.  检测变更，可以通过时间戳或者内容的哈希来判断文件是否「过时」
3.  按需执行，只重新生成受影响的目标

从经典的 [make](https://www.gnu.org/software/make/), 到现代构建系统如 [Bazel](https://bazel.build), 尽管它们的实现方式各异，但是基本都遵循着这一思路。

所以我们会参考 `make`, 从零实现一个类 `make` 的构建工具，核心功能包括：

1.  构建规则(rule)：描述目标(target), 依赖(dependency)和生成命令(recipe)
2.  依赖图(DAG): 避免循环依赖并确定构建顺序
3.  增量编译：仅重新生成「过期」的目标
4.  变量与通配符（如 `@TARGET` 和 `%` ） 以提高灵活性.


## <span class="section-num">3</span> 设计 {#设计}


### <span class="section-num">3.1</span> 构建规则 {#构建规则}

构建工具的输入是一系列的规则，每个规则必需包含三个关键要素，分别是：

1.  目标，即构建命令生成的最终结果
2.  依赖，即目标依赖于哪些文件
3.  生成命令：具体的命令，用于描述如何将依赖生成出目标。

以 `make` 的规则文件 `Makefile` 举例:

```makefile
target: dependencies
    recipe
```

以一个 C 语言程序的构建规则为例:

```makefile
hello: utils.c main.c utils.h
        gcc main.c utils.c -o hello
```

其中 `hello` 是构建目标， `utils.c main.c utils.h` 是多个依赖文件, `gcc main.c utils.c -o hello` 是生成命令.

`Makefile` 是 `make` 专属的配置文件格式，我们可以使用 JSON 或者 YAML 作为配置文件，避免重复造轮子，只要要表达列表和嵌套关系。

那么为什么 `make` 不使用 JSON 或者 YAML 作为配置文件呢？因为 `make` 被造出来的时候(1976年)，JSON 和 YAML 离诞生还有几十年呢.


### <span class="section-num">3.2</span> 依赖图 {#依赖图}

以前学数据结构的时候，难免会觉得图(graph) 这个数据结构真的没有什么用，除了刷题和面试会被问到.

但是在开发这个构建工具的时候，会发现图是必不可少的数据结构，准确来说是有向无环图(directed acyclic graph, DAG).

在构建工具中，每个构建规则（target: dependencies）定义了文件之间的依赖关系，这些关系天然形成一个有向无环图（DAG）。

例如：

-   A 依赖于 B 和 C（ `A → B, A → C` ）
-   B 依赖于 D（ `B → D` ）

此时，\*构建顺序必须满足依赖的先后关系\*：D 必须在 B 之前构建，B 和 C 必须在 A 之前构建。

而拓扑排序的作用，正是将图中的节点排序，保证每个节点在其依赖之后被执行。

而如果依赖图中存在环（例如 A → B → A），拓扑排序会失败.

拓扑排序的经典算法有 [Kahn](https://www.youtube.com/watch?v=cIBFEhD77b4) 算法(基于入度)与 DFS 算法, 以 Kahn 算法为例, 步骤如下:

1.  先初始化一个队列，存入所有入度(in degree)为0的节点（无依赖节点）
2.  依次处理队列中的节点，并将其人图中「移除」，更新后续节点的入度
3.  若最终未处理所有节点，则说明存在环

{{< figure src="/ox-hugo/build_dependencies.png" >}}

我曾经写过一篇关于拓扑排序的[英文博客](https://ramsayleung.github.io/en/post/2022/topological_sorting/)，有兴趣可以移步阅读.


### <span class="section-num">3.3</span> 过期检测 {#过期检测}

增量编译的关键是仅重建「过期 」的目标，那么要怎么找到「过期」的目标呢？

最简单方式就是使用时间来作为判断标准，假如我们的源文件在上一次构建之后发生了修改，
那么我们就可以认为其对应的目标「过期」了，需要重新构建。

那么我们就需要记录上一次是什么时候构建的，然后再把文件最近的修改时间(last modification timestamp)作为比较, 用额外的文件来记录也太繁琐了，为此我们可以取一下巧：

把目标的生成时间作为上一次的构建时间，那么只要依赖的 `last modification timestamp` 大于目标的 `last modification timestamp`, 那么我们就可以认为其「过期」了。

这个就是 `make` 的实现方式，但是时间并不是总是可靠的，尤其是在网络环境下。

所以像 `bazel` 这样的现代构建系统，使用的就是源文件的哈希值来作为比较的标识：
即文件内容哈希值发生了变化，那么就认为发生内容变更，目标「过期」，需要重新生成。


### <span class="section-num">3.4</span> 设计模式 {#设计模式}

上文已经提到，我们构建工具的核心功能是解析构建规则, 构建依赖图，增量编译，变量与通配符匹配，那么我们可以很容易地写出对应的实现原型:

```js
loadConfig(): rules
buildGraph(rules): graph
variableExpand(graph)
incrementalBuild(graph)
```

那么要如何实现上面的原型呢？在面向对象的编程思路里，要不使用继承，或者是组合，而两者对应的设计模式分别对应[模板方法(Template Method)](https://refactoring.guru/design-patterns/template-method) 与 [策略模式 (Strategy Pattern)](https://refactoring.guru/design-patterns/strategy)

模板方法的核心思想是继承与流程固化，在父类中定义算法的整体骨架（不可变的执行流程），将某些步骤的具体实现延迟到子类，通过 ****继承**** 扩展行为。

而策略模式核心思想是组合 + 运行时替换，将算法的每个可变部分抽象为独立策略（接口），通过 ****组合**** 的方式注入到主类中。

[System Design By Example](https://third-bit.com/sdxjs/build-manager/) 原书使用的是模板方法，其实现可谓是充分展示了继承的不足：紧耦合，新增功能需要创建新子类，导致类爆炸，各种类变量在继承链传递，真的是无法维护，最后甚至「丧心病狂」地实现了八层继承，真的是完美诠释了 [Fragile base class](https://en.wikipedia.org/wiki/Fragile_base_class) 的 code smell.

{{< figure src="/ox-hugo/reinvent_template_method_inheritance_chain.png" >}}

在体现到维护与扩展 `template method` 代码的痛苦之后，我最终选择了策略模式，因为其可以实现不同策略之间的松耦合，每个策略可以独立修改和扩展，不影响其他组件；易于测试，每个策略可被单独测试。

此外，构建工具需求可能会很多样，比如支持不同的增量编译算法（时间戳与内容哈希），支持不同的配置格式(Makefile/JSON/YAML), 策略模式不需要改写核心代码即可支持这些变体，并且支持不同策略的组合。

为了方便对比两者实现的差别，我把 [template method](https://github.com/ramsayleung/reinvent/tree/master/build_manager/template_method) 和 [strategy pattern](https://github.com/ramsayleung/reinvent/tree/master/build_manager/strategy) 的实现都保留了。


### <span class="section-num">3.5</span> 自动变量 {#自动变量}

`make` 支持在 `Makefile` 中使用自动变量(Automatic Variables)来指代目标或者依赖，而无需显示将目标或者依赖名写出来，其变量含义如下:

假设目标是 `output: main.o utils.o`

| 变量 | 含义                                      | 示例                           |
|----|-----------------------------------------|------------------------------|
| `%`  | 通配符, 表示匹配任意非空字符串，通常用于模式规则（Pattern Rules）中 | `%.o: %.c` 匹配任意 `.c` 文件生成 `.o` |
| `$@` | 目标文件名                                | `output`                       |
| `$^` | 所有依赖文件                              | `main.o utils.o`               |
| `$<` | 第一个依赖文件                            | `main.o`                       |

这些自动变量可以极大简化 Makefile 的编写，避免重复输入文件名, 只不过 `$@` 这样的格式有点难以理解，我们可以定义自己的自动变量:

| 我们的自动变量  | `make` 变量 | 含义             |
|----------|-----------|----------------|
| `%`             | `%`       | 通配符, 表示匹配任意非空字符串 |
| `@TARGET`       | `$@`      | 目标文件名       |
| `@DEPENDENCIES` | `$^`      | 所有依赖文件     |
| `@DEP[0]`       | `$<`      | 第一个依赖文件   |
| `@DEP[n-1]`     |           | 第 `n` 个依赖文件 |


## <span class="section-num">4</span> 实现 {#实现}

在介绍完设计细节，实现就没有太多需要提及的内容，根据[入口函数](https://github.com/ramsayleung/reinvent/blob/master/build_manager/strategy/driver.ts)以及[单元测试](https://github.com/ramsayleung/reinvent/tree/master/__tests__/build_manager/strategy)就能理解个七七八八了。


## <span class="section-num">5</span> 示例 {#示例}

假设我们的 [src](https://github.com/ramsayleung/reinvent/tree/master/build_manager/strategy/src) 目录有如下的文件:

```sh
> tree src
src
├── Makefile
├── main.c
├── utils.c
└── utils.h
```

`main.c` 内容如下:

```c
#include "utils.h"

int main() {
  print_message("Hello from Makefile!");
  return 0;
}
```

`Makefile` 的内容如下:

```makefile
hello: utils.c main.c utils.h
        gcc main.c utils.c -o hello
varexpand_hello: utils.c main.c
        gcc $^ -o $@
clean:
        rm -f hello
cleanvar:
        rm -rf varexpand_hello
```

通过 `hello` 和 `varexpand_hello` 目标可分别生成 `hello` 与 `varexpand_hello` 的目标文件:

```sh
> make hello
gcc main.c utils.c -o hello

> ./hello
Message: Hello from Makefile!

> make varexpand_hello
gcc utils.c main.c -o varexpand_hello

> ./varexpand_hello
Message: Hello from Makefile!
```

与 `Makefile` 相同含义的构建规则 `build_c_app.yml` 如下:

```yaml
- target: hello
  depends:
  - src/utils.c
  - src/utils.h
  - src/main.c
  recipes:
  - "gcc src/main.c src/utils.c -o hello"
- target: varexpand_hello
  depends:
  - src/utils.c
  - src/main.c
  recipes:
  - "gcc @DEPENDENCIES -o @TARGET"
- target: clean
  depends: []
  recipes:
  - "rm -rf hello"
- target: cleanvar
  depends: []
  recipes:
  - "rm -rf varexpand_hello"
```

```sh
> npx tsx driver.ts build_c_app.yml # 未指定目标，构建第一个目标，对齐 make
gcc src/main.c src/utils.c -o hello

> npx tsx driver.ts build_c_app.yml hello # 生成 hello
target: hello is up to date, skipping execute the recipe

> ./hello
Message: Hello from Makefile!

> npx tsx driver.ts build_c_app.yml varexpand_hello # 生成 varexpand_hello
gcc src/utils.c src/main.c -o varexpand_hello

> ./varexpand_hello
Message: Hello from Makefile!
```

测试增量编译，重新构建 hello 目标

```sh
> npx tsx driver.ts build_c_app.yml hello
target: hello is up to date, skipping execute the recipe
```

修改 `main.c` 源码:

```c
#include "utils.h"

int main() {
  print_message("Hello from build_c_app.yml!");
  return 0;
}
```

重新编译及运行 `hello` 目标:

```sh
> npx tsx driver.ts build_c_app.yml hello
gcc src/main.c src/utils.c -o hello

> ./hello
Message: Hello from build_c_app.yml!
```


## <span class="section-num">6</span> 总结 {#总结}

终于又造了一个轮子，完成了这个类 `make` 的构建工具:

除了核心的依赖管理和增量编译，还实现了自动变量替换（如 =@TARGET=）、通配符规则和策略模式的灵活扩展。

写到这里总会忍不住地想起Unix的 KISS 原则, 即("Keep it simple, stupid!), **复杂的工具往往由简单的概念组合而成**

[回到本系列的目录]({{< relref "reinvent_project" >}})


## <span class="section-num">7</span> 参考 {#参考}

-   <https://third-bit.com/sdxjs/build-manager/>
