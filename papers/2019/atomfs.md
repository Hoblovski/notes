# Using Concurrent Relational Logic with Helpers for Verifying the AtomFS File System

# Problem
## Research Question
验证并发文件系统, 证明的性质是 linearization.

## Existing Solutions, Their Troubles, c.f. Proposed Solution
CSL

符号执行肯定很难
* 状态爆炸
* Lock set 只能处理 race

## My Proposed Answer




# Proposed Solution
## Background
### Linearizability
* [TOPLAS'90] Linearizability: A Correctness Condition for Concurrent Objects
* The Art of Multiprocessor Programming

> [The Art of Multiprocessor Programming]
>
> The basic idea behind linearizability is that every concurrent history is
> equivalent, in the following sense, to some sequential history. The basic rule
> is that if one method call precedes another,  then the earlier call must have
> taken effect before the later call. By contrast,  if two method calls overlap,
> then their order is ambiguous,  and we are free to order them in any convenient
> way.

> [论文中]
>
> It allows overlapping operations executed concurrently to take effect in any
> order but requires preserving the real-time order of non-overlapping
> operations.

### Rely-Guarantee

## General Idea
2. 证明的关键定理是什么?

或者把 reanme 换成 rm 也可以?? 不行! rm 不能删除非空目录

R G 还有共享状态满足的 I

I 共享状态 (cfs, afs, ghost) 满足的条件

R G 共享状态 (cfs, afs, ghost) transition 满足的状态
> (thread) t’s rely conditions are simply defined as the union of guarantee
> conditions of all other threads.


helperMetadata = helpList * threadPool
AopState = (aop, args) | (end, ret)

helpList : ThreadId -> AopState
threadPool : ThreadId -> Descriptor




state : afs * cfs * ghost
invariants : assertion, over state
specifications: relational,  only abstract specification. for cfs
    we verify that cfs refines afs and the specifications
rely/guarantee: relational,  over state

R, G, I |-
    {P * (aop, args)}
    C
    {Q * (end, ret)}

C ::= C statements          modify concrete state
    | auxiliary commands    modify abstract state

Proof
1. give afs, cfs, spec,   I, R, G
2. prove simulation
3. simulation -> contextual refinement -> linearizability

## Implementation

基于 Coq, 一年半.

* CRL-H: 十万行 coq, 一半从 certiuos 借的.

* AtomFS: in-memory, 不考虑 crash safety, 直接用 C 写, 基于 fuse.
  - 代码似乎只有 673 行...
  - spec 约 2k 行. 证明 6w 行.

## Main Obstacles and Resolution



# Evaluation
## Metric
setting: ramdisk

* 实际效率
* scalability.

## c.f. Existing solutions
* baseline: DFSCQ, ext4, tmpfs

* 效率: 接近 ext4, 好于 DFSCQ.
  - scalability 还考虑了 vfs, 所以有 biglock 的版本





# Retrospect
## Remaining confusions
* 3.2 Generality 里面, xv6 根本没有 rename.

* 好别扭啊... 本来 concrete impl 是对的, 只是 linearizability 描述不对...
  能不能改 linearizability 而不是用一堆 lin-before 之类的?

* rename 是万恶之源, 导致 external LP.

## Future Work and Weakness of Proposed Solution
e.g. Performance? Labor? Economicalness? Elegance? Compatibility? Usefulness?

既然它处理了并发的问题, 能不能用到自动验证的符号执行里面? 比如搞个 decision procedure 之类的.


## My main take-away


# Others


