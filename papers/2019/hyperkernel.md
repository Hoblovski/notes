# Hyperkernel: Push-Button Verification of an OS Kernel

# Problem
## Research Question
验证 OS kernel 的 functional correctness.

* 希望 low proof burden

## Existing Solutions and Their Troubles
* seL4: 构建 machine checkable proof, 用 coq / isabelle.
  - 太费人力 (22 my)

* Verve: deductive verification
  - TODO

## My Proposed Answer
* 使用安全语言: 防止底层 bug 如 null deref
* deductive verification: 防止逻辑 bug
* 符号执行: 也可以用来替代 deductive verification


# Proposed Solution
## General Idea
使用类似模型检查的方法, 用符号执行 (不严格是, 因为它是无循环的 SMT encoding).

### Specification
基础 functional correctness 写出两种内核的 spec.
不是严格分层的, 和 seL4 不同, 但都是两种 spec.
* 状态机
* 声明式性质检查

状态机
* 状态是内核状态 (包含所有全局变量的 store)
* 变迁是中断/系统调用
* 例如 `newstate = state { proc_nr_fds[state.currentpid] + 1, ... }`

不是所有状态都有意义, 故还包含 representation invariant:
* 状态的 invariant, 静态检查 e.g. `0 < S.currentpid < NR_PROCS`
* 要求变迁不损害 representation invariant

声明式
* 高层性质, 只需验证和状态机的匹配. 易写易推理.
* 例如 `forall f, pid, fd . file_nrfds[f] = 0 ==> proc_fd_file[pid][fd] != f`

### Proof
* 证明内核实现 LLVM IR 和状态机间的 **refinement** 关系.
  - $S$ 是 spec 抽象状态. $C$ 是实现的具体状态, $valid$ 是 representation invariant
$$
\forall S, C, \sigma\, .\;
  S \sim C \land valid(C) \Rightarrow S' \sim C' \land valid(C'),
  \qquad
  S \hookrightarrow^{\sigma} S', C \hookrightarrow^{\sigma} C'
$$

* 证明状态机 spec 和声明式 spec 的 **crosscutting** 性质
  - $P$ 是声明式 spec
$$
  \forall S, \sigma \, .\; P(S) \Rightarrow P(S'),
  \qquad
  S \hookrightarrow^{\sigma} S'
$$

## Other takebacks
* 验证一定要说, 那些东西是信任的
  - 基础定理
  - LLVM 后端, Z3, Python, 硬件 (尤其是虚拟化硬件)

* 验证还需特别要考虑验证
  - 内核初始化: seperate checker
  - stack overflow: 静态 trace 分析

* 部分 SMT 理论的尤其好处是能给出 **反例**

## Main Obstacles and Resolution
* 模型检测效率:
  - 有限化内核接口: syscall 有确定的 trace 上界. 能被 SMT 编码.
    尤其避免任何循环和递归: **执行上界不和系统参数 e.g. `NR_PROCS` 相关**.
    例如避免 `dup(old)` 而只支持 `dup(old, new)`.
  - 只使用能够高效判定的 fragment 对 SMT 友好
  - 用奇怪但等价的编码方法来对 SMT 友好:
    e.g. injective 用存在逆表示

* C 语言: 分析 LLVM IR

* 内存映射: 非单射 i.e. 不同虚拟地址是同一物理地址
  - 类似 dune 用 VT-x 使得内核完全有自己的地址空间.

* 我: 内核太大. 类似微内核 / exokernel 把一堆东西扔到用户态
  - lwip loading etc. Dune 类似的设计亦帮助了这个设计.

* 类似符号执行的不好处理 interleaved execution 等副作用
  - 内核不可抢占, trap handler 关中断执行
  - 限制异步操作 DMA 的副作用范围


# Evaluation
## Metric
* 能够有效防止 bug:
  - 把 xv6 的 bug 拉过来分析

* 需要的人力:
  - 代码量: 一万行, 比 seL4 等少多了

* verification 效率:
  - 验证一次 15 分钟

* 运行效率:
  - 把 dune 的能用的 benchmark 拉过来

## c.f. Existing solutions
* 效率: c.f. Linux Linux-Hyp


# Retrospect
## Remaining confusions
* 他在 encoding 中是说的 resource 和 object 不是很能理解为什么取这两个名字

* 为什么他那个奇怪的 refcnt 编码能对 SMT friendly?

* 为什么 performance 不和 xv6 比

## Future Work and Weakness of Proposed Solution
* 多核处理: interleaved execution

* 不支持整数到指针的转换

* 不支持进程间的共享内存

* 没有考虑复杂的 memory model

## My main take-away
如果说 (半) 手动验证的主要问题是难, 那么自动验证的主要问题就是弱.
Hyperkenel 为了让自动验证在有限时间内可行, 采取一系列手段来不考虑一系列东西
* 利用硬件虚拟化 VT-x
* 把一堆东西 lwIP / loader 丢到用户态
* 单处理器而且还关中断
* 用奇奇怪怪的内核接口
* 用奇奇怪怪的定理编码方法

