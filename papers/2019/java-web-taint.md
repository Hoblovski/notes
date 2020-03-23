# Type-based Taint Analysis for Java Web Applications

和 CFL 同样作者. 这篇 FASE'14 提出 SFlow, 而 CFL (PPPJ'14) 提出 DFlow.



# 摘要
问题: 探索做静态 taint analysis.
那么已有的缺点: 使用 data-flow 和 point-to 方法, 需要上下文敏感全局分析, 不 scalable.

方法: typed-based taint analysis.


# 总结
类似 CFL, 因此略看. 基本是一样的.

亮点/不同是
* 处理反射
* 使用 modular inference (没细看)



# Questions
1. 静态, 基于类型的 taint analysis 在 fmtstr usenix-sec'01 就有了.
  我希望看到它们的创新.

> To the best of our knowledge, this is the first type-based taint analysis for
> Java web applications, as well as the first analysis that is low polynomial and
> yet precise.

  好吧他们引用了这篇, 然后指出 1. 没有 polymorphism 2. CQual 使用 point-to 分析.

2. 称 context-sensitive, 那么是什么 context?
一个是 qualifier 多态 (我们的 depend-on 也是一样的), 另一个就是 viewpoint adaption.
