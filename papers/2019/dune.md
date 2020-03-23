# Dune: Safe User-level Access to Privileged CPU Features

# Problem
## Research Question
应用访问特权硬件资源 e.g. 页表中断 TLB:
* 直接
* 安全

## Existing Solutions, Their Troubles, c.f. Proposed Solution
改 kernel
* 麻烦
* 不同应用不同修改, 不一定 compose

libOS + 虚拟化
* 难以集成 host OS
  - lib OS 不能完全模拟 host OS 的环境

Dune
* 不改 kernel 用一个 kmod
* 客户就是普通进程不是 VM

## My Proposed Answer
目标有两个
* 高效访问特权硬件资源
* 能利用 host OS 的 syscall 等

我的想法: 用一个 type 2 VMM.
* 那么进程跑在 ring [123] 吗? 这样访问特权资源不高效.
  应用进程必须跑在 ring 0, 还要和 kernel 隔离,
  我们还不信任应用不会搞坏 kernel.

* 结论: 必须有两个有硬件隔离的 ring 0 -- VT-x.



# Proposed Solution
## Background
* VT-x: *Shadow copy of privileged state*
  - VMX root / nonroot 模式: 前者是 VMM. 后者可以访问 IDT 等而不用 trap.
  - 内存用 EPT.

## General Idea
> By hardware virtualization, provide *process* abstraction rather than *machine* abstraction*

允许用户访问
* 中断/异常: e.g. debugger, GC 等.
  - 传统 c.f. signal 的方法

* 页表和 TLB: e.g. GC
  - 传统 c.f. syscall 的方法

* 特权级: 特权隔离 / 不信任代码执行
  - syscall 被应用自己捕获. 用 VMCALL 来真正的 syscall host.

组成部分
* 一个 kmod
* 一个 ulib

### kmod
* kernel 在 VMX root -- 见 figure 1
* dune 进程 (运行在 dune mode i.e. VMX nonroot ring 0)

* dune 类似 VMM, 实质上也是一个 type 2 hypervisor. 但不同
  - 其上运行进程而非 guest OS. 提供 process 抽象而非 machine 抽象
  - lightweight: syscall vs hypercall, 对设备的处理 ...

### ulib
* 用户程序跑在 ring 0 -- 据称没有太多不兼容
  - libdune 检测应该是 hypercall 的 syscall. 也可以用修改过的 libc.

* `dune_signal` 还没集成到 `signal/sigaction` 中 -- 只是实现限制
* pthread 可用但是 libdune 部分还不 thread-safe -- 只是实现限制

## Main Obstacles and Resolution
* 避免 dune 进程乱改页表
  - EPT: 进程只能改上层页表 GVA -> GPA

* EPT 和 kernel PT (i.e. HPT) 并不兼容有不同的 binary format
  - dune 从 HPT 构建 EPT 懒惰地

* syscall 需要 HVA = GPA 但是 dune 进程中都是 GVA
  - 类似 "DMA 需要 VA 但是内核中都是 VA"
  - 对等映射 / 手动 walk page table



# Evaluation
## Metric
* Dune 对效率影响 -- 具体数字, 以及来源分析 (和基此的 microbenchmark)

* 给出了 positive case 以及 negative case.
* 给出了 microbenchmark 和 application benchmark.

主要 overhead 来源
* VT-x (syscall 用 VMCALL 实现, page fault 要做 VMENTRY)
* EPT (TLB double walk).

主要优化在
* 中断/异常:
  - ptrace: 没有切换特权级以及调度的开销
    (反正无论是否 ptrace 都要走 dune 过一遭)

* 页表和 TLB:
  - 应用程序接收到 pagefault 通知
    (SIGSEGV vs 直接硬件给 VMX nonroot ring 0 的 dune 进程)

* 特权级:
  - 似乎这个没有效率相关的. 只是让 sandbox 更加安全

## c.f. Existing solutions
* sandbox: 在 linux 原生跑和用 dune (VMX nonroot ring [03])
* wedge: linux 原生和 dune 中. 主要是受益于 "页表与 TLB"
* GC: gc bench. Boehm GC 以及其放到 Dune 上, 然后加上不同 tune



# Retrospect
## Remaining confusions
* EPT 的开销多大? c.f. 无虚拟化 c.f. shadow PT
  - ISCA'12: Revisiting Hardware-Assisted Page Walks for Virtualized Systems.
  - ASPLOS'08: Accelerating Two-Dimensional Page Walks for Virtualized Systems.

## Future Work and Weakness of Proposed Solution
* 标题有点奇怪 -- dune 进程明明运行在 ring 0 却叫 "User-level Access"
  - 改成 Application-level Access

* 工程努力: 更多 e.g. 探索 cache 等也放到 dune 里面

* 是否能用 SFI? 似乎不行
  - SFI 主要还是对内存的限制
  - 并不是不能用硬件. 而且 SFI 也有自己的开销.

* 没有想到特别的 weakness

## My main take-away
大家都觉得内核成为了效率瓶颈, 有的是网络 (DPDK/IX/mTCP) 有的是 I/O (DB),
然后各种搞 kernel bypassing. 而 Belay et al 甚至觉得传统认为必须是 OS
管理的特权资源也要做 kernel bypassing, 但是当然乱搞 bypassing 是不安全的,
所以才有 "safe user-level access to privileged CPU features".
