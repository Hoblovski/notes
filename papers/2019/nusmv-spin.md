# From NuSMV to SPIN: Experiences with model checking flight guidance systems

# Problem
## Research Question
分析作者使用 SPIN 做飞控系统的体验, 以及对比使用 NuSMV 的体验.

## Existing Solutions, Their Troubles, c.f. Proposed Solution

## My Proposed Answer



# Proposed Solution
## Background
NuSMV 是一个 symbolic model checker, 而 SPIN 是 explicit model checker.

经验是, 符号 MC 对 (例如硬件的) 同步系统更好, 显式 MC 对 (例如软件的,
多进程交互的) 异步系统更好, 不过似乎也有人号称符号 MC 对异步系统也更好,
所以不能简单的用同异步来判断.

## General Idea
直接把模型翻译成 SPIN 会导致非常差的状态, 因为变量太多了.
尽量使用 process-local 变量, 然后通信用共享变量上的消息传递.

如何优化各种 MC:
* 符号 MC: 变量顺序
* 显示 MC: 归约顺序

选取符号 MC 还是显示 MC 其实也取决于状态
* synchronous dataflow specifications
* asynchronous control-flow based specifications

简单的把同步 spec 翻译到异步 SPIN 导致极差的时空效率.
优化如
* 减少全局变量, 局部化然后使用 消息传递 / 共享变量 通信.

最后优化之后, 还是没有选择 exhaustive 模式 (SPIN 还有 hash 模型, 牺牲 soundness
以获得 scalability). 这样才能优化.

不过似乎对于非常复杂的系统, SPIN 可以 manage, 但是也只能 unsound: 发现一部分的 state space.

## Main Obstacles and Resolution



# Evaluation
## Metric

## c.f. Existing solutions

# Retrospect
## Remaining confusions

## Future Work and Weakness of Proposed Solution
e.g. Performance? Labor? Economicalness? Elegance? Compatibility? Usefulness?

## My main take-away
所以我认为, 对于同步的 spec, 使用异步的 SPIN 就是浪费.

> Our investigation reveals that Spin performs poorer than the symbolic model
> checker NuSMV on the one-sided synchronous FGS model.

# Others
> In other words, Spin can be more flexible than NuSMV in handling large scale
> systems, but can also be more difficult to use by non-experts, as
> optimization can raise a couple of issues in terms of usability[.]
