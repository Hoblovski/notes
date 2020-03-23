# seL4: Formal Verification of an OS Kernel

# Problem
## Research Question
首次对一个 general purpose OS 做 functional correctness 证明.

## Existing Solutions, Their Troubles, c.f. Proposed Solution
没有已有的完整的对 general purpose OS 做 functional correctness.

保证安全
* 减少代码大小:
  - security / seperation / micro- / isolation kernel
  - small hypervisor

* type system:
  - singularity, SPIN

seL4 也很小.

## My Proposed Answer
* deductive verification:
  - 函数 -- spec

* symbolic execution:
  - 手写 assert 之类的作为 spec?



# Proposed Solution
## Background
function correctness 比 no-crash, no-unsafe 更强 -- 表示 impl 完全遵照 spec.
* 形式化的说是 refinement: 建立高层对象和底层对象的 correspondence
* 保证高层对象满足的 hoare property 也被底层对象满足
* access-control 的 security 似乎是可以从 functional correctness 分开的

## General Idea
形式化和 OS 的 fusing: 'rapid kernel design and implementation'
* 核心是一个 Haskell prototype, 作为 OS 和 formal 的中点.
* Haskell prototype 作为 executable spec 比直接从 theorem prover 中抽取要简单多了.
* 手动实现对应 haskell 的 C 以保证效率

形式化验证分为三层:
* 最高层的 spec
* <从 haskell prototype 生成的> 可执行 spec
* C 实现

### OS Design
使用 hoare triple, 包括 pre/post-conditions.

* 如果 invariant 涉及到全局状态, 证明会很麻烦
  - 限制 preemption
  - 通过 Haskell 使得副作用变得显然

* 内存分配由用户进程完成
  - 证明 use count 一致

* 并行证明很难 -- future work. 单核只考虑并发.
  - 避免 yield / preemption: 使用 event-based kernel execution model
  - 只在特别的地方开中断: $\{\text{pre(intrhand)}\}\; eni\; \{\text{pre}(B)\} B$
  - 中断 $eni$ 使用 polling 而非 $sti$
  - 完全避免 exception

* I/O 给用户程序处理

### Verification
abstract spec:
* 包含 what 但是不包含 how

executable spec:
* 定义内核 how, 但是不需要考虑 C 的复杂
* 需要证明和 abstract spec 符合

C impl:
* 需要保证 C 到 isabelle 的转换是准确的
  - C 标准语义. 从底层开始 (low level abstract machine)
* 使用 C subset
  - 不允许对栈数据取地址
  - 不使用函数指针

machine model
* 只有 C 不够 -- memory model etc
  - 包含 cache / TLB flushing

proof
* 关键定理 $\mathcal{M}_E$ refines $\mathcal{M}_A$ and $\mathcal{M}_C$ refines $\mathcal{M}_E$

> ... [B]ehaviour of the C implementation is fully captured by the
> abstract specification.

* 当然可能 $\mathcal{M}_A$ 有 bug, 但是
  - 已经消灭了一大类 bug
  - $\mathcal{M}_A$ 更容易分析论证

* proof 很大一部分是 invariant 的证明
  - memory / typing: -> type safety
  - data structure / algorithmic




# Evaluation & Experiences
## Metric
效率:
* IPC 效率. 不是重点. comparable.

验证工力:
* 设计构建内核:
  - 4 person year
* 证明:
  - 9 py 学习证明背景
  - 11 py 实际证明

重验证工力:
* 取决于修改和已有实现有多 tangled
  - 非常 tangled 的改动还是导致极大的 recertification 代价

## c.f. Existing solutions
* 效率和其他 L4 比较

* 验证工力中的实现和 pistachio / hazelnut 对比

* 不知道为什么没有和前人 OS 验证的工力做对比
  - 放到 related work 了


# Retrospect
## Remaining confusions
* 什么是 refinement, 是某个 model 的吗? 形式化定义是什么?

* 很多对 seL4 的设计都是为了减少不确定性 e.g. 中断处理
  - 能否在 retaining undeterminedness 同时做 verification?

* 能不能用 type system / effect system 减少验证工力?

* 能不能搞一个 deductive semi-manual verification 和 full automatic (e.g. SE, MC) 的 fusing?

## Future Work and Weakness of Proposed Solution
e.g. Performance? Labor? Economicalness? Elegance? Compatibility? Usefulness?
* 把其他没有验证的部分 e.g. compiler, hardware 验证一次

* 加强 OS 然后验证加强的 OS e.g. multicore

* 我: 如何减少验证代价?

## My main take-away
直接验证 abstract spec 对照 C impl 对于 OS 开发者太困难,
因此增加了一层 indirection layer (Haskell / executable spec),
却意外成了验证的中心.



# Others
* 讨论正确性的时候要把系统中那些部分可信任说明白
  - hardware? compiler? ...

* 自称 functional correctness 比自动的 model checking 更强
  - 自动证明 functional correctness 到后来 Hyperkernel 等才出现

> ... Haskell runtime relies on garbage collection which is unsuitable for
> real-time environments.

> Simple typos also made up a surprisingly large fraction of discovered
> bugs in the relatively well tested executable specification in the first
> refinement proof,  which suggests that normal testing may not only miss
> hard and subtle bugs,  but also a larger number of simple,  obvious
> faults than one may expect.

## OS 验证的相关工作 (Related Work)
* 最初工作: secure unix (UCLA), provably secure OS
  - secure unix 提出了 functional correctness 和 refinement
  - secure unix 表示为了让 OS 可被 TP 证明付出大量效率代价

* 不少人都只提出了 formal model / design 但是没有对应 impl
  - PSOS, Mach, EROS, SELinux, MASK
  - 类似的有其他工作只证明了 kernel 的一小部分

* 有一部分工作是做在 information flow 上的
  - MASK, hardin06,

* verisoft 尝试验证整个系统 stack -- 从 hw 到 app sw 到 compiler
  - 验证很好, 但是系统效率太差

* 其他方法有 static analysis 和 model checking
  - 以及 type checking: 他们称 type safety 不强
