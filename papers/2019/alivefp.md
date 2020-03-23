# Alive-FP: Automated Verification of Floating Point Based Peephole Optimizations in LLVM

# Problem
## Research Question
Alive 不支持浮点代码, 这篇工作基于 Alive 自动验证浮点 peephole 优化的正确性.

## Existing Solutions, Their Troubles, c.f. Proposed Solution

## My Proposed Answer



# Proposed Solution
## Background

## General Idea
不支持浮点因为 Alive 的时候, S(z)MT(3) 还没太好的浮点支持?
Alive-FP 还是使用 SMT 的浮点算术, 即 SMT-LIB 中的 FPA 理论.

浮点优化的困难之处就是精度, 甚至 `x+1-x == 1` 都不一定成立.
论文里面只处理比特级别 (无取整误差) 精确的优化 (但可以不考虑特殊值 inf / nan / +-0 ...).

要处理的有: 浮点类型, 二元算符, 比较算符, 和 int 的转换, 以及 fastmath.

浮点操作如何编码:
* 一部分, 如 double 类型 / fadd 操作 / fpToUBV 转换, 在 SMT-LIB:FPA 中直接有

重写 Alive.

## Main Obstacles and Resolution



# Evaluation
## Metric
Effectiveness: 把 LLVM 中 104 个浮点优化翻译到 Alive-FP, 然后其中 7 个是错的.
发现 LLVM 中的错优化: 7 个.

验证耗时:
* 3 个是 SMT solver 挂了无法验证
* 基本上 50% 可以在 1s 内验证, 100% 可以在 100 s 内验证
* 越小的浮点类型验证越快


## c.f. Existing solutions



# Retrospect
## Remaining confusions

## Future Work and Weakness of Proposed Solution
e.g. Performance? Labor? Economicalness? Elegance? Compatibility? Usefulness?

## My main take-away
浮点好复杂, 精度啊, 特殊值啊都要特别处理.

一旦 SMT 有了好的理论, 上层会相当容易, 甚至沦为调包侠.



# Others
LifeJacket 和 Alive-FP 做一样的事情, 不过似乎被抢发了 orz.
