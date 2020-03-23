# Instant Bug Testing Service for Linux Kernel

# Problem
## Research Question
为了达成对 kernel bug 的快速反馈, 提出即刻的内核测试.

## Existing Solutions, Their Troubles, c.f. Proposed Solution
* 用户反馈周期太长
  - 反过来更让老 bug 难修复: too much noise

* 静态分析等找 bug 工具
  - 分析每个 commit?

* 已有的 CI 和测试框架:
  - 不够快, 不能每个 commit 都测试

## My Proposed Answer
* ktest 不够吗?



# Proposed Solution
## General Idea
首先分析 bug 证明 kernel 变得不稳定是的确存在的问题, 然后提出 kis.

* 图 1: bug 的数目和 lifetime 都在增加.
  - 原因: 快速发布的压力.
  - 是个恶性循环: 不尽早修好导致以后更难修

设计 KIS, 主要目的
* 快速反馈 (小时级别)
* 无需开发者学习
* 能自动找出 root cause commit 给作者反馈.


## Main Obstacles and Resolution
* 测试每个 commit 需要大量计算力
  - 尤其是编译内核, 静态分析和动态测试
  - obj 缓存到内存; 奇怪的短期优化; {并行化三个操作如 lock-free; delta-executing.}

* 怎么找 root cause, bisect 不好
  - 每个 bug 只报告一次
  - kernel developer 加上 regex 筛选等等减少 false positive.

* config 多, 怎么改进 coverage
  - 仔细选择 config: 一半手选, 一半随机
  - 其他 arch: qemu



# Evaluation
软工一样的 evaluation.


# Retrospect
## Remaining confusions

## Future Work and Weakness of Proposed Solution
e.g. Performance? Labor? Economicalness? Elegance? Compatibility? Usefulness?

## My main take-away


# Others

