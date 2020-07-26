---
title: ARTS-4
date: 2020-07-19 22:52:39
tags: arts
---

# Algorithm

**[678. Valid Parenthesis String](https://leetcode.com/problems/valid-parenthesis-string/)**

继续括号匹配，相比基础匹配增加了通配符 *，缺什么补什么，可作为左括号或右括号或空。这意味着遍历结束时才能确定 * 代表什么。

列举几种典型情况找规律

- `*)`：遇到右括号时，可以找 * 匹配。
- `(*` 和 `(*)`：到结束时才能确定 * 情况，前者作为右括号，后者作为空，后者优先找了左括号匹配。
- `*(`、`)*`：有顺序要求。

大概方案

- 遍历：遇到右括号优先找左括号匹配，其次找 * 匹配，都找不到则失败。
- 遍历结束后：左括号找 * 匹配

具体方案

- 方案 A：为了右括号能先后匹配，左括号与`*`分别放入两个栈。遍历结束后则两个栈出栈匹配。
  写完了带入 case `*(` 才发现顺序问题，两个栈导致不知道前后顺序，无法验证顺序。
- 方案 B：为了解决顺序问题，决定还是放入同一个列表，这样记录了顺序。但实现了一半发现虽然有顺序了，但是因为列表混合了两种字符，匹配过程需要各种查找位置，逻辑很乱，方案 A 的栈思路清晰很多。于是又想到是否可以在方案 A 基础上改进。
- 方案 C：基于方案 A 的两个栈，额外记录了 index，用于最后匹配时比较顺序。

最后方案 C 一次通过了，因为吸取上次教训将边界条件考虑全了。但是中间来回换方案浪费了非常多时间，反思是编码之前应该先想清楚。



# Tip

**APT 踩坑：如何获取 Annotation 中的 class 值？**

注解类可以声明 Class 类型的属性

``` Kotlin
@Target(AnnotationTarget.CLASS)
@Retention(AnnotationRetention.SOURCE)
annotation class ServiceProvider(vararg val value: KClass<*>)
```

运行时想获取这些属性很简单，调用对象的属性即可

``` kotlin
element.getAnnotation(ServiceProvider::class.java)?.value
```

但 APT 中运行上述代码会报错 `java.lang.reflect.InvocationTargetException (no error message)`。
原因是 APT 运行在 Java 编译完成前，这是因为 APT 可能生成新的源码参与到编译中。所以 APT 运行时拿不到编译后的信息，比如 Class。

那 APT 中需要类型信息怎么办？使用 AnnotationMirror 和 TypeMirror，这里的 Mirror 我理解就是实际类型的镜像，用于在 APT 中替代无法获得的 Class。曲线救国如下：

``` kotlin
fun Element.findAnnotationMirror(annotationClass: Class<*>): AnnotationMirror? {
    return annotationMirrors.find { it.annotationType.toString() == annotationClass.name }
}

fun AnnotationMirror.findAnnotationValue(key: String): AnnotationValue? {
    return elementValues.entries
        .find { it.key.simpleName.toString() == key }
        ?.value
}

// 一个 AnnotationValue 表示 Annotation 的一个属性，所以类型是未知的，但我们是知道类型的，所以强转即可。而如果类型不对编译时就会报错，所以很安全。
fun AnnotationValue.getClasses(): List<TypeMirror> {
    return (value as List<AnnotationValue>).map { it.value as TypeMirror }
}
```

使用如下，借助 Kotlin 扩展方法总算能和调用 Class 可读性差不多

``` kotlin
val provideServices = element.findAnnotationMirror(ServiceProvider::class.java)
  ?.findAnnotationValue("value")
  ?.getClasses()
```

# Review

[Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)

<img src="https://blog.cleancoder.com/uncle-bob/images/2012-08-13-the-clean-architecture/CleanArchitecture.jpg" alt="img" style="zoom:50%;" />

架构的通识原则：独立于框架、独立于 UI、独立于数据库、独立于任何外部组件，可测试。

越接近圆心的逻辑，抽象层级越高，逻辑越稳定，比如业务对象。反之越外层变动可能性越高，比如 UI。所以依赖关系必须是由外向内的，这样才能保证变化少的部分不被影响到。

但是控制流与依赖关系是不一致的，比如业务数据变动后需要渲染到 UI，但业务不应该依赖 UI，怎么办？依赖反转为此而生，由此控制流与依赖关系无关，系统的设计者可以根据需要自由调整控制流与依赖关系。至此我才真正理解了依赖反转的目的。

# Share

《Clean Architecture》关于测试

程序很复杂，如何保证正确性？理论上大方向有两个，数学的证明方法——推导，和科学方法验证——证伪。
无论哪个方向，都需要用分解法将大型问题拆分成更小的可证明的单元。
但是 goto 语句的某些用法会导致模块无法被递归拆分成更小的可证明的单元。
而 Bohm 证明了用顺序结构、分支结构、循环结构可以构造出任何程序。
结构化编程范式，通过限制 goto 语句，保证了模块可递归拆分为可推导的单元。

但即使如此，通过数学推导来证明程序正确性的成本太高了（主要因为我太菜了），所以人们尝试了科学方法。
科学理论可以被证伪，但无法被证明。如果某个理论经过一定的努力都无法证伪，我们则认为它在当下是足够正确的。
程序的证明也是如此，程序递归降解为一系列可证明的小函数，然后单元测试视图证明这些函数是错误的。如果这些测试无法证伪这些函数，则我们认为这些函数是足够正确的，进而推导出整个程序是正确的。

软件架构的目的之一，就是让软件可以方便的进行证伪，包括模块、组件以及服务。为了达到这个目的，需要将类似结构化编程的限制应用到更高的层面上。

后面架构部分，讲到应该将系统中难以测试的部分与容易测试的部分分离。比如 GUI 通常是难以进行单元测试的，因为计算机自动检查显示内容很难。所以视图层的逻辑越简单越好，只应负责将数据填充到 GUI 上，而不应该对数据进行任何处理。这样视图层不进行测试也关系不大了。

第一次从证明和证伪的角度来看待测试，假如从科学证明的角度看，可测试性的要求是相当高的，这要求每个模块、类、函数，都是独立的可证明的单元。带来的价值也是相当高的。像验证科学定律一样验证自己的程序是正确的，这有点酷。