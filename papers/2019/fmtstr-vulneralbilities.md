# Detecting Format String Vulnerabilities with Type Qualifiers

# 背景介绍
问题:
* format string 的 *security* vulnerabilities.

论文中 page 3 包含了使用 type checking 的好处.
* familiar for programmers
* convenient error report
* type theory is well studied
* analysis easier to be sound

用了 CQual, 基于 Foster99.

Background 开始直到第 10 页把 Foster99 和 Type System 已有的研究重述了一遍?!
似乎他们的写作可以学习. 感觉他们是 idea 没有创新.
好歹我们稍微有点创新, 类似 C++ 的 const method.



# 方法
目标
* 防止 untrusted 数据作为 fmtstr 的 argument.
  使用 taint analysis. `untainted <: tainted`

如何表示 qualifier variable / lattice.
* lattice 是有限偏序集, 总可以被 $2^{\mathbb{N}}$ 给 embed.
* 如果 lattice element embed 到 $\{1, 2\}$ 那么在源代码中用 `$_1_2` 表示.

处理 pointer casts: "collapsing qualifiers".
* 将 $\alpha\, ref \; \beta\, ref\; \gamma char$ 给 cast 到 $\delta \, ref\; \epsilon void$, 那么有限制
$$
\begin{align*}
  \gamma &= \epsilon\\
  \beta &= \delta\\
  \alpha &= \delta
\end{align*}
$$

它们要处理 varargs, 因为 fmtstr 强烈依赖. 我们不用, 因为 varargs 自然的就不安全.



# 结果验证
* false positive: 源于保守的 taint analysis 以及缺乏 full polymorphism.

用包含已知 vulnerabilities 做测试, 验证
* 已知 vulnerabilities 的覆盖率
* false-positive 多不多
* 易用度 (标注 + 分析理解错误信息)
* typing 分析耗时



# Questions
* 他做的是否在某种意义上可以看成 isolation / access control?
正常地, 不能. 它是用 qualifier 做 taint analysis.

* type qualifier 的 checking rule 是什么?
传统 rule. context insensitive. 同 Foster99.
rule 要求所有 fmtstr 的参数都是 untainted 的.

* 如何确定 Term 的 qualifier?
同 taint analysis, 根据数据是否可信.
另外允许强制转换如 `int a = (untainted int) b`. 作者认为这种 easy to audit.

* 他的 type checking 是否是 context sensitive 的?
不是.

* 它提到了 soundness, 那它的 type system 是 sound 的吗?
只是说用 typing 做静态分析容易 sound.
但是后文没有论证自己的方法是 sound 的,
当然也可能它们沿用了 Foster99 的 soundness 证明.
