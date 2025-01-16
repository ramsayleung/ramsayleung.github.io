+++
title = "Maven 小记"
date = 2018-10-23T12:15:00+08:00
lastmod = 2022-02-25T07:59:43+08:00
tags = ["java", "maven"]
categories = ["java"]
draft = false
toc = true
+++

Maven 在工作中的经验以及《Maven 实战》读后感


## <span class="section-num">1</span> 前言 {#前言}

蚂蚁金服的伯岩大大曾经说 [Java 生态都太重量级，连Maven 都是怪兽级的构建工具，需要整整一本书来讲解](http://blog.fnil.net/index.php/archives/70/). 平心而论，Maven 的确如此, 但是无论是怪兽级，还是迪迦级的工具，只要能把事情做好了就是好工具, 而 Maven 恰恰就是这样的工具


## <span class="section-num">2</span> 配置文件 {#配置文件}


### <span class="section-num">2.1</span> pom.xml {#pom-dot-xml}

就好像 Unix 平台的 Make 对应的 `MakeFile`，Cmake对应的 `CmakeFile.txt`, Maven 项目的核心是 `pom.xml`, POM(Project Object Model,项目对象模型)定义了项目的基本信息，用于描述项目如何构建，声明项目依赖等等，可以 pom.xml 是 Maven 一切实践的基础


## <span class="section-num">3</span> 依赖管理 {#依赖管理}


### <span class="section-num">3.1</span> 坐标 {#坐标}

Maven 仓库中有成千上万个构件（jar，war 等文件），Maven 如何精确地找到用户所需的构件呢，用的就是坐标。说起坐标，可能第一反映是平面几何中的 x，y坐标，通过 x，y坐标来唯一确认平面中的一个点，而Maven 的坐标就是用来唯一标识一个构件。

Maven 通过坐标为构件引入了秩序，任何一个构件都需要明确定义自己的坐标，而坐标是由以下元素组成：`groupId`, `artifactId`, `version`, `packaging`, `classifier`, `scope`, `exclusions`等。一个典型的Maven 坐标：

```xml
<dependency>
  <groupId>springframework</groupId>
  <artifactId>spring-beans</artifactId>
  <version>1.2.6</version>
</dependency>
```

坐标元素详解:

-   groupId(必填): 定义当前Maven 项目隶属的实际项目, 一般是域名的方向定义
-   artifactId(必填): 定义实际项目中的一个Maven 项目，推荐的做法是使用实际项目名称作为 `artifactId` 的前缀, 比如上例的 `artifactId` 是 spring-beans，使用了实际项目名 spring 作为前缀
-   version(必填): 定义了Maven 项目当前所处的版本，如上例版本是 1.2.6
-   packaging(选填): 定义了Maven 项目的打包方式。打包方式和所生成的构建的文件扩展名对应，如果上例增加了`<packaging>jar</packaging>`元素，最终的文件名为spring-beans-1.2.6.jar(Maven 打包方式默认是 jar)，如果是 web 构件，打包方式就是 `war`，生成的构件将会以`.war` 结尾
-   classifier: 用来帮助定义构建输出的一些附属构件. 附属构建和主构件对应，如上例的主构件是`spring-beans-1.2.6.jar`, 这个项目还会通过使用一些插件生成\`=spring-beans-1.2.6-doc.jar=, `spring-beans-1.2.6-source.jar`, 其中包含文档和源码
-   exclusions: 用来排除依赖
-   scope: 定义了依赖范围，例如 `junit` 常见的scope 就是`<scope>test</scope>`, 表示这个依赖只对测试生效


### <span class="section-num">3.2</span> 依赖范围 {#依赖范围}

上文提到，JUnit 依赖的测试范围是test，测试范围用元素scope 表示。首先需要知道，Maven 在编译项目主代码的时候需要使用一套classpath，上例在编译项目主代码的时候就会用到`spring-beans`，该文件以依赖的方式呗引入到classpath 中。

其次，Maven 在执行测试时候会使用另外一套 classpath。如上文提到的 JUnit 就是以依赖的方式引入到测试使用的 classpath，需注意的是这里的依赖范围是`test`. 最后，项目在运行的时候，又会使用另外一套的 classpath，上例的`spring-beans`就是在该classpath里，而JUnit 则不需要。

简而言之，依赖范围就是用来控制依赖与这是那种 classpath (编译classpath, 测试 classpath, 运行 classpath 的关系，Maven 有以下几种依赖范围:

-   compile: 编译依赖范围，如果没有显式指定`scope`, 那么`compile`就是默认依赖范围，使用此依赖范围的Maven 依赖，对于编译，测试，运行三种 classpath 都是有效的
-   test: 测试依赖范围，指定了该范围的依赖，只对测试 classpath 有效，在编译或者运行项目的时候，无法使用该依赖；典型例子就是 JUnit
-   provided: 已提供依赖范围。使用此依赖范围的 Maven 依赖，对于编译和测试classpath 有效，但在运行时无效
-   runtime：运行时依赖范围。使用此依赖范围的 Maven 依赖，对于测试和运行的classpath 有效，但在编译主代码时无效
-   import: 导入依赖范围，该依赖范围不会对三种 classpath 产生实际的影响
-   system: 系统依赖方位。与 provided 依赖范围完全一致, 即只对编译和测试的classpath有效，对运行时的 classpath 无效. 但是，使用system 范围的依赖必须通过systemPath 元素显式地指定依赖文件的路径 如：

<!--listend-->

```xml
<dependency>
  <groupId>javax.sql</groupId>
  <artifactId>jdbc-stdext</artifactId>
  <version>2.0</version>
  <scope>system</scope>
  <systemPath>${java.home}/lib/rt.jar</systemPath>
</dependency>
```

**由于此类依赖不是通过Maven 仓库解析的，而且往往与本机系统绑定，可能造成构建的不可移植，因此应该谨慎使用**
上述除import 以外的各种依赖范围与三种classpath 的关系如下:

| 依赖范围 scope | 对于编译classpath有效 | 对于测试classpath 有效 | 对于运行时classpath有效 | 例子            |
|------------|-----------------|------------------|------------------|---------------|
| compile    | Y               | Y                | Y                | spring-core     |
| test       | --              | Y                | --               | JUnit           |
| provided   | Y               | Y                | --               | servlet-apt     |
| runtime    | --              | Y                | Y                | JDBC 驱动实现   |
| system     | Y               | Y                | --               | 本地的，java类库以外的文件 |


## <span class="section-num">4</span> 仓库 {#仓库}

上文提及了依赖管理，通过声明的方式指定所需的构件，那么是从哪里获取所需的构件的呢？答案是 Maven 仓库，Maven 仓库可以分为两类: 本地仓库和远程仓库。

当 Maven 需要根据坐标寻找构件的时候，它首先会查找本地仓库，如果本地仓库存在该构件，则直接使用，如果本地不存在该构件，或者需要查看是否有更新的构件版本，Maven 聚会去远程仓库查找，发现需要的构件之后，下载到本地仓库在使用.

如果本地和远程仓库都没有所需要的构件，那么 Maven 就会报错。如果需要细化远程仓库的类型，还可以分成中央仓库，私服和其他公共库。

-   中央仓库：Maven 核心自带的的远程仓库，它包含了绝大部分开源的构件。在默认的配置下，当本地仓库没有 Maven 需要的构件的时候，它就会尝试从中央仓库下载。
-   私服：为了节省带宽和时间，可以在内网假设一个特殊的仓库服务器，用来代理所有的外部的远程仓库

{{< figure src="https://raw.githubusercontent.com/ramsayleung/images/master/20191104112131.png" caption="<span class=\"figure-number\">Figure 1: </span>repo" >}}


### <span class="section-num">4.1</span> SNAPSHOT {#snapshot}

在Maven 的世界中，任何一个项目或者构件都必须有自己的版本，版本可能是 `1.0.0`, `1.0-alpha-4,2.1-SNAPSHOT` 或者 `2.1-20181028-11`, 其中 `1.0.0`, `1.0-alpha-4` 是稳定的发布版本，而 `2.1-SNAPSHOT` 或者 `2.1-20181028-11` 是稳定的快照版本。

Maven 为什么要区分快照版本和发布版本呢？难道1.0.0 不能解决么？为什么需要2.0-SNAPSHOT。

我对此 `SNAPSHOT` 这个特性印象非常深刻，在蚂蚁金服的新人培训中，其中就有一项是大家协作完成一个 Mini Alipay，一个 Mini Alipay 分成三个应用`bkonebusiness`, `bkoneuser`, `bkoneacccount`，以SOA 的架构进行拆分，应用之间相互依赖。

在开发过程中，`bkoneuser` 经常需要将最新的构件共享 `bkonebusiness`, 以供他们进行测试和开发。

因为`bkoneuser`本身也在快速迭代中，为了让`bkonebusiness` 用到最新的代码，我们不断地变更版本，`1.0.1`, `1.0.2`, `1.0.3`,... `bkoneuser` 不断发版本，`bkonebusiness` 不断升版本，甚至有一次`bkoneuser` 在没有更新版本号的情况下发布了最新代码，而 `bkonebusiness` 已经有原来版本的 jar 包，所以就没有去远程仓库拉取最新的代码，就出问题了....

其实 Maven 快照版本就是为了解决这种问题，防止滥用版本号和及时拉取最新代码。

bkoneuser 只需将版本指定为`1.0.1-SNAPSHOT`, 然后发布到远程服务器，在发布的工程中，Maven 会自动为构件打上时间戳，比如 `1.0.1-20181028.120112-13` 表示 2018年10月28号的12点01分12秒的13次快照，有了时间戳，Maven 就能随时找到仓库中该构件`1.0.1-SNAPSHOT`版本的最新文件。

这是，`bkonebusiness`对于 `bkoneuser`的依赖，只要构建`bkonebusiness`，Maven就会自动从仓库中检查 `bkoneuser`的罪行构建，发现有更新便进行下载。

基于快照版本，`bkonebusiness` 可以完全不用考虑 `bkoneuser` 的构建，因为它总是拉取最新版本的 `bkoneuser`,这个是 Maven 的快照机制进行保证。

如果到了 release，就要及时将 `1.0.1-SNAPSHOT`, 否则 `bkonebusiness` 在构建发布版本的时候可能拉取到最新的有问题的版本.


### <span class="section-num">4.2</span> 仓库搜索服务 {#仓库搜索服务}

在公司开发的时候有私服，但是在开发自己项目的时候，我一般到 [SnoaType Nexus](https://repository.sonatype.org/) 找对应的构件


## <span class="section-num">5</span> 插件与生命周期 {#插件与生命周期}


### <span class="section-num">5.1</span> 何为生命周期 {#何为生命周期}

在有关 Maven 的日常使用中，命令行的输入往往就对应了生命周期，如 `mvn package` 就表示执行默认的生命周期阶段 `package`.

Maven 的生命周期是抽象的，其实际行为都由插件来完成，如`package` 阶段的任务就会有`maven-jar-plugin` 完成。

Maven的生命周期就是为了对所有的构建过程进行抽象和统一，包括项目的清理，初始化，编译，测试，打包，集成测试，验证，部署等几乎所有的构建步骤。

需要注意的是 **Maven 的生命周期是抽象的，这意味着生命周期本身不作任何实际的工作，实际的任务(如编译源代码)都交由插件来完成. 每个步骤都可以绑定一个或者多个插件行为，而且Maven 为大多数构建步骤编写并绑定了默认的插件**

例如：针对编码的插件有 `maven-compiler-plugin`,针对测试的插件有`maven-surefire-plugin` 等，用户几乎不会察觉插件的存在


### <span class="section-num">5.2</span> 三套生命周期 {#三套生命周期}

Maven 有用三套相互独立的生命周期，它们分别是`clean`, `default` , `site`. `clean` 生命周期的目的是清理项目，`default` 生命周期的目的是构件项目，而 `site` 生命周期的目的是建立项目站点


#### <span class="section-num">5.2.1</span> clean 生命周期 {#clean-生命周期}

clean 生命周期主要是清理项目，它包含三个阶段:

1.  pre-clean: 执行一些清理前需要完成的工作
2.  clean 清理上一次构造生成的文件
3.  post-clean 执行一些清理后需要完成的工作


#### <span class="section-num">5.2.2</span> default 生命周期 {#default-生命周期}

default 生命周期奠定了真正构件时所需要执行的所有步骤，它是所有生命周期最核心的部分，其包含的阶段如下：

-   validate
-   initialize
-   generate-sources
-   process-sources 处理项目主资源文件。一般来说，是对src/main/resources 目录内的内容进行变量替换的工作后，复制到项目输出的主classpath 目录中
-   generate-resources
-   process-resources
-   compile 编译项目的主源码，一般来说，是编译 src/main/java 目录下的java 文件至项目输出的主 classpath 目录中
-   process-classes
-   generate-test-sources
-   process-test-sources 处理项目测试资源文件。一般来说，是对src/test/resources 目录的内容进行变量替换等工作后，复制到项目输出的测试classpath 目录中
-   generate-test-resources
-   process-test-resources
-   test-compile 编码项目的测试代码。一般来说，是编译 src/test/java 目录下的java 文件至项目输出的测试classpath 目录中
-   process-test-classes
-   test 使用单元测试框架运行测试，测试代码不会被打包或部署
-   prepare-packae
-   package 接受编译好的代码，打包或可发布的格式，如 jar
-   pre-integration-test
-   integration-test
-   post-integration-test
-   vertify
-   install 将包安装到Maven 本地仓库，供本地其他Maven 项目使用
-   deploy 将最终的包复制到远程仓库，共其他开发人员和Maven 项目使用


#### <span class="section-num">5.2.3</span> site 生命周期 {#site-生命周期}

site 生命周期的目的是建立和发布项目站点，生命周期包含如下阶段

-   pre-site 执行一些在生成项目站点前需要完成的工作
-   site 生成项目站点文档
-   post-site 执行一些在生成项目站点之后需要完成的工作
-   site-deploy 将生成的项目站点发布到服务器上


#### <span class="section-num">5.2.4</span> 命令行和生命周期 {#命令行和生命周期}

从命令行执行Maven 任务的最主要方式就是调用 Maven的生命周期阶段。需要注意的是，各个生命周期是相互独立的，而一个生命周期的阶段是有前后依赖关系的。

下面以一些常见的Maven 命令为例，解释其执行的生命周期阶段:

-   mvn clean: 该命令调用clean 生命周期的clean 阶段。实际执行的阶段为clean 生命周期的pre-clean 和clean 阶段
-   mvn test: 该命令调用default 生命周期的test 阶段。实际执行的阶段是 default 生命周期的 validate, initialize, 直到 test 的所有阶段。这也解释了为什么在测试的时候，项目的代码能够自动得以编译
-   mvn clean install: 该命令调用 clean 生命周期的clean 阶段和default 生命周期的 install 阶段。实际执行的阶段为 clean 生命周期的 pre-clean, clean 阶段，以及default 生命周期的从validate 到 install 的所有阶段。该命令结合了两个生命周期，在执行真正的项目构建之前清理项目是一个很好的实践


## <span class="section-num">6</span> 继承 {#继承}

如bkoneuser 的项目结构所示:

{{< figure src="https://raw.githubusercontent.com/ramsayleung/images/master/20191104112215.png" caption="<span class=\"figure-number\">Figure 2: </span>bkoneuser 的项目结构" >}}

按照 DDD(Domain Driven Design) 的驱动，`bkoneuser` 下有多个对应的子模块，每个模块也是一个 Maven 项目，每个模块里面可能有相同的依赖，如 `SpringFramework` 的 `spring-core`, `spring-beans`, `spring-context` 等。

如果每个子模块都维护一份大致相同的依赖，那么就有10几份相同的依赖，这还会随着子模块的增多而变得庞大。

如果我们工程师的嗅觉, 会发现有很多的重复依赖，面对重复应该怎么办？通过抽象来减少重复代码和配置，而 Maven 提供的抽象机制就是继承(还有聚合，只是个人觉得不如继承常用).

在 OOP 中，工程师可以建立一种类的父子结构，然后在父类中声明一些字段供子类继承，这样就可以做到“一处声明，多处使用”, 类似地，我们需要创建 POM 的父子结构，然后在父POM 中声明一些供子 POM 继承，以实现“一处声明，多处使用”


### <span class="section-num">6.1</span> 配置示例 {#配置示例}

parent 的配置如下:

```xml
<groupId>com.minialipay</groupId>
<artifactId>bkgponeuser-parent</artifactId>
<version>1.0.0-SNAPSHOT</version>
<packaging>pom</packaging>

<properties>
  <java.version>1.8</java.version>
  <bkgponeaccount.common.service.facade.version>1.1.0.20180919</bkgponeaccount.common.service.facade.version>
</properties>

<modules>
  <module>app/core/service</module>
  <module>app/core/model</module>
  <module>app/biz/shared</module>
  <module>app/biz/service-impl</module>
  <module>app/common/util</module>
  <module>app/common/service/facade</module>
  <module>app/common/service/integration</module>
  <module>app/common/dal</module>
  <module>app/test</module>
</modules>
```

需要主要的关键点是parent 的 `packaging` 值必须是 `pom`, 而不是默认的 `jar`, 否则则无法进行构件.

而 `modules` 元素则是实现继承最核心的配置，通过在打包方式为 pom 的Maven 项目中声明任意数量的 `module` 来实现模块的继承, 每个 `module`的值都是一个当前POM 的相对目录，比如 `app/core/service` 就是说子模块的POM在 parent 目录的下的 `app/core/service`目录


### <span class="section-num">6.2</span> 子模块配置示例 {#子模块配置示例}

```xml
<parent>
  <groupId>com.minialipay</groupId>
  <artifactId>bkgponeuser-parent</artifactId>
  <version>1.0.0-SNAPSHOT</version>
  <relativePath>../../../pom.xml</relativePath>
</parent>
<modelVersion>4.0.0</modelVersion>

<artifactId>bkgponeuser-core-service</artifactId>
<packaging>jar</packaging>
```

上述pom 中使用 `parent` 元素来声明父模块，`parent` 下元素groupid， artifactId 和 version 指定了父模块的坐标，这三个元素是必须。

元素 `relativePath` 表示父模块POM的相对路径, `../../../pom.xml` 指父POM的位置在三级父目录上


### <span class="section-num">6.3</span> 可继承的POM 元素 {#可继承的pom-元素}

可继承元素列表及简短说明:

-   groupId: 项目Id, 坐标的核心元素
-   version：项目版本, 坐标的核心元素
-   description: 项目的描述信息
-   organization: 项目的组织信息
-   inceptionYear: 项目的创始年份
-   url: 项目的url 地址
-   developers: 项目的开发者信息
-   contributors: 项目的贡献者信息
-   distributionManagement：项目的部署配置
-   issueManagement: 项目的缺陷跟踪系统信息
-   ciManagement: 项目的持续继承系统信息
-   scm: 项目的版本控制系统信息
-   mailingLists: 项目的邮件列表信息
-   properties: 自定义的Maven 属性
-   dependencies: 项目的依赖配置
-   dependencieyManagemant: 项目的依赖管理配置
-   repositories: 项目的仓库配置
-   build: 包括项目的源码目录配置，输出目录配置，插件配置，插件管理配置等
-   reporting: 包括项目的报告输出目录配置，报告插件配置等


### <span class="section-num">6.4</span> dependencyManagement 依赖管理 {#dependencymanagement-依赖管理}

可继承列表包含了 `dependencies` 元素，说明是会被继承的，这是我们就会很容易想到将这一特性应用到 `bkoneuser-parent` 中。子模块同时依赖 `spring-beans`,=spring-context=,=fastjson= 等, 因此可以将这些依赖配置放到父模块 `bkoneuser-parent` 中，子模块就能移除这些依赖，简化配置.

这种做法可行，但是存在问题，我们可以确定现有的子模块都是需要 `spring-beans`, `spring-context` 这几个模块的，但是我们无法确定将来添加的子模块就一定需要这四个依赖.

假设将来项目中要加入一个`app/biz/product`, 但是这个模块不需要 `spring-beans`, `spring-context`, 只需要 `fastjson`, 那么继承 `bkoneuser` 就会引入不需要的依赖，这样是非常不利于项目维护的！

Maven 提供的 `dependencyManagement` 元素既能让子模块继承到父模块的依赖配置，又能保证子模块依赖使用的灵活性。在 `dependencyManagement` 元素下的依赖声明不会引入实际的依赖，不过它能够约束 `dependencies` 下的依赖使用。

例如在 `bkoneuser-parent` 用 `dependencyManagement`声明依赖:

```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>fastjson</artifactId>
      <version>1.1.33</version>
    </dependency>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.7</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
</dependencyManagement>
```

在 `app/core/service` 子模块进行引用:

```xml
<dependencies>
  <dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
  </dependency>
  <dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
  </dependency>
</dependencies>
```

子模块的`fastjson` 依赖只配置了 `groupId` 和 `artifactId`, 省去了 `version` , 而 `junit` 依赖 不仅省去了`version`, 连`scope` 都省去了。

《Maven 实战》作者强烈推荐使用这种方式，其主要原因在与在父POM 中使用 `dependencyManagement` 声明依赖能够统一规范依赖的版本，当依赖版本在父POM中声明之后，子模块在使用依赖的时候就无须声明版本，也就不会发生多个子模块使用依赖版本不一致的情况


## <span class="section-num">7</span> 依赖冲突 {#依赖冲突}

在Java 项目中，随着项目代码量的增长，各种问题就会接踵而至，jar 包冲突就是其中一个最常见的问题. jar 冲突常见的异常: `NoSuchMethodError`, `NoClassDefFoundError`


### <span class="section-num">7.1</span> 成因 {#成因}

当Maven根据pom文件作依赖分析, 发现通过直接依赖或者间接依赖, 有多个相同`groupId`, `artifactId`, 不同 `version` 的依赖时, 它会根据两点原则来筛选出唯一的一个依赖, 并最终把相应的jar包放到 classpath下:

1.  依赖路径长度: 比如应用的pom里直接依赖了A, 而A又依赖了B, 那么B对于应用来说, 就是间接依赖, 它的依赖路径长度就是2. 长度越短, 优先级越高. 当出现不同版本的依赖时, maven优先选择依赖路径短的依赖.
2.  依赖声明顺序: 当依赖路径长度相同时, POM 里谁的声明在上面, Maven 就选择谁.

<!--listend-->

```java
public class A {
    private B b =new B();
    public void func_a(){
	b.func_b();
    }
}

// 来自b-1.0.jar
public class B {
    private C c=new C();
    public void func_b(){
	c.func_c();
    }
}

// 来自c-1.0.jar
public class C{
    public void func_xxx(){

    }
    public void func_c(){

    }
}

// 来自c-1.1.jar
public class C{
    public void func_xxx(){

    }
    public void func_c1(){

    }
}

// d.1.0.jar
public class D{
    // 来自c-1.1.jar
    C c = new C()
	public void func_d(){
	    c.func_xxx();
	}
}

public class MyMain{
    public static void main(String[] args){
	new A().func_a()
	    }
}
```

应用程序里有个A类, 里面含有一个属性B, 这个B类来自 `b-1.0.jar` 包. A类有个 `func_a()` 方法, 里面会调用b类的 `func_b` 方法.B类含有一个属性C, 这个C类来自`c-1.0.jar`. B类还提供一个方法 `func_b()`, 里面调用C类的 `func_c()` 方法.

这时, 应用程序的主POM里间接依赖了 `c-1.1.jar` 包, 但是这个jar里的C类中已经把 `func_c()` 删除了.

这样由于B类使用的 `c-1.0.jar` 对于应用程序来说, 是间接依赖, 依赖路径长度是2 (A -&gt; B -&gt; C), 比应用程序主pom中间接依赖的 `c-1.1.jar` 路径(D-&gt;C)长, 最后就会被maven排掉了 (也就是应用程序的 classpath 下, 最终会保留 `c-1.1.jar`).

最后执行main函数时, 就会报 `NoSuchMethodError`, 也就是找不到C类中 `func_c()` 方法.


### <span class="section-num">7.2</span> 解决方案 {#解决方案}

强制Maven 使用`c-1.0.jar`, 也就是将`c-1.1.jar`排除掉:

```xml
<dependencies>
  <dependency>
    <groupId>com.d</groupId>
    <artifactId>d</artifactId>
    <version>1.0</version>
    <exclusions>
      <exclusion>
	<groupId>com.c</groupId>
	<artifactId>c</artifactId>
      </exclusion>
    </exclusions>
  </dependency>
</dependencies>
```

在 `d.1.0.jar` 的依赖排除 `c.1.1.jar` 的时候，不需要指定版本, 因为这个时候`d.1.0.jar` 的依赖的版本一定是 `c.1.1.jar`. 需要注意的是，如果 `d` 使用了`c.1.1.jar` 的 `func_c1()`，排掉 `c.1.1.jar` 是会报错的，因为满足了B类的 `func_c()` 就无法满足 D 类的 `func_c1()`, 这个就是著名的“菱形依赖问题”(diamond dependency problem)。

不得不说，入职的时候，遇上了各种jar 包冲突的问题，排包都排出心得. 在此推荐个排包神器, Intellij Idea 的插件：[maven helper](https://plugins.jetbrains.com/plugin/7179-maven-helper), 比手动`-verbose:class` + `mvn dependency:tree`排包方便多了


## <span class="section-num">8</span> 总结 {#总结}

的确，写到这里，必须再次承认 Maven 是怪兽级的 构建工具，但是同样无可否认的是，它出色的构建和依赖管理功能。写go 语言的时候，我多希望有个 Maven 可以用呢 ╥﹏╥...

<div center class="qr-container">
<img src="/ox-hugo/qrcode_gh_e06d750e626f_1.jpg" alt="qrcode_gh_e06d750e626f_1.jpg" width="160px" height="160px" center="t" class="qr-container" />
公号同步更新，欢迎关注👻
</div>

