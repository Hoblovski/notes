# -OVERIFY : Optimizing Programs for Fast Verification

# Problem
## Research Question
就像优化可以优化时间 (-O2 -O3) 空间 (-Os -Oz),
通过调整代码的优化方法 (-Overify) 使得符号执行能更快执行.

## Existing Solutions, Their Troubles, c.f. Proposed Solution

## My Proposed Answer



# Proposed Solution
## Background

## General Idea
比如用 `sp += (a==1)` 代替 `if (a==1) sp+=1`, 避免分支.
因为分支需要让 SEE 复制状态, 这比给 SMT 公式加子句麻烦多了.

实现其实没有那么复杂
* 对 pass 做选择, 只用对验证友好的 pass
  - jump threading, loop unswitching
  - 传统的 copy propagation 等
  - 去掉那些 padding 啊 cache 友好优化等等
* 重新设置 cost parameter
  - 设置 branching 代价更高
* 改变程序结构
  - graph 分裂称 tree: 内联, 循环展开
  - 减少内存访问
* 编译后程序中保存更多元数据
* 自己写 C 库

有些工具用 CIL 预处理 C 代码来对验证友好.
作者号称编译到 Boogie (但是不可执行的) 比 -Overify 对验证更友好.


## Main Obstacles and Resolution



# Evaluation
## Metric
基于 LLVM 实现了工具
* 激进的避免 branch, 展开循环, 函数内联.

实现完编译 Coreutils 然后用 KLEE 跑, -Overify 和 -O3 比, 验证时间:
* 1/3 没太大区别
* 1/3 -O3 稍微好一点 (不到 50%)
* 1/3 -Overify 好, 有些好非常多
  - 没有 breakdown


## c.f. Existing solutions



# Retrospect
## Remaining confusions
另外也没有执行时间对比, 是不是优化弱了很多? 以至于 executability 失去意义.

## Future Work and Weakness of Proposed Solution
e.g. Performance? Labor? Economicalness? Elegance? Compatibility? Usefulness?

## My main take-away
处处都是可以做的, 优化符号执行不是搞 SEE 而是弄源代码.

# Others
想法真的很有意思, 但是不知道为什么似乎没有人接着做.

