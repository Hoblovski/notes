# Parallel Symbolic Execution for Automated Real-World Software Testing

# Problem
## Research Question
符号执行
* *scalability* 分布式, 并行化
* *applicability* 新的符号化 POSIX 环境接口
* *usability* 符号化的测试套件, 简化测试套件编写

## Existing Solutions, Their Troubles, c.f. Proposed Solution
主要和 KLEE 对比.

* KLEE 只能执行单线程程序. 重点也不是 scalability.

* 环境建模: KLEE 也是将具体环境替换成模型
  - 但是不包含所有 POSIX 环境 e.g. threads etc.
  - 还有 concretization 但是其有明显缺点 i.e. 环境污染

* 测试套件: 一般软件的测试套件是手动编写然后祖传
  - 已有方法是 fuzzing -- 但是似乎 out of scope

以及找 bug 还有 model checking 和 fuzzing.

## My Proposed Answer
* 分布式 scalability: 不同核跑不同的执行树的 branch

* 并行程序处理: 全局状态变成 threadlocal 的.

* POSIX 环境: 只是工程难度 -- 正好被击中

* 简化测试: 也很 trivial -- 比如把输入作为 `make_symbolic` 最后 `assert` 一下即可.



# Proposed Solution
## General Idea
分布式, 并行化
* 将执行树分化, 统计地使得各个 node 分配到的诸子树 *load balance*
  - 执行树是动态探索的, 故需要动态分配
  - 包含诸 worker 和一个 load balancer.
  - worker 定期将统计数据发送给 LB.
  - LB 主动告诉 worker X 去抽取 worker Y 的 workload.
  - 具体 job 发送是 worker 之间 peer to peer.
  - 只发送路径而非 program state, 需要 dest replay.
  - 执行树结点分为 dead / fence / candidate nodes.
    candidate 又分为 materialized (确定 program state) 和 virtual (还未
    replay).
  - load balancing 通过保证各个 node 的 job queue 长度差不多完成

环境处理: 建模
* 需硬件支持的特性都需修改 symexe engine 故设计 SEE 很重要也是主要的挑战
  - 对于无状态 / 只读的 external call, cloud 9 允许 concretization.
    保证不同探索状态间的隔离.
  - 类似 klee, 有 symbolic libc 和 libc 共存, 而且加入一个 symbolic 'OS'
    进程 / 线程 / fd / 网络操作使用 symbolic libc.
    memcpy 等简单的就使用 libc.
  - 需要支持 program state 中有多个 address space -- 符号化 fork 的结果
  - 线程支持 cooperative sched.
  - 实现是针对两种关键的 buffer 做了符号化.

符号化的测试套件
* 数据直接使用 `cloud9_make_symbolic` 即可
* 符号化网络和 fault injection (syscall / posixcall)

## Main Obstacles and Resolution
* 基于 KLEE. KLEE 中很多全局信息. 改成 per state 的.
  - allocator

* rc 指针导致连续递归的 destructor call
  - batching

* 执行树结点的字段越来越多, 缓慢而麻烦
  - layered tree



# Evaluation
## Metric
* applicability 能否应用到实际的复杂软件上
  - 只需要 POSIX. 几十万行代码亦可.
  - 什么软件找到了什么 bug

* scalability
  - 达到目标总体耗时 -- 基本 linear
  - per worker 效率  -- 分布式的开销

* effectiveness 就是能提升多少 coverage
  - 针对 KLEE

* 对于一个目标有多个手段的时候 (scalability / model), 应当做隔离分析


## c.f. Existing solutions
纵向比较就是和 KLEE.

横向比较是 num of workers.



# Retrospect
## Remaining confusions
如果每个环境都要建模, 认为非常麻烦. 能否自动生成环境模型?
* 似乎每个环境, 若非 trivial, 都需要大幅修改 SEE

他的方法是否能够处理数据竞争?
> Preemption occurs at explicit points in the model code, but it is
> straightforward to extend Cloud9 to automatically insert preemptions calls at
> instruction level (as would be necessary, for instance, when testing for race
> conditions).
我认为如果这么做虽然理论上可以但是开销不小?
> The latter case is useful when looking for concurrency bugs, but it can be a
> significant source of path explosion, so it should be disabled when not needed.
的确. 所以符号执行处理并发错误比较困难?
话说又想到另一类问题 -- rust 的所谓安全 (除了 type safety) 能不能被符号执行做? (valgrind 可以动态地做)

> In general, improving coverage becomes exponentially harder as the base coverage increases ...
* 我觉得不应该啊 -- line based 应该很简单吧 -- 或许是 SMT solver 的问题

## Future Work and Weakness of Proposed Solution
Evaluation 开头写了
> How do its different components contribute to overall efficiency (§7.4)?
结果 7.4 就提一个 load balancing.

e.g. Performance? Labor? Economicalness? Elegance? Compatibility? Usefulness?

## My main take-away


# Others
* 学习它的 Figure 4 画法.

## 细看 POSIX modelling
* model 不能做所有事情, 所以需要 external call.

* 为了支持 POSIX, 给 KLEE 加上如下特性, 通过符号 syscall 的形式提供
  - AS 隔离 :: 支持如 fork
  - 线程调度 :: 为了实现 threads

* 为了效率才是 cooperative scheduler.
  - 为了探测并行 bug 可以每执行一步都 fork, 一个线程一次

* 实现中没有太多提到 pthread 的事情

