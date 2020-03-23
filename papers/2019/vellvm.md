# Formalizing the LLVM Intermediate Representation for Verified Program Transformations

# Problem
## Research Question
对 LLVM IR 这个语言, 形式化它的语义和类型系统.

应用如, 验证过的解释器 `lli`, 可以形式化验证 IR 上的变换.

## Existing Solutions, Their Troubles, c.f. Proposed Solution

## My Proposed Answer
按照文档给就是了. 可以用 state monad.


# Proposed Solution
## General Idea
首先形式化地描述其语法:
* 命令式地定义 `module <- prod (globdef) <- basic block <- phi, command, terminator`

然后描述静态性质
* Well-typedness (没有细描述) + SSA def-use dominance

对于形式化语义, 内存相对较难处理: 尤其是处理 malloc 和 free
* 继承 CompCert 的内存模型
* 为什么不是 `addr p-> val`, 因为需要记录 alloc 和 free.
* cell 是一个 byte, 区分不同类型: `mc <- value | pointer | uninit`
* block 是 malloc / alloca / free 的单位, block 中包含一系列 cell
* memory 是一系列 block
* 处理 struct: 做 flattening

操作语义
* 不同操作语义用于不同用途: 不确定性 / interp / ...
* 状态是 `state ::= (memory, frame list)`
  - 语义是状态上的变迁系统
  - 其中每个 frame 都有 `func, pcbb, bbcont, bbtmn, localvar, localmemblk`
* 对错误建模: 偏函数
* 语义的正确性: preservation / progress

形式化用了 Coq, 共 32000 行.

## Main Obstacles and Resolution



# Evaluation
## Metric
评估实现了 interpreter, 验证了 softbound
(做 memory safety, 把指针变成胖指针并且访问插桩) 的正确性.

* 建模内存安全: 定义新的 IR 语义, 其中内嵌 SB 中的内存安全约束
  - 证明这个语义上不会出现 memory unsafety
* 建模 SB pass: 定义函数 `SBtrans: module -> module`
  - 证明 SBtrans 后跑在原语义上精化了原程序跑在 SBspec 语义上

的确发现了 SB 中的 bug.

# Retrospect
## Remaining confusions
LLVM 还包含很大的前端 / 后端, 这部分也被形式化了吗?
* 似乎只是 IR 作为语言

LLVM 语义是自然语言说明的, 如何确认形式化模型是对的?
* 对着测试集 validate.

用了什么逻辑吗, 如 CSL?
* 没有

处理并发吗?
* 不处理

如何处理环境交互, 例如库代码, 系统调用, 以及 varargs?
* 不处理: 这些不是语言的东西

这种 "给一个语义" 的, 总会加一条 Threats to Validity

## Future Work and Weakness of Proposed Solution
限制如单线程, 32 位等.

## My main take-away
如果要严格分析优化的正确性, 就必须形式定义语义.


# Others
作者喜欢用上横线表示序列.
