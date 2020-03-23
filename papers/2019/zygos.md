# ZygOS: Achieving Low Tail Latency for Microsecond-scale Networked Tasks
------------------------------------------------------------------------------
# 背景知识
## DPDK
dataplane development kit 完成高性能包处理. 使用了
* 轮询而非中断
* 用户态驱动 -> 零拷贝, 绕过 kernel
* direct cache access -> 绕过缓慢的 DRAM
* 批量包处理 (ZygOS 中禁用了)
* 高效内存管理: huge-page, NUMA-aware

> CoNEXT'13: Scalable, High Performance Ethernet Forwarding with CUCKOOSWITCH
> https://haryachyy.wordpress.com/2014/11/05/learning-dpdk-intro/
> https://www.cnblogs.com/Anker/p/6156149.html

## Work Conserving Scheduling
比如, 有 CPU 空闲, 而且有 RUNNABLE 的线程的时候, 
如果 scheduler 总是把 RUNNABLE 分配给空闲的 CPU,
那么这个 scheduluer 就是 work conserving 的. 否则就不是.

> 维基百科

## NIC RSS
用来提高并行度, 为多处理器系统优化.
receive side scaling 就是, 把接收到的包按照某种方法, 比如哈希, 放到不同队列中.
主要是为了均衡的提高表现, 也可以用来优先处理部分包.

> https://www.kernel.org/doc/Documentation/networking/scaling.txt

## Control Plane vs Data Plane
* control plane 相关协议类似 RIP, OSPF, BGP, 用来决定流量发送到哪里.
* data plane 按照 control plane 决定的逻辑,
将接收到的流量发送到下一跳 (路由器 / 用户进程)

## Head of line blocking
计算机网络的概念. 队列的队头受阻, 导致之后的一系列元素受阻.
比如队头 packet 因为其出端口忙而等待, 导致后面所有 packet 等待,
即使他们的出端口空闲.

## IX
做 ZygOS 一帮人之前 2014 年做的.

## epoll


# 术语
* in-memory date service: 我的理解就是, 类似 redis, memcached

* 开环闭环: 是否使用反馈机制. 画出块图, 闭环系统有环而开环没有.

* FCFS vs PS: 单独地说这两个的时候, 我们不考虑多核.
FCFS 就类似 event-driven 并发处理, 每次用全部的计算力服务最先到的请求.
PS 类似一个请求一个线程.
按照线程并发执行假设, 所有请求同时被服务, 每个线程使用部分计算力.


# INTRODUCTION
为了满足 SLO (通常用 tail latency 定义), 数据都放到内存中.

high fan-in: 单个内存数据服务很多 server 同时访问它.
high fan-out: 单个用户请求可能需要很多服务支持

ZygOS: 高效调度. 上下文: 细粒度的 (high fanout) 内存中服务
理论答案:
	多核共享一个队列 比 每个核有自己的队列 要更高效
	对于 low dispersion 任务, FCFS 更好
			 high dispersion 任务, processor sharing 更好		TODO: ?
现实并非如此简单


# BACKGROUND
## RPC 的 scaling
IMDS 通常以 RPC 的方法暴露借口, 高效支持 RPC 的 scaling 就回到了 c10k 问题.

古代方法是每个连接一个线程, 但显然切换代价太大.

* **symmetrical 方法**: 如 libevent 和 libuv.
使用如 epoll 的非阻塞 API, 将连接分派到各个线程上.
*TODO*

* **asymmetrical 方法**: 如 nginx 和 gRPC.
前端线程负责网络 IO 和解析请求, 将请求放到一个全局队列中.
后端服务线程从队列中取出请求并且服务请求.

## kernel bypass 和 sweeping simplification
Dataplane 方法: IX, 用户态协议栈
* 绕过内核
* 轮询而非中断

关键问题是, 每个线程/核只能处理 NIC 转发给自己的请求.
一些情况下导致各个核负载不均衡, 增加 tail latency.
已有的解决方法只能解决持续的不均衡, 不能探测解决瞬发的不均衡.

## queueing theory
不均衡导致了 tail latency 增加
* 不同 NIC 接受包的速率. 源于 conection skew
* 请求 burst. 造成瞬发不均衡.
* 处理不同请求所用时间不同.

可以用排队论处理 -> Kendall 记号.
一般设置请求到达速率服从泊松分布, 处理请求的时间服从高斯分布.

文章用简单的排队论做了仿真, 有两个维度的比较
* 单队列还是多队列? 区别在于, 有多少 worker, 以及对于每个 worker 请求到达的速率.
* FCFS 还是 PS? 区别就类似事件驱动还是线程驱动.
启示是
* 单队列比多队列好, 就算后者使用 random 放入而非
* FCFS 比 PS 好


# Methodology
分析了两个 metric
* 理论 bound (忽略调度, 系统开销等) 和实际 bound 的区别
* 达到 SLO 时最大系统负载 (越大越好, 表示系统在重负载下也能低延时响应)

结论时, 当服务时间越长 (相对的系统开销占比越小), 实际系统越接近理论 bound.
IX 对轻请求服务较好, 但随着请求越来越重, Linux-floating 更好.


# 问题
* INTRO 何谓 dataplain [principles] 为何
* INTRO 何谓 sweeping simplification: 不切实际的过于简化的简化
* INTRO 何谓 head of line blocking
* INTRO 何谓 commutative API 为何
* METHOD 为何 IX 看起来是 centralized-FCFS 但是逼近的理论 bound 却是 partitioned

