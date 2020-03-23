# Xen and the Art of Virtualization
paravirtualization 重要论文.
paravirtualization 开山之作是 Denali.



# 基础信息
背景: 好用的 VMM. 目标有
* 效率
* 安全: 空间, 以及更重要的是时间 (i.e. 效率) 的隔离.
* 能在 COTS 硬件上运行
* 减少 porting 工作

另外 resource-managing 包括控制资源访问, 计费等.



# INTRODUCTION
挑战:
* VM 间隔离, 最重要的是 performance isolation i.e. QoS
* VM 上 OS 和应用的 heterogenity
* 效率

方法:
* 修改操作系统源代码. 不过不改 ABI, 不改客户应用.
* 对于 performance isolation: 在底层完成资源复用 ??


# APPROACH & OVERVIEW
Full-virtualization:
* 低效, x86 不好做 full virtualization
* 有时希望 Guest OS 知道它跑在 VMM 上, 并允许它看到物理资源 e.g. (hp) timer

用 domain 表示运行中的 Guest OS 实例.

## 内存虚拟化
x86 是硬件 TLB (c.f. MIPS 软件管理 TLB), 且 TLB 非标签化 i.e. TLB ent 没有 ASID.
每次切换 AS 都要完全清空 TLB. 因此, Guest OS 管理硬件页表.
* 创建, 修改用 hypercall. 不允许直接改 GPT.
* 允许 batch update.

分段类似, 类似管理 GPT 一样管理 GGDT, GLDT ...

dzy: 看起来似乎 GPA 就是实际的物理地址.

## CPU 虚拟化
Guest OS 特权级更低.
* x86 有 4 个 ring, 让 Guest OS 在 ring 1 执行就好. 基本上没有人用 ring 1.

中断异常从 Xen 绕一圈. Guest OS 看到的异常上下文是 Xen 设置的.
* syscall: 为了效率, 不需要从 ring 0 绕.
* pf: ring 1 的 pf handler 不能直接读 CR2. Xen 把 CR2 放到异常栈上面了. 还是要绕

## 外设 IO 虚拟化
使用异步 shm, IO 数据传到 Xen 之后透明地对 Guest OS 即可用.

## Resource Access Policy
不在 Xen 中而在另一个授权 domain "domain0" 中.
其中运行 control plane 用户程序 (c.f. data plane 在 Hypervisor 中完成).
Xen 提供 mechanism, 而 control plane 用户程序提供 policy.

Xen 给 domain0 提供特别接口, 用来管理 domain (创建 etc), VD (VIF, VBD etc).
尤其, 创建新 domain 在 domain0 中做, 更 robust, Xen 更简单.



# DETAILED DESIGN

## Xen 和 Guest OS 的通信
通信控制流的变动如
* Guest 到 Xen 的请求使用同步的 hypercall.
* Xen 给 Guest 的提醒使用异步的 event.
  - 类似 signal

通信 payload 的传输使用异步 IO ring. 其为一个 ring buffer:
```
     unack resp processin unack req  free
          |         |         |        |
head      v         v         v        v     tail
<----+---------+---------+--------+------------->
     |         |         |        |
    response   response  request  request
    consumer   producer  consumer producer
     g:rw      x:w       x:rw     g:w
               g:r                x:r
```
其中元素为 descptr, 包含请求 ID (等于响应 ID) 以及 guest OS 分配 buffer 的指针.
不要求 processin 中的请求按序执行. 方法类似 sliding window.

## 子系统
* 调度: BVT 算法. work-conserving. fast dispatch (为了及时处理事件).
* timer: 给 Guest OS real/virtual/wall clock. 其中 virtual clock 类似 user time.
* 内存分配: 物理内存静态分配给 domain.
  - 实现 balloon driver
* 分页: 不用 SPT.
  - 硬件直接使用 GPT.
  - 但是 Guest OS 看到的 GPT 是只读的, 要用 hypercall 改.
  - Xen 对每个 frame 维护其类型 e.g. PD, PT, GDT, regular-frame...
* 网路: 虚拟网卡.
  - 使用 pattern-action 作为 rule. domain0 维护.
* 磁盘: 虚拟块设备.
  - domain0 控制 VBD.
  - Xen 保存一个映射表 `(VBDIdent, offset) -> (physDevId, physSecNum)`


# EVALUATION
需要修改的代码量在 x86 base 的 1% 左右.

1. Xen 和其他虚拟化方法, 以及 native 的效率差别.
  - CPU bound: SPEC CPU; IO bound: 编译 Linux; 数据库; dbench; 综合: SPEC WEB.
  - macrobenchmark: Xen 优于其他虚拟化方法, 接近 native.

2. Xen 提供的 performance isolation
  - 数据库, WEB, 大量访问磁盘, forkbomb + membomb
  - 隔离很好. c.f. 若作为 Linux 下进程, 机器变得根本不可用.

3. Scalability
  - Xen 的中断均衡使得多核下 Xen 的表现相对变好
  - VM schedulability 基本和 Linux proc schedulability 同.



# RELATED WORK
* full virtualization: VMWare, Connectix 兼容性好, 效率不好
* paravirtualization: IBM, Denali (isolation kernel).
* 虚拟化在分布式系统
* 效率隔离: vs 语言 (SPIN), proff-carrying code (PCC), 中间件
* 语言 VM: JVM



# DISCUSSION
主要就是 future work. 略.


# 其他
Xen 做网络发送的过程我们 MCS 可以参考

> To transmit a packet, the guest OS simply enqueues a buffer descriptor onto
> the transmit ring. Xen copies the descriptor and, to ensure safety, then
> copies the packet header and executes any matching filter rules. The packet
> payload is not copied since we use scatter-gather DMA; however note that the
> relevant page frames must be pinned until transmission is complete. To ensure
> fairness, Xen implements a simple round-robin packet scheduler.


# 问题
1. 何谓 low level? 尤其是底层完成资源复用, 以及 low level virtualization.

2. GPA 是 PA 吗?

> ... page-table memory: all subsequent updates must be validated by Xen. This
> restricts updates in a number of ways, including only allowing an OS to map
> pages that it owns,

如果这么说, 看起来就是 GPA 等于 HPA. 否则完全可以 GPA 是整个物理空间.

并且 3.3.2 提到硬件直接使用 GPT, 所以 GPA 是 PA.
