# Termination-Checking for LLVM Peephole Optimizations

# Problem
## Research Question
基于 Alive (写 peephole 优化的 DSL), 检查一类导致 peephole 优化不终止的情况.
* A 优化 undo B 优化, 导致不动点算法不停迭代.

(当然, 如果你能求解普遍的终止性检查图灵就跳起来了)

## Existing Solutions, Their Troubles, c.f. Proposed Solution

## My Proposed Answer



# Proposed Solution
## Background
Alive: 写 peephole 优化的 DSL.
* peephole 优化统一表示为 `optimization ::= condition? srcPattern '->' dstPattern`
* 使用 SMT 检查正确性
* 其实是 DAG 优化, 把一系列的指令变成 DAG. 不受 physical layout 的影响.
* 可以自动生成 C++ 的优化实现, 插入 validity check
* 当时不支持内存相关的优化 (?)

## General Idea
大意是, 利用优化组合的概念 `opt1(opt2(f)) -> optcompose(f)`,
多个 peephole 优化可以组合成一个.
* 优化组合是一个偏函数: 不是任何两个优化都可以组合
* 枚举固定长度的优化序列 $\langle o_1 o_2 \ldots o_n\rangle$
  - 就是全排列枚举, 非常暴力
* 把它们组合得到一个 "大" 优化 $O=o_1 \cdot o_2\cdot \ldots \cdot o_n$
* 检查是否 $O$ 可能 (precondition 可满足), 且 $O\cdot O$ 可能导致一个 "更大" 的 pattern

> When the optimization composes with itself with a satisfiable precondition,
> the optimization can be applied infinitely many times when the
> self-composition of the optimiza- tion consumes a source program no larger
> than original optimization (see Figure 4 for an illustrative example).

技术难点主要是
* 优化组合: DAG 对其等.


## Main Obstacles and Resolution



# Evaluation
## Metric
2k 行 python.

主要讲了 effectiveness:
* 发现 184 个循环圈, 主要的原因是优化的前置条件太弱
  - "猜想" 真正的 cycle 其实很小, 不用枚举太长的组合
* 手写实际 IR, 真正死循环的有 179 / 184 个.

例子 `y := C2 & (x | C1) ==> y := C2 & (x | (C1 & C2))`, 如果 `C1 & C2 = C1`
就会卡死.


## c.f. Existing solutions



# Retrospect
## Remaining confusions
没说跑一次需要多少时间啊, 也没说 SMT 会不会挂.

## Future Work and Weakness of Proposed Solution
e.g. Performance? Labor? Economicalness? Elegance? Compatibility? Usefulness?

## My main take-away
内存是优化和验证的敌人. 不用内存也能写图灵完全的程序吧!

把优化看成函数, 然后利用 "函数复合" 这个概念做终止性检查, 很有意思.

作者小心的只解决了一类 bug, 毕竟停机问题很难.

SMT 真的好用.



# Others

