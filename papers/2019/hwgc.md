# A Hardware Accelerator for Tracing Garbage Collection
MICRO best pick 版本

# Problem
## Research Question
加速 GC.
* 针对 tracing GC i.e. mark and sweep

## Existing Solutions, Their Troubles, c.f. Proposed Solution
* java machine / lisp machine:
  - 被通用 CPU 打败了
  - [wiki](https://en.wikipedia.org/wiki/Java_processor)

* 例如 Azul’s Vega 和 IBM z14
  - 他们还是 CPU, 而 CPU 不是处理 GC 最好选择

* CPU 加速指令
  - Moore 定律使得当时通用 CPU 发展迅速
  - 他们对架构的修改太大

> past work typically focused on general CPU features that improve some aspect of GC (such as read barriers)

## My Proposed Answer
* marking phase 可以做 fetch-orv 类似 writev 加速 -- 还是 CPU 自身
* 在内存中区分指针和数字, 指令变成 `st.ref LOC LOC` 和 `st.val LOC VAL`
  - 然后硬件就可以自己 marking 了
* 至于 sweeping 由专门的 concurrent 的另外一个核完成



# Proposed Solution
## Background
GC 算法
* 假设 obj 的 `ptr` 满足 `[ meta ptr ]` 包含其元信息 (marked? refs...)
  - 一次 load 就能读取
* obj 的诸引用域存放在 `[ptr + X]` 中
* 他们的 GC 把 `visit[v]?` 放到 dequeue 而不是 enqueue 的时候判断
  - 一个 `ptr` 可能入队多次

```
simple GC
  let Q = {roots}
  while Q is not empty
    let ptr = dequeue Q
    let already_marked, nrefs = fetch-or [ meta marked+refs ptr ]
    if already_marked then continue
    for x in (refs ptr nrefs)
      enqueue Q [x]
```

## General Idea
不是修改 CPU 而是在 memory controller 靠近的位置加入一个 accelerator.
* 是一个 memory mapped 设备, 可以 DMA
* 和 CPU 共享 VAS (virtual address space) 故有自己的 PTBR 和 PTW, TLB
* 和 kernel driver 以及 JVM 通过共享内存访问 (kernel driver 负责发送 PTBR 等)

分为两个部分: traversal 和 reclamation.

traversal 包含一个指针队列, 使用类似上面的算法.
* traversal 是 memory-bound 希望尽量最大化利用 mem bw (若不考虑 interference)
* bidirectional layout 使得 `[ meta refs ptr ]` 更简单, 不用查 type descriptor
* 把算法分成两个流水部分 marker 和 tracer: 不因对方而 stall, 最大化利用 mem bw
* GC 的内存访问无需序

reclamation
* 不是 compacting, 而是把堆分成 block 然后一堆 linear sweeper 线性扫描 block

至于 root 就不 offload 了



# Evaluation
基于 RISC-V rocket 和 Linux. 没有上板子 -- 都是 (cycle accurate) 模拟实验.

* 用 Dacapo benchmarks
* 基准是 rocket

## Metric
图 4 的 (b) (c) 标反了

* mark / sweep phase 耗时
* 芯片的 area size
* mem bw 利用率
* power
  - DRAM 部分 power 一定上升. 总体 power 还是下降.

## c.f. Existing solutions
baseline 只有 Rocket. 没有和 Vega / IBM 做.


# Retrospect
## Remaining confusions
* 为什么要把 `visit[v]?` 放到 dequeue 而不是 enqueue 判断?

## Future Work and Weakness of Proposed Solution
作者写了个很有意思的未来方向: oblivious GC -- 就像现在的 oblivious PTW.

话说能不能有 hwrc (硬件引用计数).

## My main take-away
就像 Patterson 说的, 在性能要求很高的地方, 通用 CPU / 语言的时代基本上不够了.
DS accelerator 和 DSL 是这些方面的未来.



# Others
* wimpy CPU: 计算能力弱, 但是能耗好的 (还可能有 specialized) 的 CPU
  - IO efficient
  - Atom

* brawny CPU: 计算能力强, 能耗高的 CPU
  - Xeon

c.f. Brawny vs. Wimpy: Evaluation and Analysis of Modern Workloads on Heterogeneous Processors

> The CPU’s sequential control flow and its load-store queue therefore limit
> performance, while both operations do not use most of the CPU’s functional
> units or its area-intensive caches, hence wasting chip area.
> Similar to the argument for page table walkers, we therefore believe GC
> should be entirely performed in hardware to both increase the throughput of
> the operation and reduce the required area.
