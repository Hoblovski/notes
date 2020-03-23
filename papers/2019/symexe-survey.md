# A Survey of Symbolic Execution Techniques


*writing: 这个作者喜欢用4, 5行的长句, 而且抽象地不给例子, 读起来很费力. 不要学他*


# INTRODUCTION
* 目的: 获取程序性质
* 对每个控制流路径保存
  - path condition: 一阶布尔方程. 被 branch 控制.
  - 变量到符号值的映射. 被复制控制.
* model checker: path 是否可达, 其上是否有非法性质

## 例子
* 仍然是简单的无环单线性函数, assert 表示禁止路径.
* 不能被静态分析确定的值都用 symbol 表示, e.g. 参数, syscall 返回值
* 状态包含 $(s, \sigma, \pi)$, 其中
  - $s$: 现在在 CFG 中哪里
  - $\sigma$: 将程序变量映射到符号的表达式 $\sigma: PVar \to SymExpr$
  - $\pi$: path condition

## 问题
* 都用树来表示, 其实为什么不用 CFG 来表示呢?
  不过实际上是等价的, CFG 表示起来还稍微不直观.
* 似乎 SSA, CFG 等概念都可以使用


# BASICS
## Concolic Execution
* 传统 symbolic execution 主要的挑战: 循环, 外部函数调用, 副作用...
  因此提出将 concrete execution 和 symbolic execution 混合到一起

### Dynamic Symbolic Execution:
将输入变成 concrete 的值, 每次执行都是 concrete 的执行, i.e. conexe drives the process.
但 path condition 还是 symbolic 的, 并且也保留 $\sigma$.
记 $\sigma_c$ 为符号到具体值的指定.

每次执行相当于 explore 了一条从根到某叶子节点 $l$ 的路径, 而我们的目标是遍历所有节点.

于是从 $l$ 向上, 找到 *上面某个* 分叉节点 $u$, 其 path condition 是 $\pi_u$, 并且分叉条件是 $c$.
之前我们是所有 concrete 的值 $\sigma_c$ 满足了 $\pi_u \land u$ 才最终走到 $l$ 的.
现在求解 $\pi_u \land \lnot u$, 让 SMT 给出一个特解 $\sigma_c'$,
这样我们就能探索 $u$ 的另一颗子树了.

寻找 *上面某个* 节点的方法: (DART) 最低; (SAGE) 最大化能遍历的新节点.

对外部函数的处理展现了 DSE 的主要问题:
* **false negative** 加入黑盒函数 $f(\alpha)$.
  显然若条件中有 $f$, 则 SMT 不能求解.
  这种只好跑完一次之后, 随机输入, *希望* 遍历到新节点.
* **path divergence** 比如 `mutate(&x)`, 如果 SEE 不知道 `mutate` 会修改 `x`,
  那么它就会使用错误的符号作为 path condition / $\sigma$.

### Selective Symbolic Execution
主要思想: 通常, 我们只关心 symbolic execution tree 中的部分路径.

包含执行模式的切换: e.g. 函数调用的时候切换
* C $\to$ S $\to$ C: 其实 S 又要 symbolic 执行又要 concrete 执行.
* S $\to$ C $\to$ S: 影响 completeness 和 soundness.
TODO

## Path Selection
是一个 heuristic 的问题, 但是具体的 heuristic 要看目标 (发现 bug? 完善测试?)

* DFS / BFS: DFS 比较省内存, 但是被递归和循环限制了 (要拓展完才能继续).

* random path selection: 使用 heuristics, KLEE 偏好被访问较少的 path. 例子如下.
  - 一种 heuristics 希望最大化 coverage, 如 KLEE 的 coverage optimize search.
  - 一种 heuristics 是希望达到 interesting states, e.g. buggy-path first

## Symbolic Backward Execution
从程序中间某处反向符号执行到程序开始, e.g. 用来发现能达到该处的输入.

挑战:
* inter-procedural control flow analysis.
* 如果 caller 众多, 执行代价就很大.

## Design Principles of Executors
效率
* 保持资源使用不超限
* 避免重复工作, 尽量重用工作

并发执行不同路径 (online), 和回溯算法 (offline), 分别在两个方面做得更好,
就像 BFS 和 DFS 一样. 另外 BFS 还要注意不同执行路径的隔离, 因为它们共用 e.g. OS.
也有 hybrid 方法, (dzy: 就像 BFS 开始然后发现内存快不够了, 就切换成 DFS 一样)


# MEMORY MODEL
需要处理指针和数组了, 因此要把内存位置映射到 SymExpr (当然也包括 ConExpr).
程序变量也是内存地址, 因此 $x \mapsto e$ 其实是 $\text{addrof}\, x \mapsto e$.
最大的问题就是, 对符号地址的访问.

## Fully Symbolic Memory
* state forking: 读写符号地址, 则考虑所有可能的读写位置.
  比如写了 `a[i]`, 则 fork 出 `i` 等于 0 .. `len(A)-1` 的子树.
* if-then-else: 某些 solver 可以处理 if-then-else.
  比如写了 `a[i] = V`, 那所有 `a[X]` 变成 `if X == i then V else 原值`.
  读也是同样的 `if ... then ... else ...`.
