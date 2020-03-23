# PtrSplit: Supporting General Pointers in Automatic Program Partitioning
和 type system 有关系
> We describe a type-based algorithm for performing deep copies of pointer data...



# 摘要
背景是: 基于 principle of least privilege 的 program partitioning

问题是: C/C++ 自动 program partitioning 中很大一个障碍是指针, 尤其是可能跨 partition 的指针.
* 指针不包含 bounds info, 很难做 marshalling, 可能要求程序员手动写 buffer marshalling.
* 如果有指针, 简单的全局 pointer analysis 并不 scalable.

的确是问题. privtrans 不考虑这个问题.

方法是:
* PDG, 加上 parameter tree 做依赖分析. 解决的问题中第二个挑战.
* 插桩, 使得跨 partition 的指针包含 bounds info.
* type-based 算法来完成 auto marshalling.

其实是在 LLVM IR 上面做.



# Step1 PDG -> raw partitions
PDG 定义
* 点是 instr.
* 数据依赖边: def-use; 以及对内存的写后读 `*p=1; y=*p`
* 控制依赖边: n2 出发, 有多条路径, 只有一部分到达 n1.
* 调用边: callsite 连接到所有可能的 callee. (考虑 indirect call).

Parameter-Tree: 为了过程间分析 scalable.
* 旧的是, 把 caller 中所有相关 instr 和 callee 中所有相关 instr 连起来.
  - 这样是乘法的复杂度.
* 用 parameter tree 之后, 把 caller 里面所有相关 instr 和 actual 连起来,
把 callee 中相关指令和 formal 连起来, 再把 actual 和 formal 连起来.
  - 注意, 如果 actual 和 formal 是指针, 还要把 \*actual 和 \*formal 连起来, 等等.
  - 这样是加法的复杂度.
  - 并且 PDG 构建可以模块化. (这个想法并不困难啊)

构建 Parameter Tree 的过程是 type-based, 不过也很 trivial.
基本上, 遇到指针就多一个儿子, 表示解引用又可能是什么数据.

partitioning 就很简单, 从被 **标注** 为 sensitive 的开始,
一口气把所有在 PDG 里面有数据/控制 (但不含调用 -- RPC) 相关的放到 monitor 里面.
但是还需要 downgrade (declassify) 部分数据, 否则显然整个程序都会变成 sensitive.



# Step2 raw partitions + IR -- selective pointer bounds tracking ->
略



# Step3 -- typed-based [un]marshalling -> final artifact
问题: 指针, 尤其是循环队列这种.

如果 RPC 有一个参数是指针, 那么发送的时候必须把下层的 buffer 一并发过去.
否则对面拿到指针, 指针指向的地址没有意义. 用到 bounds tracking 的结果.

归纳定义了类型 $t$, 值 $v$. 那么同样归纳定义 $t$ 类型的值 $v$ 的 marshalling encoding.
对于指针类型的值, 还要包含其指向区域的 encoding.
这样可能产生无限循环 (循环队列), 因此保证只有在指向区域没有 encode 过才包含.



# Questions
1. typesys 是怎么相关的?
首先, 标注使用的是 qualifier, (它们用的是 attribute).
其次, 构建 Parameter Tree 和 Marshalling 的时候是 type-based,
  在归纳定义的类型的基础上归纳定义这些 type-based 概念/计算.
虽然只是 naive type, 没有用 type theory 的研究结果.

2. 我希望更好理解 PDG 的概念.
Figure 3. 是很好的例子.
PDG 构建可以直接将库函数做成 model.

3. 是否考虑 unsound conversion. 如 `char* -> int`, 以及 `void*`.
我没看到. 原文如下, 不过真的都有 corresponding node 吗?
> With parameter trees, inter-procedural dependence representation becomes
> trivial. For each function call site, we just connect nodes in the actual
> parame ter trees to the corresponding formal parameter trees of the callee
> function, using bidirectional flow edges.

Discussion 提到了. 称, 实验中没遇到. 解决可以用 RTTI.
