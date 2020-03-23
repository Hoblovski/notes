# Provably Correct Peephole Optimizations with Alive

# Problem
## Research Question
通过提出一个 DSL 能自动生成 C++ 代码, 解决/减少 LLVM 中 peephole 优化的 bug.

## Existing Solutions, Their Troubles, c.f. Proposed Solution

## My Proposed Answer



# Proposed Solution
## Background
Peephole 优化: 程序局部的简单 "algrbraic" 重写.
* 主要关心其实就是 InstCombine 这个包含一系列 (1k+) peephole 优化的大 pass.

## General Idea
一个优化写作一个可选的前置条件, 以及源/目标模式, 用同根 DAG 描述.
源目标模式中类型是多态的 e.g. i32, i64.
利用类型规则保证优化是 well-formed 的.
处理内存使用 gep 和 inttoptr.
poison value 用来表示某条指令可能导致 UB, 不能被用到有副作用的语句中

正确性的描述: 用 SMT 检查证明性.
* 最简单的就是等价性: 同样输入同样输出, 但只有在 poison-free 的情况下
  - 直接用 SMT 的 bit-vector 求解, 包括前后置条件和 IR 语句
  - 似乎 defineness 是指, SMT 和 C 都定义了; 而 poison 是指, C 未定义但是 SMT 定义了
* 优化还需要保持 poison-free 性质; 对原模板中 undef 可以任意选取但是目标 undef 不能假设任何值...
  - 论文中 $P, p$ 只是用来处理 may alias 的, 取个 $\lnot$ 似乎就好了
* 内存用 byte array 建模
  * 数组理论使用 Ackermannization 做, i.e. 一堆 if-then-else 来做.
  * 用新的自由变元表示 alloca, 这也需要约束
  * 内存 load 用 select 表示 (不固定下标?) `select(m, ptr) -> val`
  * 内存 store 用 array update 表示 `m' = store(m, ptr, val)`
* 因为用 SMT 所以可以有反例.

$$
\begin{align*}
    \forall \mathcal{I}\, \mathcal{\bar{U}}.\;\exists \mathcal{U}.\;& \phi \land \delta \land \rho \land \alpha \land \bar{\alpha} \to \bar{\delta}\\
    \forall \mathcal{I}\, \mathcal{\bar{U}}.\;\exists \mathcal{U}.\;& \phi \land \delta \land \rho \land \alpha \land \bar{\alpha} \to \bar{\rho}\\
    \forall \mathcal{I}\, \mathcal{\bar{U}}.\;\exists \mathcal{U}.\;& \phi \land \delta \land \rho \land \alpha \land \bar{\alpha} \to \iota = \bar{\iota}\\
    \forall i.\;\forall \mathcal{I}\, \mathcal{\bar{U}}.\;\exists \mathcal{U}.\;& \phi \land \alpha \land \bar{\alpha} \to \text{select}(m, i) = \text{select}(\bar{m}, i)
\end{align*}
$$

翻译到 C++ 就是直接的一对一翻译.


## Main Obstacles and Resolution



# Evaluation
## Metric
实现 5200 loc python, 使用 z3 和 QBV.

手动翻译 InstCombine 中 334 / 1028 个优化 (不是所有优化 Alive 都能做 e.g. FP).
* 发现 8 个 bug, 乘除模最多
* Alive 可以翻译这些所有优化到 C++, 替代 InstCombine 编译快 7%, 运行慢 3%.


## c.f. Existing solutions



# Retrospect
## Remaining confusions

## Future Work and Weakness of Proposed Solution
e.g. Performance? Labor? Economicalness? Elegance? Compatibility? Usefulness?

## My main take-away
z3 是人类的好朋友.

# Others
论文中很大内容处理的是未定义行为.

