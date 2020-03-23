# KLEE: Unassisted and Automatic Generation of High-Coverage Tests for Complex Systems Programs


# INTRODUCTION
* 手工测试和随机测试效果都不太好
* 符号执行可以达到较好的 coverage, 但是主要的问题还是 path explosion 和 environment problem.

Contribution:
* 提出 KLEE, 目的是 high coverage.
* 在实际的应用 coreutils 上测试:
  - 良测试的
  - 复杂环境交互


# OVERVIEW
以 `tr` 为例, 说明了程序本身的复杂性和环境交互.
也是用了一个 motivating example.

KLEE 目标
* 尽量高的覆盖率
* 在潜在危险操作 (deref, assert) 处检查是否可能出错.

运行
* LLVM IR 上运行
* 符号化的命令行输入 `--symexe MAXLEN_A1 MAXLEN_A2 ...`


# DESIGN & IMPLEMENTATION
* 任何时刻可能有多个状态待探索.
  KLEE 每次选择一个, 执行一步 (一条指令), 得到新的 state.
  也就是说不是直接简单地系统 `fork()` 然后探索, 这太贵了.

* 遇到 branch, 用 solver 检查能达到哪个目标, 加入新的 state.
  可能加入两个新的目标.

* 危险操作也是分成两个 branch -- 安全和出错.
  检查出错是否可达, 但是之后沿安全探索.

## 处理内存访问
* 紧凑地表示内存.
  - 地址空间不是 flat continous,
    而是将每个有效的 object 映射成 STP array.
    i.e. segmented memory address.
  - state fork 的时候, object 做 COW.

* 指针: 对于每个可能指向的对象 fork state (新 state 有额外的 constraint).
  实际中, point-to 集合很小.

## 减少 solver 负担
SMT 求解是瓶颈, 所以专门将它的优化.
当然, NPco 问题中优化都是特例式的, 给出的也是特例式优化.

*学习 writing: 每个优化都给出了例子*

*学习 writing: 每个优化给出了只有它的时候 performance gain*

* expression simplification: 就和编译里面用到的一样

* constraint simplification: 如果新状态 refine 已有 constraint,
  那么采用重写已有的 constraint, 而非加入额外的 constraint.

* constraint independence: constraint set 可以分成引用符号不相交的子集.
  这样 solve 的时候, 只需要 solve 部分子集.

* counter example cache: 用 UBTree 做 cache 存储, 支持高效的子集查询.
  - 子集无解 $\to$ 约束无解
  - 超集有解 $\to$ 约束有解.
  - 子集的解可能是约束的解

## 探索策略 (scheduling)
用两个 heuristics, round-robin 交替使用.

* random path selection: 基于执行二叉树的
  - 喜好较浅的节点
  - 防止 fork bombing 造成 starvation

* coverage optimized search:
  选择看起来很可能快速发现覆盖新代码的状态
  - 计算状态的权重, 按照权重选择状态

防止昂贵的指令 (分叉/内存访问$\to$SMT) 独占执行时间.

## 环境交互
将对环境的访问发送到 [interface] model, model 能产生我们想要的 constraints.
* model 针对 syscall 构建而非 libc 构建.

例子: 文件系统.
* 如果操作的是具体的文件, 那么直接具体地读写;
* 否则 *模拟* 一个简单的符号化的文件系统.
* **隔离**: 不同 state 操作的是不同文件, 但它们在相同的进程中运行
* **符号化的文件系统**: $N$ 个符号文件, 每个内容也是符号. 不过简化地, 长度是确定的 (Fig. 3).
* **环境失效**: disk crash, disk full 等. 让 syscall 失败: 用 ptrace 来让 syscall 变成失败.


# Evaluation
metric 有
* coverage: 简单的 line coverage, 不过 path coverage 最好.
  使用 eloc 而非 loc (3x 差别).
* coreutils: workload 特点, failing syscall 的效果,
  和手写的比, 是否完成了发现 bug 的功能, 和 random testing 比.
  - 和手写的比: 低级/高层的错误
  - coreutils 测试完善, 比 random 好多了
  - 还特意说了一句 "对整个 coreutils 做 evaluation, 我们没有任何 cherry-pick 的意思"...
* busybox: 类似 coreutils, 但是效果更好.
* oskernel: 用户态程序, 将符号输入作为 syscall 参数.

KLEE 还可以用来测试功能等价, 或称交叉验证: 简单地 `assert(f1(args) == f2(args))` 即可. 于是
* 类似 automatic theorem prover *(dzy: 你还不是依赖 SMT, 顶多做了副作用和环境处理?)*
* 证明优化的正确性
* 证明 bugfix 不引入新的 bug
* 证明 `forall x . f_inv(f(x)) == x`


# RELATED WORK
* 已有的 traditional symexe:
  不考虑 environment, 或者 environment 处理不好.
* path explosion 的处理:
  - search heuristics: BFS, generation search, hybrid concolic testings
  - compositional test path
* model checking: manual effort

