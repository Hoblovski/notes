# RedLeaf: Towards An Operating System for Safe and Verified Firmware

# Problem
## Research Question
为了达成 provably secure firmware，方法是实现验证了的 OS（RefLeaf）。

## Existing Solutions, Their Troubles, c.f. Proposed Solution
* 手动：CertiKOS / Certiuos / seL4
* 半自动：hyperkernel
* 全栈：IronClad，但是开销太大

## My Proposed Answer
不知道


# Proposed Solution
## Background
现代 firmware 通常需要 OS 的一些特性 e.g. 并发线程 / 进程隔离。
例如 trusty TEE (Android) 和 little kernel (Fuschia)。

标准的内存建模称一个大/无限数组 `nat -> byte` 不 scalable。
把内存组织成不会 alias 的对象看起来更 scalable。
并且有数组和量词也让 SMT 不开心，例如 list reachabity 谓词。

> To summarize, it is well-accepted that pointer separation and non-aliasing is
> really hard to both specify and reason about, especially once one starts
> introducing linked data structures such as lists and trees.

我日，传统的 pre/post-condition 还能花哨地叫做 assume/guarantee。

## General Idea
依赖 Rust 的 linear types 和 SMT 做验证。
验证使用 pre/post-condition 和 loop invariant 来“modular verification”。

也类似 Prusti 搞了个验证。从 MIR 翻译到 Boogie。

还拓展了 RustBelt 来稿 DMA 等硬件也要碰的指针。

内存分配和进程分配都用验证过的 memb 分配器。

OS 是不可抢占的。OS 执行过程中关中断。



# Evaluation
HotOS 没有 evaluation。


# Retrospect
## Remaining confusions
怎么把已有验证系统 memb 集成进来？

## Future Work and Weakness of Proposed Solution
支持多核？

## My main take-away
就提了个 idea。
不过思想还是，通过语言保证的 immutability 和 ownership 来简化验证。

