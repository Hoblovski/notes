# SYMBOLIC EXECUTION FOR TESTING COMPLEX SOFTWARE


# INTRODUCTION
防止软件错误:
* code review
* manual / random testing: 麻烦 / 随机效果不好
* dynamic tools: 构建在 testing 之上
* static analysis: poor reasoning

提出 symexe, 自动生成 testcase.
* 高覆盖
* 能生成实际的具体 testcase, 方便检查 / 验证.


# OVERVIEW
## Path Exploration and Test Generation
符号执行的大致思想.
* `make_symbolic`: 标记内存位置
* 当 program path 终止, i.e. 主要是 `exit` 和 `assert` 时,
  用 STP 求解 constraint 得到具体的输入.
* 支持指针和数组. 通用的检查如
  - 访存之前检查是否无效内存 (NULL / 越界)
  - 除法取模之前检查是否除数是 0

## EXE
针对 C 代码. 流程设计
1. CIL 源码级插桩
  - 在 branch 处插入 fork, 就用 OS 的 `fork()`.
  - 危险操作前加入检查
  - 每个操作中检查, 是否有符号算数; 若有则移交操作给 EXE runtime.
2. 编译, 和 EXE runtime 链接

其他设计
* 用 testcase replay 发现 EXE 中的问题.
* program space exploration 的算法
  - 简单的随机 DFS: 容易 stuck, 但是不会 fork bomb
  - *best-first search*:
    每次 explore 产生新状态后, 新状态记录到状态池中.
    每次从状态池中按照某 heuristic 选择下一步 explore 的状态.
  - EXE 每次 explore 不止做一步, 它 explore 直到经过 4 个 branch.
    其中 branch 都是随机选择而非 fork.
    $\to$ 较少 explore 的状态继续执行更能发现新状态.
  - EXE 的 heuristic 针对 coverage 优化.

## KLEE
和 EXE 的区别
|                     | KLEE                                                               | EXE                                    |
| ---                 | ---                                                                | ---                                    |
| state fork 的实现   | 单进程 + 状态池                                                    | 用 OS 的 `fork()`                      |
| 执行什么代码        | LLVM IR                                                            | C 源代码                               |
| 怎么执行            | 解释执行                                                           | native 执行 + EXE runtime              |
| 有无环境符号化      | 有                                                                 | 无                                     |
| 每次 explore 的粒度 | 执行一条 IR                                                        | 经过 4 个 branch                       |
| COW 的粒度          | object level                                                       | page level, 如果 OS 的 `fork()` 有 COW |
| 探索策略            | 交替 *coverage optimized best-first search* + *random-path search* | 通常就是 best-first search             |

另外的区别
* KLEE 引入 time slice 限制 symbolic process 执行的时间, 防止常使用耗时操作的 symbolic process 独占 cpu.

和 EXE 的相同
* branch 同样 state clone (i.e. state fork)
* 危险操作同样插入检查

改进方面
* scalability
  - COW, fork IPC
* reliability
  - LLVM

## 局限
* constraint solving
* loops: 现在还是暴力做法
* symolically sized object: 现在只能 concretize size
* symbolic pointer dereference
* 如果有 concretization / infinite loop truncation 就不 sound 了.
* high level design errors
* floating point
* 汇编函数, 多线程, `longjmp`.


# Memory modeling
## Bit-Level Modeling of Memory
内存被视为无类型的字节串, STP 希望适应这样的模型.
* STP 中只有三种类型: bool, fixed_len_bitvec, array_of_bitvec.
* C 中有类型, STP 只有在操作的时候才把 bitvec 临时 coerce.

内存分为 concrete 和 symbolic, 执行过程中不断有 concrete 地址变为 symbolic.
* `make_symbolic` 将内存变为 symbolic (unconstrained).
* `v=e` 让 `&v` 处 `sizeof(v)` 的内存变为 symbolic, constraint 是 `e`.
* `*ptr`: `ptr` 是 symbolic, 设 `*ptr = arr[idx]`, 其中 `idx` 是 symbolic,
  那么将 `arr` 的内存也变为 symbolic, constraint 是初值.
  (dzy: 为什么? STP 需要分析 ptr 可能指向的内存?)

变为 concrete 也有
* 符号失效 e.g. out of scope
* constrant 变得只有一个值
* concretization

内存更新
* 对于简单的变量, 是 strict substitution by name,
* 对于数组和符号下标, 让每个数组有一个 update list. STP 原生支持这种操作.
* STP 不直接支持符号指针, 将指针解引用变成数组访问 `ptr -> baseobj[index]`.
  - EXE: 插桩中追踪指针赋值 (以及指针算数), 让每个指针都 bound 到某个数组.
    i.e. 确定 base object.
    无法处理 pointer-integer-cast.
  - KLEE 中, 检查 `*ptr` 处所有对象, 检查是否可能 `ptr` 指向它.
    更慢更可靠.

## Constraint Solving Optimizations
STP 的瓶颈在 array reasoning, 所以有优化
* 常量下标的数组访问, 变成对象访问
* 符号下标, 用 approximation-refine 的方法, 没细说

irrelevant constraint elimination:
* branch condition 有时是独立的.
* 将 constraint 分成独立子集, 按照使用那些符号操作数
* 求解 branch condition 的时候只需要传入相关的 constraint 子集

subset caching:
* 把 STP 求解的结果 (求解给出的具体解) 缓存
* 同 KLEE paper

两个优化是相关的, 加到一起效果最好.


# Redundant Path Elimination


# Environment Modeling


# Evaluation


# 问题
* 和之前相比有那些改进
* 能避免那些问题? 高层设计语义?
* 是怎么处理 memory, path, environment 问题的
* EXE 和 KLEE 的区别. KLEE 好在那里?
