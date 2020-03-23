# COLO: COarse-grained LOck-stepping Virtual Machines for Non-stop Service
高可用.


# 基础
背景:
VM replication, 指把 Primary VM 的状态复制到 Secondary VM 中.
Nonstop service 中完成应用无关的 fault recovery.
考虑 client-server 模型.

已有方法:
lock stepping. 开销太大.

解决方法:
无需细粒度的 lock stepping, 只需保证 PVM 和 SVM 对于同样的请求, 生成同样的响应 i.e. 观测等同.
如果响应不同, 那么把 PVM 状态拷贝到 SVM 中, 等响应相同了再 commit 响应.


# INTRODUCTION
Software based replication: **VM** replication PVM 状态复制到 SVM 中.
文章中使用 Xen 的 *incremental* state transfer.
一旦 PVM 检测到错误状态, 就用 SVM 代替. c.f. hardware replication: 贵.

lock-stepping: 粒度在
* 指令:
  1/7 的速度. 现在处理器有高度的不确定性 (OOE etc), 但中间状态的差别并不意味最终输出不同.
* 周期性:
  只有在周期 checkpoint 才能 commit 所有 response, 否则发送错误 response. latency 太大.

COLO:
* inbound packet 发给 PVM 和 SVM 都有.
* 检查 outbound packet, 只有 response 出现差别的时候才 synchronize.



# BACKGROUND
Remus: active-PVM / passive-SVM.
SVM 平时不执行, 只接受 state transfer.
软件 heartbeat monitoring, 发现 failover 就切换到 SVM.



# DESIGN
client-server 模型: 用 request-response 系统描述.
但是考虑有非确定性指令存在 (IO, 中断, TSC 等),
因此 response 不止取决于之前的 request, 还取决与这些不确定的指令.

不确定指令的效果越来越累积, 就可能造成 response 的不同.
这时就是 COLO 出场做 replication 的时候.
replication 发生的频率和 workload 有关.
CPU-bound 的很少会有 divergence.

COLO 基于 Xen. PVM 和 SVM 所在结点的 domain0 里都运行 COLO.
SVM 的 COLO 接受 SVM 的 response, 发给 PVM 的 COLO, 让 PVM 的 COLO 做检查.

COLO 需要考虑 per TCP connection response comparison.
因为结点上可能跑多个 VM 有多个 TCP 连接, 合到一起 response 就可能乱序.
还如, e.g. 某 VM 发 packet 后 200ms 另某 VM 还没有 packet, 那么就需 checkpoint.

希望修改 TCP 使得尽量保持 PVM 的 SVM 的 similarity, 减少 checkpoint.
还如, TCP 的 header 中不确定元素 e.g. timestamp, 即使 payload 是相同的.
需要修改 Guest OS 的 TCP 协议栈.

同步内存的时候, 如果是 passive checkpointing, 那么只需要传送 $D^p$ (PVM 修改了那些页).
但是 active checkpointing 中, SVM 的执行引入了 $D^s$, 所以 checkpointing 需要传输 $D^s \cup D^p$.
为了高效, SVM 保存上次 checkpoint 的状态, 每次只需传送 $D^p$ 即可.

根据 fail 的时机不同, 有时可能丢包 e.g. 确定让 PVM 返回 packet 后, PVM 真正返回 packet 前.
这种依赖 TCP 重传.

存储设备的处理:
* local 存储 - 作为 VM 内部状态:
  - checkpoint 就还需要传输存储设备. 传输耗时.
  - COLO 实际选择的方法. 监控存储访问, 只传输增量.
* local 存储 - 存储设备在 VM 之外:
  - COLO 需要比较 SVM 和 PVM 对各自 local 存储的访问. 触发更多 checkpoint.
* remote 共享存储 (remote 独占的情况 trivial):
  - remote 存储需要也提供 nonstop 服务.
  - 发给 remote 存储的请求被 COLO 比较, remote 发回来的响应传递给 PVM 和 SVM 两者.


# EVALUATION
* 修改的 TCP 协议栈本身不会有太大变动. (i.e. 需要和 COLO 结合)

* microbenchmark 相对 native / Remus 的效率, 以及修改的 TCP 协议栈能增加

* 针对 #vcpu 和 #task 的 scalability
  - 针对 #vcpu 不过, 但 #tasks (i.e. #thr) 太多的话下降 -- dirty page 太多.

* failover 时间

evaluation 章被有一节说 performance limiting factor:
disk snapshot, 调度导致的 #task scalability.



# RELATED WORK
* OS/app-specific 的 fault tolerange / nonstop service.

* TCP-specific 的 nonstop service

* 多线程程序的 determinstic replay.

* VM replication: 指令级别, 周期性, 优化 checkpoint 中 memory transfer.



# 问题
* lock stepping 的开销有多大, **相对 native** 来说?
没写... evaluaton 只比较了 native, Remus 和 COLO.
而 Remus 是 periodic checkpoint, 而非 instruction level lock-stepping.

* 为什么说
> More importantly, the result of memory accesses in a multiprocessor system is
> typically non-deterministic ...
这里说的 result 包含什么? 延时吗?

似乎是 [分布式的概念](http://www.cs.cornell.edu/courses/cs5414/2018fa/notes/week5.pdf).
nondeterministic instruction 是要访问环境的指令, (尤其是 I/O, trap).
那么如果地址空间有 memory mapped processor specific 的区域, 上面的一句话就可以成立了.
