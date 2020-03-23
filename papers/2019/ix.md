# IX: A Protected Dataplane Operating System for High Throughput and Low Latency 
------------------------------------------------------------------------------

# Introduction and Background
已有操作系统中的网络栈和高性能 data center 需要的网络需求不匹配.
  - 大量小包
  - 低 tail latency
  - 高连接数

IX: 分离
* control plane: 系统配置, 粗粒度的资源提供
* 打他跑了plane: 运行网络栈和应用逻辑

输入包通过哈希分派到队列, 每个 dataplane instance 独占某些队列
所以不需要同步


data center 应用, 如 redis, RPC
* 毫秒级别的 tail latency
* 高 packet rate, 都是小包
* 保护: 单服务器上多个服务之间的隔离
* 资源效率: 系统 load 变化很大

硬件发展很快, 高性能 CPU, 内存和 NIC 投入使用, 硬件不是瓶颈.
瓶颈在 OS, 它原来并没有针对 data center 应用设计.

已有的解决方法
* 用户态网络栈: 主要是安全, 应用 bug 可能损坏网络栈.
* 用 RDMA 而非 TCP: 需要特别的 adapter, infiniband 等等
* 替换 POSIX API: 效果不明显
* 改进 OS: e.g. Linux 的驱动轮询

# Design
防火墙, 负载均衡等将网络栈和应用结合成 dataplane.
* (run to completion) 同时对包进行协议处理和应用操作.
  做完一个包再做下一个.
* (synchronization free) 使用无同步操作来提高多核 scalability,
  将网络流量分派到多个队列中, 队列间无同步.

IX 分离 dataplane 和 controlplane, 
* control plane: 就是 Linux kernel. 复用资源, 调度资源.
* dataplane: 每个 dataplane 有其独占的核. NIC 队列分派给核.
每个 dataplane 运行单个应用, 在单个地址空间中.

隔离 controlplane, dataplane 和应用是通过硬件虚拟化完成的.
control plane 类似 host OS, dataplane 类似 guest OS, 而应用类似 guest 应用.

IX 也用 run to completion 思想. 不再需要 buffer, 优化 locality,
并且发送响应包的速率不超过应用处理的速率, 因此系统变慢之后 peer 能很快知道.

IX 用 batching 提高 packet rate, 网络栈的每个阶段都有, 包括 syscall, hardware 和 API.
并且 batching 是 adaptive 的.

# Implementation
每个 dataplane 支持单个 多线程的应用, 

# Evaluation

# Discussion

Fig.6: 一般来说 batching 导致 throughput 变好但是 latency 变坏.
IX Adaptive batching 能够优化 throughput, 并且低 load 情况下不影响 latency.
