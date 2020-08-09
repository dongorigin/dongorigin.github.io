---
title: ARTS-5
date: 2020-08-09 20:58:47
tags:
---

# Algorithm

[46. Permutations](https://leetcode.com/problems/permutations/) 不重复数字的全排列

按正向思考，对于 M 个数，先固定第一位（M 种可能性），然后剩下 M-1 个数继续全排列。

递归的思路
前 1 位固定 + 全排列(M-1 个数)
前 N 位固定 + 全排列(N-N 个数)
前 M-1 位固定 + 全排列(1 个数) return

递归方法：`fun permute(fixedNums: List<Int>, nums: List<Int>): List<List<Int>>`

最终写出来可读性还是挺好的，不过中间会生成很多 List 对象，感觉性能不会很好。最后速度 81，内存 57，符合预期。之后再想办法优化。

# Tip

**APT in Kotlin 踩坑**

坑的本质问题

- APT 设计上是针对 Java 的，并不支持 Kotlin。Kotlin 是通过生成 stub 对象，让其看起来像 Java，从而支持 APT 的。
- Kotlin 虽然兼容 Java，但不等于编译后和 Java 一样！Kotlin 部分语法逻辑上看起来和 Java 一样，但编译后差很多。
- 综上，所有 Kotlin 编译后长得不像 Java 的地方，用 APT 都是坑。

**父类 pulic 属性有注解，子类想获取继承到的属性的注解**

父类如果是 Java 写的，用 Elements.getAllMembers() 即可。但父类是 Kotlin 则不行，为什么？一开始我也不知道原因，想到编译后可能有区别，做了如下实验。

一个 Kotlin 类 Foo，声明了一个 public 属性 bar，子类可见。

``` Kotlin
class Foo {
  @Module
  var bar = Bar()
}   
```

Kotlin 编译成字节码后反编译到 Java，即 Kotlin 的等价 Java 情况。（用 IDE 的 Kotlin 插件 Show Kotlin Bytecode 然后再 decompile 很方便查看）
问题的关键是，字段 bar 是 private 的！所以子类才获取不到！子类虽然能获取到 getter 和 setter 方法，但是注解并不在这两个方法上，而是在字段上。

``` Java
public class Foo {
  @Module
  @NotNull
  private Bar bar = new Bar();
 
  @NotNull
  public final Bar getBar() {
    return this.bar;
  }

  public final void setBar(@NotNull Bar var1) {
    Intrinsics.checkParameterIsNotNull(var1, "<set-?>");
    this.Bar = var1;
  }
}
```

Kotlin 的属性（property），虽然长得与 Java 的字段（Field）很像，但实际上抽象层级更高，属性背后可能有一个字段，也可能没有，比如 delegate 时并没有实际字段。所以更直接的理解是，属性表达了 getter 和 setter，而非字段。
所以属性的可见性，只是 getter 和 setter 的可见性，并非属性背后字段的可见性。

搞清楚问题原因，解决就很简单了，注解在字段上，字段是 private，子类获取不到，但父类自己可以获取到，这对于 APT 或反射都不成问题。

# Review

Homebrew 作者曾面试 Google 因没写出反转二叉树被拒，引起热议，两年后他在 [Quora 上回答了这个问题](https://qr.ae/pN2XNA)，下面翻译一段重点。

> 显然我写了一些值得 Google 认可的东西，不是吗？不是的，我写了一个简单的包管理器，任何人都可以写出来，实际上它很烂，并没有做好依赖管理，并没有处理好边界行为，并没有做好测试。
> 所以我没能回答好计算机科学的问题，也就不奇怪了。
> 但另一方面，我的软件非常成功，为什么？答案与计算机科学领域无关，而是我的软件一直都专注于用户体验。Homebrew 关心它的用户。当发生问题时，Homebrew 会尽最大的可能告诉你为什么，它会搜索指向 Github 上的相似问题。它在乎**你**。大多数工具只会让你困扰。而 Homebrew 会帮助你。如果它真的无能为力了，修复起来也很简单（我尽可能让修改变得简单），你可以让 Homebrew 变得更好。

给我的启发主要是 Homebrew 的用户体验确实很好，出现问题时会直接指出应该怎么做，而不是给我一堆错误代码然后需要我自己想办法找答案，或者找文档。
正好最近团队打包流程增加了更多的自动化检查，帮助我们规避问题，但同时打包遇到的问题也变多了。有小伙伴提议需要完善文档，我则提出了另一个改进方向，完善错误提示，直接在错误提示中告知为什么错了以及需要如何修复，并直接附带操作链接。我觉得这比全放在独立的文档中要高效太多了。类似的，代码本身就应该是最好的文档，大到模块则应该内置 README，不要让我去另一处根本不知道在哪里也不知道关键词的地方去搜索文档。

# Share

无