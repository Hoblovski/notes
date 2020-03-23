# Formal Verification of SSA-Based Optimizations for LLVM

# Problem
## Research Question
形式化地验证 SSA (LLVM IR) 上程序变换的正确性.

## Existing Solutions, Their Troubles, c.f. Proposed Solution

## My Proposed Answer



# Proposed Solution
## Background
SSA 优化证明的困难在于 "non-local reasoning about the program".
> Second, the SSA property is global to the code of a whole function and not
> straightforward to exploit locally within proofs.

程序变换的正确性: 用精化说明.
* 基于 trace equivalence, 使用 LTS 表示程序变迁
* trace 包括正常终止和中途出错 stuck
* 使用 simulation 图证明 refinement: 找一个程序状态上的 ~ 关系

## General Idea
基本思想就是:
* SSA 变换基础性质是 SSA preservation (scoping / dominator property),
  可以把其他我们想要的性质编码到其中, 然后证明 preservation.

SSA Scoping
> [A] program satisfy certain invariants to be considered well formed: every
> variable in the top-level function must dominate all its uses, and be
> assigned exactly, once statically

使用一系列的 type rule 定义 SSA scoping.
然后发现证明 SSA scoping 中, 使用的 invariant WF_FR 可以被任意的谓词 P 参数化.

基于 Vellvm 验证了 mem2reg. mem2reg 把本来在内存中的变量提升到 SSA 里面.
为了形式化, 重新设计一套 vmem2reg.

方法: 由一系列微变换组成, 但是每个都满足 SSA preservation 和 refinement
1. 找出 promotable alloca (函数内 alloca 且不逃出函数), 对每个 promotable alloca:
2. 插入 phi 结点及其前后的 load / store.
3. 消除 trivially 额外的 load 和 store (LAS SAS)
4. Dead Store/Alloca Elimination
5. phi 结点消除: `phi [v1, l2] -> v1`

正确性: 每个微变换需要
* 保持 SSA 性质
* 证明 refinement
  - 用 simulation. Contextual refinement.
* 保持 promotable 性质: 额外性质, 类似归纳假设

最终定理:
* refinement: `|- prog => |- prog[f!->f'] /\ prog >= prog[f!->f']`

50000 行 coq, 800 行 vmem2reg.

## Main Obstacles and Resolution



# Evaluation
## Metric
* 比较 mem2reg 和 vmem2reg 的效率: 编译时间, 运行时间


## c.f. Existing solutions



# Retrospect
## Remaining confusions

## Future Work and Weakness of Proposed Solution
e.g. Performance? Labor? Economicalness? Elegance? Compatibility? Usefulness?

## My main take-away
SSA 上优化很难: 因为 non-local, 局部修改可能直接导致优化后根本不是 SSA 了.

学习证明也很有用: 作者直接把自己的东西嵌入了 SSA preservation 的证明过程中的引理.


# Others

