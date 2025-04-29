+++
title = "How to fool the Jacoco ◜◡‾"
date = 2019-03-14T11:14:00+08:00
lastmod = 2022-02-25T09:49:13+08:00
tags = ["java", "test"]
categories = ["java"]
draft = false
toc = true
showQuote = true
+++

刷POJO类的变更行覆盖率


## <span class="section-num">1</span> 反射大法好 {#反射大法好}


### <span class="section-num">1.1</span> 背景 {#背景}

众所周知，蚂蚁对代码质量要求很高，质量红线其中一项指标就是变更行覆盖率。如果你的变更行覆盖率没有达到80%，测试同学是不会允许你上测试环境的（如果对此有所不满，测试同学就会过来捶你)。
为了提高代码质量，这项要求倒是无可厚非，变更的代码逻辑需要充分的测试；但是如果我新增了一堆的POJO类，只是为了逻辑模型，变更行也会变得非常可观。为了覆盖这些POJO类的变更，你免不了会测试一堆的Getter/Setter
方法：

{{< figure src="https://raw.githubusercontent.com/ramsayleung/images/master/20191104102036.png" caption="<span class=\"figure-number\">Figure 1: </span>getter/setter" >}}

(红色是指没有覆盖到的变更行)


### <span class="section-num">1.2</span> 反射 {#反射}

如果为了变更行覆盖了，我要写上一堆的Getter/Setter 方法测试用例，测试用例也只是单纯调用一下方法，未免过于痛苦，能否偷个懒，解决覆盖率问题，也不需手写这些没啥用的测试用例.

但是一时间没有想到解决方法，我就一边写这些没啥用的用例，一边思考，突然发现了规律：

```java
public SomeType getXxxx(){}
public void setXxxx(SomeType Xxx){}
public SomeType getYyy(){}
public void setYyyy(SomeType Yyyy){}
```

所有这些方法都是的前缀都是 `set/get` (真.废话)，如果我能获取一个Pojo类所有的方法，然后循环执行所有以`get/set`开头的方法，不就不用手动写方法了么?

```java
public class MerchantBusiModelTest {
    protected static final Logger LOGGER = LoggerFactory.getLogger(ModelUtils.class);

    /**
     * get类型方法的前缀
     */
    private static final String GET_METHOD_PREFIX = "get";

    /**
     * set类型方法的前缀
     */
    private static final String SET_METHOD_PREFIX = "set";
    MerchantBusiModel merchantBusiModel = new MerchantBusiModel();

    @Test
    public void testModel() {
	Method[] methods = merchantBusiModel.getClass().getDeclaredMethods();
	for (Method method : methods) {
	    if (Modifier.isPublic(method.getModifiers())
		&& method.getName().startsWith(GET_METHOD_PREFIX)) {
		Object[] parameters = new Object[method.getParameterCount()];
		try {
		    method.invoke(merchantBusiModel, parameters);
		    LoggerUtil.warn(LOGGER, "调用方法, method: {}.{}",
				    merchantBusiModel.getClass().getSimpleName(),
				    method.getName());
		} catch (IllegalAccessException e) {
		    LoggerUtil.warn(LOGGER, "调用方法异常, method: {}.{}", e,
				    merchantBusiModel.getClass().getName(),
				    method.getName());
		} catch (InvocationTargetException e) {
		    LoggerUtil.warn(LOGGER, "调用方法异常, method: {}.{}", e,
				    merchantBusiModel.getClass().getName(),
				    method.getName());
		}
	    }
	}
    }
}
```

这样很快就把`MerchantBusiModel`所有的get方法执行了(set
方法也同理啦)，调用结果如下:

```log
19/03/14 10:46:32 WARN util.ModelUtils: (,N,20190314104632162,-,,,-,-,-,)[调用方法, method: MerchantBusiModel.getMcc]
19/03/14 10:46:32 WARN util.ModelUtils: (,N,20190314104632256,-,,,-,-,-,)[调用方法, method: MerchantBusiModel.getOutMerchantId]
19/03/14 10:46:32 WARN util.ModelUtils: (,N,20190314104632256,-,,,-,-,-,)[调用方法, method: MerchantBusiModel.getMerchantName]
19/03/14 10:46:32 WARN util.ModelUtils: (,N,20190314104632256,-,,,-,-,-,)[调用方法, method: MerchantBusiModel.getMerhantType]
19/03/14 10:46:32 WARN util.ModelUtils: (,N,20190314104632256,-,,,-,-,-,)[调用方法, method: MerchantBusiModel.getDealType]
19/03/14 10:46:32 WARN util.ModelUtils: (,N,20190314104632257,-,,,-,-,-,)[调用方法, method: MerchantBusiModel.getAlias]
19/03/14 10:46:32 WARN util.ModelUtils: (,N,20190314104632257,-,,,-,-,-,)[调用方法, method: MerchantBusiModel.getLegalPerson]
19/03/14 10:46:32 WARN util.ModelUtils: (,N,20190314104632257,-,,,-,-,-,)[调用方法, method: MerchantBusiModel.getPrincipalCertType]
省略一大片类似的输出，省点篇幅
```