* 上面两种方式都是, 如数组 `a[2]`, 那么状态中就加入 `a[0], a[1]`.

* fully symbolic memory: 实际中可能的地址通常较小,
  所以 KLEE 等使用 fully symbolic memory. 但是最坏情况, 解决方法有
  - 紧凑的表达内存: 如写 `a[i]=V`: 加入一个 predicate over `a`: `on X=i => V`.
  - 限制指针: 要么 NULL, 要么指向已知的对象.
  - address concretization: 不 sound

## Address Concretication
* pointer 可能指向的地址太多.
* 把 `pointer<T>` 的值 concretize 成 NULL 或者某个 `T` 实例.
* unsound

## Partial Memory Mapping
* 写的地址总是 concretize, 如果可能的值范围较小则读的地址是 symbolic 的

## Lazy Initialization
TODO


# INTERACTION WITH THE ENVIRONMENT
* 环境分为系用环境和应用环境.
  系统环境如 os kernel, file system...
  应用环境如 android, (java) swing, 外部 (可能闭源的) 库...

* concretizing external calls:
  - 限制了 fully symbolic analysis
  - 并且无法预测副作用从而可能导致矛盾状态

* 对环境建立抽象模型
  - KLEE: 符号文件系统, 在 syscall 层面做.
  - AEG: 文件系统, 网络, 环境变量, syscall...
  - cloud9: reordering, delays, packet dropping...

* 抽象模型: 难写 (尤其如果环境像 android 之类), 不一定准确, 容易过时.
  - 自动生成抽象模型, 用 program slicing / program synthesis
  - S2E: 还是让程序和实际的环境交互, 不过用多条路径.
    需要防止副作用互相干扰 $\to$ QEMU + virtualization.
  - DART: 外部函数: 模拟返回随机值.


# PATH EXPLOSION
* 主要的问题就是循环和递归, 在循环/递归条件有符号值的时候出现的问题
* 最简单的就是把循环限制到固定次数内

## Pruning Unrealizable Paths
* a.k.a. path condition 的 eager evaluation -> 因为每次循环跳转的时候都求解条件
* 减少多次求解的重复工作

## Function / Loop Summerization
* function summaries: *compositional approach*.
  重用对函数的分析结果. 用一个方程 $\phi_w$ 来描述函数的作用.

* loop summaries: 使用动态计算的 loop pre/post-condition.
  - 简单的循环如 induction variable 每次固定 +c.
    复杂的包括嵌套循环, 以及包含 if 的循环.
  - 有用 FSM 和 template 的, 但是都没说清楚.

## Path Subsumption and Equivalence
作者没有说明什么是 path subsumption.

* interpolation:
  某些路径不满足我们关心的性质, 希望抽取这些路径的特点, 然后避免这些路径.
  craig interpolation (1957): 没看懂. 不过这个技术在 model checking 中很常用.

* subsumption with interpolation:
  只关心导向 `assert(false)` 的路径.
  那么只关心 path condition 不隐含不导向 `assert(false)` 的路径.
  具体怎么用 interpolation 的?

* subsumption with abstraction:
  合理地描述状态, 抽象状态?

* path partitioning:
  使用 dependency analysis.

## Under-constrained Symbolic Execution
* 不管环境, 将要测试的代码隔离测试. 可能导致 false positive: 没有考虑上下文.
  用 concrete execution 取消部分 false positive.

* 分析的时候遇到错误, 除非 context insensitive 的, 否则认为我们是 under costrained,
  将错误条件的取反加入 path condition, 看之后是否遇到矛盾.

## Exploiting Preconditions / Input Features
* 输入不需要完全 symbolic, 可以有 precondition 限制范围.
  之后就可以跳过不满足 precondition 的 branch 了.
  加 precondition 让 solver 负担重, 但是大大减小了状态空间.

## State Merging
* 合并状态和 path: 如 `int a = (b) ? (c) : (d)` $\to$ 如 Figure 10.
  于是变成了 DAG 而非 symbolic execution tree 了.
  减小状态空间, 可能加重 solver 负担.

* 两个极端例子是, 完全 merge: 用一个方程表示整个程序, 以及完全不 merge.

* merging heuristics: 决定什么时候 merge 什么时候不 merge.
  如果没有复杂的操作, solver 能简单地处理 merge 后的条件, 就 merge.

## 使用其他程序优化技术

* program slicing

* taint analysis: 检查那些 path 包含 tainted value.

* fuzzing: e.g. symexe 找出输入的 constraint 然后取反让 fuzzer 找到新输入.
  用 fuzzing 让 symexe 找到更深的状态.

* branch predication: 大幅减小状态空间, 用 `ite(a, b, c)` 代替 `if-then-else`.

* type checking

* program differencing

* compiler optimization: symexe 也要做如 loop invariant value hoisting, strength reduction 等.
  可能让 symexe 更加困难, 而且效果并不好.


# Constraint Solving
* 传统是 (一阶谓词) SAT, 虽然 SAT 是 NPco, 但是现实中很大一类都是 tractable 的.
* 通常使用更 expressive 的 SMT: 变量不只是二元的 true/false 了. 支持算术.
* STP / Z3
