# Modular fine-grained concurrency verification

------------------------------------------------------------------------------
# INTRODUCTION
* 解决的问题: modular reasoning about fine-grained concurrency

几种方法
* ownership, CSL: too simple to deal with fine-grained concurrency
* temporal logic, simulation: too general and complicated, and usually not modular
* Rely/guarantee: compositional but not modular

论文中的方法: RGSep
* a combination of rely-guarantee reasoning, separation logic, linearisability, and auxiliary variables.

## 概念
* 证明是 compositional 的:
> [W]e can compose proofs of a program’s components to get a proof for the entire program

* 证明是 modular 的:
> [G]ive a specification and a proof that is reusable in every valid usage context

------------------------------------------------------------------------------
# BACKGROUND
## Shared Memory Concurrency
synchronization 方法的性质
* nonblocking:  就算某些 threads 失败, 其已能 make progress
* wait-free:    无论哪些 threads 产生延时, 余下所有 threads 均 make progress
* lock-free:    只要存在一个 running thread 那么一定有 thread make progress
* obstruction-free:
  guarantees progress for any thread that eventually executes in isolation

假设: parallel composition of threads has an interleaving semantics (SC)
* aligned single word reads and writes are executed atomically by the hardware

辅助变量:
* An auxiliary variable [62] is a program variable that does not exist in the
  program itself, but is introduced in order to prove the program’s
  correctness.
  - 不会真正 '执行', 可以合并到前后的 atomic block 里
  - 还有 auxiliary {state, code}

## Terminology and notation
粗粒度并行就用和顺序执行一样的 invariant 就行了.
invariant 用一个程序状态上的一元谓词表示.
> A sequential program with modules is essentially a coarse-grained concurrent program with one lock per module.

细粒度并行的一种方法是 rely/guarantee.
其用状态上的二元关系 (状态上的二元谓词) 表示.

## Rely-Guarantee Reasoning
* 基于 Owicki-Gries: 对于 parallel composition, 要求 C1 的任何 atomic actions
  不能影响 C2 中 atomic action 间的 assertion, 反亦.
  - non-compositional


Rely-Guarantee: 是 compositional 的.
* The specifications consist of four components (P, R, G, Q).

* specification 描述线程.

* P: precondition.
  - 状态谓词.

* Q: postcondition.
  - 状态关系 (pre-state 和 post-state).

* R: rely.
  - 状态关系.
  - properties of the individual atomic actions invoked by the environment.
  - 描述环境的干扰 (atomic actions) 共同性质.
  - P 和 Q 需要对 R 保持 stable i.e. unsensitive to R

* G: guarantee.
  - 状态关系.
  - properties of the individual atomic actions invoked by the thread itself.
  - 此程序 atomic action 的性质.
  - 我的 G 是其他部分的 R.

Hoare 逻辑记号 $\{P\}\;c\;\{Q\}$, RG 是 $R, G \vdash \{P\}\;c\;\{Q\}$

## Separation Logic
TODO

## Others

## Comparison
* global 和 local reasoning: assertion 描述整个状态还是部分状态
  - Hoare Logic, rely/guarantee 是 global 的
  - Separation Logic 是 local 的: 独立的分析模块

* relation 和 invariant: 
  - relation 严格地比 invariant 强, 对并发系统更好

作者认为, relation 比 invariant 好, local 比 global 好.


# Linearity
## Definition
* TOPLAS'90 的是 invocation-response pair 的看法 (IO automata).

* Lamport'83 提出, linearizable 当且仅当存在一个 *linearization point*:
  - where the terms “externally visible” and “atomic” are defined
  - externally visible: global memory, 或者能影响 module public method 的返回值
  - atomic: with respect to all public methods on the module 以及 with any code outside the module

> A method call is linearisable if it is observationally equivalent to an atomic
> execution of the method at some point between its invocation and its return.

> In theory, a method is linearisable if and only if modulo termination it is
> equivalent to a single atomic block performing its abstract operation.

标准证明方法: transition system.

## Proving Linearity
TODO