### <span class="section-num">1.3</span> org.reflections.Reflections {#org-dot-reflections-dot-reflections}

通过反射，就很完美地解决了POJO类的变更行覆盖率问题了，反正POJO类都是Getter/Setter 方法，我的反射方法能把它们全覆盖上啦 (๑&gt;◡&lt;๑) .

很快，我就遇到了另外的一个问题:
像`MerchantBusiModel`这样的Model类有十几二十个，难道每个Model我都需要写一个`XxxModelTest`的测试类么？也实在是太痛苦了，也太不优雅了(其实是我懒)，能不能自动把所有的Model类扫出来，然后循环执行每个Model的Getter/Setter方法呢？

因为这些Model都是继承一个统一的基类`BaseBusiModel`, 能否把这个基类的所有子类搞出来，这样就可以开心地用反射解决问题了.

调研一番之后发现，Jdk 的反射方式不支持遍历父类所有子类的方法，这做法行不通阿!!!

在我都几乎要放弃，要手写所有ModelTest的时候，我在StackOverFlow上面找到了 [reflections](https://github.com/ronmamo/reflections) 这第三方包，发现这个包非常强大(niubility), 可以获取基类的子类或者接口的实现类:

```java
Reflections reflections = new Reflections("my.project");
Set<Class<? extends SomeType>> subTypes = reflections.getSubTypesOf(SomeType.class);
```

简直了。在这”牛包”的帮助下，成功实现了扫描某个package下面所有基类的实现类的方法, 我的用例有救了:

```java
public class ModelTest {
    protected static final Logger LOGGER = LoggerFactory.getLogger(ConvertorTest.class);

    private static final String PACKAGE_NAME = "xxx.xxx.core.service.v1.busimodel"; // model所有的包

    @Test
    public void testModel() {
	Reflections reflections = new Reflections(PACKAGE_NAME);
	Set<Class<? extends BaseBusiModel>> classes =
	    reflections.getSubTypesOf(BaseBusiModel.class);
	for (Class<? extends BaseBusiModel> clazz : classes) {
	    if (Modifier.isAbstract(clazz.getModifiers())) {
		continue;
	    }
	    BaseBusiModel modelInstance = null;
	    try {
		modelInstance = clazz.newInstance();
	    } catch (IllegalAccessException e) {
		LoggerUtil.warn(LOGGER, "调用方法IllegalAccessException异常, clazz: {}", e,
				clazz.getName());
	    } catch (InstantiationException e) {
		LoggerUtil.warn(LOGGER, "调用方法InstantiationExceptionn异常, clazz: {}", e,
				clazz.getName());
	    }
	    ModelUtils.invokeGetAndSetMethod(modelInstance);
	}
    }
}
```

`ModelUtils.invokeGetAndSetMethod(modelInstance);` 这个静态方法就是上一节反射方法的完整可用版:

```java
public class ModelUtils {
    protected static final Logger LOGGER = LoggerFactory.getLogger(ModelUtils.class);


    /**
     * get类型方法的前缀
     */
    private static final String GET_METHOD_PREFIX = "get";

    /**
     * get类型方法的前缀
     */
    private static final String SET_METHOD_PREFIX = "set";

    /**
     * 调用clazz 对象的所有get, set方法
     *
     * @param clazz
     */
    public static void invokeGetAndSetMethod(Object clazz) {
	invokeMethodWithPrefix(GET_METHOD_PREFIX, clazz);
	invokeMethodWithPrefix(SET_METHOD_PREFIX, clazz);
    }

    /**
     * 通过方法前缀调用方法
     *
     * @param prefix
     * @param instance
     */
    public static void invokeMethodWithPrefix(String prefix, Object instance) {
	Method[] methods = instance.getClass().getDeclaredMethods();
	for (Method method : methods) {
	    if (Modifier.isPublic(method.getModifiers())
		&& method.getName().startsWith(prefix)) {
		Object[] parameters = new Object[method.getParameterCount()];
		try {
		    method.invoke(instance, parameters);
		} catch (IllegalAccessException e) {
		    LoggerUtil.warn(LOGGER, "调用方法异常, method: {}.{}", e,
				    instance.getClass().getName(), method.getName());
		} catch (InvocationTargetException e) {
		    LoggerUtil.warn(LOGGER, "调用方法异常, method: {}.{}", e,
				    instance.getClass().getName(), method.getName());
		}
	    }
	}
    }
}
```


### <span class="section-num">1.4</span> 总结 {#总结}

1.  Reflections 包是真的强，有空要去看一下源码
2.  懒惰是程序员的第一生产力, 这话真不是我编的，是Perl 语言之父 Larry
    Wall 说的
3.  加了其他两个类似功能的反射测试类，我的变更行覆盖率暴增30%
    (可以看出我这次的变更主要是新增模型和工具类，这样反射才能调用规律性代码)
4.  Java大法好，Java世界那么大，还需要我好好探索.

<div center class="qr-container">
<img src="/ox-hugo/qrcode_gh_e06d750e626f_1.jpg" alt="qrcode_gh_e06d750e626f_1.jpg" width="160px" height="160px" center="t" class="qr-container" />
公号同步更新，欢迎关注👻
</div>

