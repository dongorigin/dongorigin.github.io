---
title: ARTS-2
date: 2020-06-21 23:02:22
tags:
---

# Algorithm

朋友最近找工作，分享了一个他觉得有意思可扩展的题目

比较两个超大文本文件，保存两者相同的行到新文件中。其中文本文件远大于内存，比如内存 4G，文本文件是 16G。每行的字符数不超过 100，每行内容是随机并且均匀分布的。如何尽可能快的实现？

先不考虑内存，这类比较问题为了避免 m * n 的复杂度，要么排序，要么 Hash 用空间换时间。题目希望速度快，所以尝试用 Hash。
文件 A 逐行读取，存入 Map 中，key 是此行字符串的 Hash，value 是此行字符串。一共遍历一遍 n
文件 B 逐行读取，计算 Hash 后到 Map 中查找是否存在。一共遍历一遍 m
最终时间复杂度 n+m，空间复杂度 n。

考虑内存不够后，显然需要分块处理，但是如何分块没能想到。按顺序分不行，因为不确定两者相同部分分别在哪一块，每次都需要比较所有的块。真实面试的话我应该就到此为止了。朋友说了思路，我再重新梳理下。

关键思路是是利用 Hash 特性来分块，使两个文件的相同行分到同一个分块中，同时保证每个分块足够小能够放到内存中，那接下来的事情就和不考虑内存的情况相同了，一个读入 Map，另一个遍历一遍即可。
如何做到？文件逐行读取，求 Hash 值，然后对 Hash 值进行**求余**，根据求余结果分块存储，余数相同的放到同一块。这样 Hash 近似的行都会分到同一块中！而余数只需要大于文件大小与内存大小的比值，即可让每块大小小于内存了。当然前提是数据是均匀的，不能 Hash 值都近似，所以这是题目条件之一。
举例，余 10 会将数据分为 10 块，每块大小大约 1.6G。比如 Hash 值 1001，余 10 得 1，2001 余 10 得 1，这两行存入分块 1。Hash 值 1002，余 10 得 2，放在分块 2。对 A B 两个文件分别处理，将得到分块 a1-a10 与 b1-b10 共 20 个分块。最后分 10 次依次比较 ai 和 bi 即可，此时文件大小小于内存，再用上面内存足够的方法即可。
最终每个文件只遍历了两遍，第一遍分块，第二遍比较，所以时间复杂度是 O(n + m)。

# Tip

**Android 的 Handler Message 回收问题**
之前用到 Message 都是一次性的，处理之后就不再用了，没遇到过问题。上周修改某个问题时，需要将一个 Handler 处理过的 Message 发给另一个 Handler 继续处理，心想 handlerB.sendMessage(msg) 就搞定了，伪代码如下。

``` Kotlin
val handlerB = object : Handler() { 
  override fun handleMessage(msg: Message) {
    when (msg.what) { ... }
  }
}

val handlerA = object : Handler() {
  override fun handleMessage(msg: Message) {
    handlerB.sendMessage(msg)
  }
}
```

但测试后发现  Handler B 居然未执行任何操作，原以为是消息没发对，但是一通 debug 后才发现 Handler B 其实接收到了 Message，但是 Message 中所有数据都为空（what=0），这是为什么呢？
我知道 Message 用了对象池技术来避免频繁创建对象，所以很快想到与对象池回收有关，但此时我才意识到，之前居然一直都不清楚 Message 是何时何地被回收的。猜测应该在 Looper 循环处理处，果不其然。

``` Java
class Looper {
  public void loop() {
    for (;;) {
      Message msg = queue.next(); // 获取消息
			...      
      msg.target.dispatchMessage(msg); // 处理消息
      ... 
      msg.recycleUnchecked(); // 回收消息
    }
  }
}
```

了解原因后，解决问题就很简单了，因为原来的 Message 会被回收，所以需要获取一个新的 Message 发给 Handler B。

``` Kotlin
val newMessage = Message.obtain(msg)
handlerB.sendMessage(newMessage)
```



# Review

[Google Android 架构指南](https://developer.android.com/jetpack/docs/guide)

我非常喜欢官方这个小指南的编排方式，首先是移动应用的典型(独特)场景，然后常见的架构原则，最后是推荐的架构，最后附赠了一组最佳实践。讲解顺序实属典范。
先是 what，通过小例子说明移动端 App 相比桌面应用有什么特殊点。引出关键问题：应用组件不该存储应用数据，以及应用组件之间不该互相依赖。
然后是 why，为什么需要这样设计，两个关键的架构原则：分离关注点和数据驱动 UI。
最后是 how，推荐的架构与如何用官方 arch 组件来实现。

# Share

这段时间做了一个字数限制需求，涉及到计算与截取，所以把字符编码相关知识补了补，以及研究了一下 Emoji 的编码，感觉还挺有意思的，下次黑牛的素材也有了。

TODO：写了一半，6.22 完成