# A Survey of Research into Mixed Criticality Systems
* MCS 关键的问题: 为了安全的隔离和为了效率的共享之间的划分平衡.

* 例子: 无人机 UAV.
  - safety critical: 被民航安全局 certify
  - mission critical: 如拍摄, 被系统设计者 sign off.
  - less critical: 寻路

不同 criticality 决定开发过程不同严格度.
高 criticality 的部分对资源有更高的优先权.
所以关键问题就是, 如何处理资源的 scheduling.
我们不关心 scheduling, 只关心模型和系统如何分割.

## MC Models
系统包含多个不同 critlevel 的组分, 每个组分包含不同的突发任务.
* seperation: 不同组分的任务不能互相干扰
  (dzy: Maojj 的 noncrit 的大量请求不能干扰 crit)
  包含 temporal seperation 和 spatial independence

之后的主要是 scheduling 了

## 和我们相关的
1. 3.3 Shared Resources
  * Resilient Mixed-Criticality Systems: 信息不能从 low crit flow 到 high,
    除非 high crit 能够处理不可信的数据.
  * 另外还如, 低 crit 对共享数据加锁太久 / 或者执行时间太长.

2. 4.3 Communication
  * 多核 MCS shared resources. 如 bus, 网络等.
    太多的 low crit 也可能占用太多带宽
  * bus: memory throttling scheme; monitoring & control protocols;
    bus arbiter (TDMA...); memory controller;
  * networks: back suction; wornhole routing; network on chip;
    controller area network...

3. 6. Systems issues
参见其他论文. TODO



# A Modular Safety Case for an IEC-61508 compliant Generic Hypervisor
J. Perez 做的.

## 基本信息
MCS 有需求, 但是 certification 耗时费力. 稍微改动一点软件就要全盘重新 certify.
希望通过 modular safety case 的方法能够完成 safety case 的重用.

## 概念
* MCS: 包含硬件, OS, 中间件以及应用.
A Methodology for Safety Case Development
* certification: 是一个 process.
  展示某系统/应用 etc <对某任务> 是安全的, 因为它符合某标准 e.g. IEC-61508, ISO-26262

* safety case: 是一个 argument.
  论证系统 <对于某应用> <在某环境> 是安全的,
  通过说明系统满足了安全性质以及各种 risk 被很好地 mitigate.

## Generic MSC (Modular Safety Case) for Hypervisors
完成 spatial independence:
* 把不同 partition 分配到不同物理地址

不同 partition 间的 communication:
* 通过 hypervisor 提供的服务 e.g. port-based communication
  c.f. ARINC-653.

## 问题
1. 硬件是 MC 的吗? 如何应对硬件 failure?
似乎硬件是 MC 的, 因为它提到了 fail safe.

并且内存也是 MC 的. 周期性地检查内存 CRC.

> In addition, the partitions have to check that the content of their memory
> areas are not accidentally or intentionally modified. A safety partition
> periodically calculates the CRC of XtratuM code and compares it with the
> off-line calculated value in order to detect any modification




# Mixed Criticality Systems: A Review
* 什么是 criticality: designation of the level of assurance against failure needed for a system component.
* MCS: 有不同 level

系统参数, 如 WCET 和 criticality 有关. 同一任务, 若 criticality 更高, 则 WCET 更长.
导致许多调度研究的结果不可用.

开山 paper 的关键一句话就是
> [t]he more confidence one needs in a task execution time bound (the less
> tolerant one is of missed deadlines), the larger and more conservative that
> bound tends to become in practice.

## Questions
1. 传统的 MC 是什么含义, 为何和 scheduling 有那么大关系
后者: 参见上一篇.



# Flying autonomous aircraft: Mixed-criticality support in seL4
> Flying autonomous aircraft: Mixed-criticality support in seL4
> at linux.conf.au, Sydney, January, 2018
> Gernot Heiser

## MCS 介绍
* cyberphysical 系统: 安全重要, 但是 certify 很昂贵.
  certification cost 可能和 loc 是线性, 甚至超线性的.
* 传统: 物理隔离, 不同功能用不同 microcontroller:
  - 空间, 重量, power 限制
  - 通信
* 希望: 一个 microcontroller 完成工作.
  - 问题: 没有物理隔离了, 只能由软件隔离
* certification: 某个人/实体声称系统能够安全工作.
  - 整个过程很复杂

* OS: 单体 kernel 中 driver 都放在 kernel 中, 无法隔离, 这种内核难以用于 MCS.

* Heiser: 还是要用 microkernel. 没有 communication channel 相连的部分就是隔离的.
  (dzy: side channel 也隔离了吗?)

## seL4
* 隔离和 channel 的实现: 通过 capability.
  - 被保护的指针, 封装了对象引用和访问权限.
  - 细粒度的访问控制, 明确的信息流动
  - 调用类似 `err = method(cap, args)`
* seL4 的验证:
  - functional correctness: 数学模型声明内核的语义, C 实现, 用 formal proof 证明两者对应.
  - translation correctness: 不信任 C 编译器, 故也验证 C 实现和最终的二进制代码.
  - isolation property: 数学模型: 证明包含保密性, 完整性, 可用性.
  - 包括 最坏情况执行时间 也有证明, i.e. 任何内核操作的耗时上界是已知的.
* 问题: 上面都是 spatial isolation. 没有 temporal isolation.

### MCS Temporal isolation
* lc 无论如何, 一定不能影响 hc. 尤其是执行速度.
* 还是使用改进的 scheduling model.
* seL4 传统就是 256 hard priorities, 高优先级进程可以占用 cpu.

* 例子: sensor control loop / network driver
  - scl: 每 100 ms 必须运行一次, 一次花费几 ms
  - nd: 突发性的, 但是处理时间很短 (几 us)
  - 结果: 需要允许 nd 抢占 scl, 但是 nd 不被信任所以抢占完也只能执行最多几 us 的时间

  - uav: 飞控是 hc, 和地台通讯是 lc 吗, 处理有某个程序是 server (hc).
    e.g. 不同 criticality 系统共用一个软件部分 (如 server), 防止 lc 独占 server.
    如 lc 系统让 hc 系统 DOS.

* MCS 系统的需求可验证的处理 spatial 还有 temporal isolation
  - 能够保证 hc 的 deadline
    (dzy: Heiser 认为 lc 是 malicious 的, 和我们的不同)
  - 安全的共享资源
  - 高效的利用资源: 空闲时运行 lc. -> 物理隔离资源利用率很差
    也是 cost.

### seL4 temporal implementation
* capability 加入 time control
* 把 time slice 替换成 scheduling context capalibity
* scheduling context obj: period + budget (lessthan period)
  完成限制 CPU 的访问.  (C=250, T=1000 那么可以使用 25% 的 CPU 时间)

* 内核总运行最高优先级, budget 非空的线程
* budget 为空的进程只能等到下个 period 运行, 到时候 budget 刷新
* 参见 seL4 sched.png.
  - 对 hc 和 mc 保证 deadline 和 budget

* client 调 server, server 用的时间由 client 的 budget 出.

### evaluation
* scheduling: 20% penality. (800 vs 1k cycles)
* macro benchmark: 说明的确满足 budget 保证, 以及 wcet 保证.

(dzy: Heiser 系统中硬件被信任. seL4 是 nonpreemptable 的)

## summary
* seL4 + MCS 很好, 效率接近传统非隔离系统.

> Heiser: Testing is for weaklings. We do formal verification.



# T-Visor: A Hypervisor for Mixed Criticality Embedded Real-time System with Hardware Virtualization Support

Currently there are hypervisors on MC, aiming to support GPOSes besides RTOSes.

Existing hypervisor on MC (Xen, KVM)
* often have large TCB.
* don't consider real-time requirements

Built on top of ARMv7.

Full virtualization with hw support.

Inter VM communication:
* shared memory pages
* virtual interrupts

## Questions
1. Is hypervisor itself mixed-criticality?
nope.

2. What are the key designs of such hypervisors?
...

3. Real-time: again scheduling?
yes.



# Hypervisor-Based Multicore Feedback Control of Mixed-Criticality Systems

> [f]ocus on the execution control of partitioned mixed-criticality systems
> running on top of a hypervisor.

restricts noncritical bus access with a performance monitor.

## Questions
Same as previous 1. and 2., and

1. What's feedback control?
...


# VOSYSmonitor, a Low Latency Monitor Layer for Mixed-Criticality Systems on ARMv8-A
不是我们的 mixed criticality.

是一个 monitor layer, 而非标准的 hypervisor.

* 使用 TrustZone

TrustZone:
* 是一种硬件架构
* 保护 CPU, 总线和外设.
* 隔离比虚拟化更强
* 将软件和硬件分为 secure 和 nonsecure world
* 不需要 dedicated security processor

RTOS 在 secure world 跑.

## Questions
1. Is hypervisor itself partitioned?
不, 他跑在 EL3, 总是 secure 的.

2. How does the hypervisor adapt to hw supported mixed criticality?
... hw no support for MC.



# Towards the design of certifiable mixed-criticality systems
因为需要 ceritification, 导致传统调度中某些方法无法使用.



# Mixed Criticality in Multicore Automotive Embedded Systems
不是我们的 mixed criticality.

只要 free from interference, 即低关键的错误不传播到高关键,
就可以 ISO 26262.

AUTOSAR OS: (AUTomotive Open System ARchitecture OS)
* 汽车界的 OS 标准
* 区分 trusted/non-trusted app
* 能达到一定的 MC: time 可以, 但是 memory 和 communication 不好.

## Questions
1. 他是否提到硬件支援 MC? 是否有涉及软件支持?
没有



# Mixed-Mode Multicore Reliability
硬件随机失效, 可以用 DMR (CPU 配对) 探测, 这种是高覆盖率但是高代价.
应用分两类, reliable apps, 这种需要用 DMR 跑,
以及 performance apps, 它们不太关心偶尔的硬件错误, 可以不用 DMR.
另外 os 必须跑在 reliable mode, 即使 performance app 的 syscall.

保护 reliable apps 不受 **硬件随机失效** 的影响.

分页内存等不可靠了, 因为可能硬件错误使得写奇怪的物理地址
  e.g. TLB 中页表项 PL 位翻转.

* 使用硬件 Protection Assistance Buffer / Table (PAB / PAT):
  perf core 上冗余地验证 **写** 内存,
  类似于在 perf core 上, 内存访问校验也是 redundant 的, 但是这么做没有影响效率.
  不过没有验证 **读**, 称为 Future Work.

* perf app 调 syscall 就要进入 reliable os, 两者不在同一 core 上 (即使 syscall).
  因为它的 OS 本身是 reliable 的, 这本身又源于它不区分对待 perf 和 reliable app.
  使用 cpu 虚拟化.

假设 memory hierar 中大部分都可靠, 如 ECC. 但是私有 L1\$ 不用.

reunion: master / slave core, 作为一个 logical core.
  slave core 有自己的 private L1\$, 结果只和 master 对比, 不改变外部状态.
  额外的流水线阶段 CHK 对比对方的 fingerprint.

PAT/PAB 的实现就是, 每个物理页包含一个位, 是否只有 reliable 才能访问.
PAB 是 PAT 的 \$. 在 reliable core 上, 不用 PAT/PAB.

另外单个 core 可以有时 perf 有时 reliable, 所以还要保护 register.
  直接复制保存 reliable app 的状态, 到 scratch pad memory.

另外和我们的区别是, 他, 包括 ARM, 都是单个 core "可配置".
我们没有考虑这一点, 认为 core 确定是 C/NC 的. 不过我们是从 OS 出发, 因此可以看成 VCPU.

## Questions
1. 他的硬件是 MR 的, 那么有没有写 "软件如何适应" 呢?
没有.

2. 他里面, 是否有一个软件 (一个进程) 分成不同 criticality 呢, 然后类似 SFI?
至少系统软件是没有的
> all privileged software must execute in reliable mode (page 3)



# Resilient Mixed-Criticality Systems
> There are strong objections to data flowing from low- to high-criticality
> applications unless the high-criticality component is able to deal with
> potentially un-reliable data (Sha 2009),
>
> MCS Survey

* 对象: cyber-physical system.
* 问题: resilient against 1. sw design faults; 2. hw failures; 3. errorneous agent command;

* make assumptions explicit and checkable

* *use simplicity to control complexity*:
  软件过于复杂以至于难以 certify:
  做完之后 check. 并且一个简单但可能低效的备份.

* *simplex architecture*: 上面方法的一个例子.
  - high assurance subsystem: 简单可靠的通过验证的逻辑.
    验证过, 或严格开发过程的应用软件; 可靠的 RTOS; 可靠的硬件;
  - high performance subsystem: 高性能多功能, 但不一定可靠, 一旦失效就要让位于 HAC.
    应用可以使用新的, 不可靠技术 e.g. 神经网络; OS 和硬件可以 COTS.
  - decision controller: 通常 HPC 管理系统, 但是 decision controller 监视系统状态,
    一旦偏离 HAC 设定的预定范围 (operational constraints) 就将控制转给 HAC.
    decision controller 也要是可靠通过验证的.

> To protect the safety core, it should be run in a different real-time virtual
> machine.

在 Sha01 的 Simplex 中提到了 HPC 的 upgrade. 并且
> The HPC can use the HAC’s outputs, but not vice versa.
>
> Sha01

* 软件模型的形式验证:
  - 先写 AADL model, 转成 Real Time Maude, 然后对着 fault model 做
    model checking.
  - dependency inversion: safety-critical 组件依赖更低 critical 的组件.
    可以被 AADL 检测.
  - *employ but not depend*: 使用 NC 的数据/命令, 但是 C 的安全不依赖
    e.g. C 中有自己的安全措施, 检查, 防止 NC 的 fault/malicious 数据/命令.

最小化要 certify 的安全组件, 为了经济性.

## Questions
1. 有那些设计模式
Simplex.

2. 如果要能够防止 sw design faults; 是否做了某种形式的 isolation?
看起来像是完全隔离的.

3. hw failure 是否意味着, hw 是 mixed 的? 总不可能所有 hw 都是 nc 的吧?
如果要处理 hw failure, 必须有 realiable hw.

> If the application required the tolerance of hardware failures, then
> fault-tolerant hardware must be used.

4. 目标是软件还是硬件还是软硬件?
软硬件结合, 如 decision logic 一定包括为了接受 plant 反馈的电路, 以及可能包括
(验证的) 软件控制切换 HAC 和 HPC.



# Using Simplicity to Control Complexity, Sha01
基本和 Sha09 相同, 稍长一点.

high assurance/performance subsystem 都有自己的硬件, sys, app.

> The high-assurance and high-performance systems run in parallel, but the
> software stays separate

## Questions
1. 软件硬件是否隔离?
子系统在这里说的是较完整的子系统, 有完整的隔离.



# A safety concept for a railway mixed-criticality embedded system based on multicore partitioning
目的: 解决 safety certification on multicore 上的诸挑战
方法: 引入 safety concept 基于 multicore partition.
方法有效性: concept 符合诸标准所以可以解决 certification 问题.

传统 federated arch 包含隔离的子系统, 隔离导致 ECU/connector/cable 的太多 Product CSW.
加上 multicore 出现了. 因此 MCS.

## Federated
系统包含两个 self-contained 子系统: C ETCS, 以及 NC traction cntl.
EVC 是 SR 的, 和 NSR 的 node 完全隔开.

为了达到某个 safety requirement, 设计中有哪些对应的部分.

## Single-core with virtualization
把 NSR node 继承进入一个 EVC 计算节点.

在一个验证的 hypervisor 之上, 不同 functionality 放到不同 partition 中. *什么是 partition?*

## Multicore processor with hypervisor
TODO 实在看不懂


## Question
1. HW 是否是 MC 的? 若是, 内存是否是 MC 的?

2. 什么是 safety concept? 其中是否有 address isolation? 是否是我们的半边读半边写?
没看出来.

3. multicore partition 是什么?
没看出来.



# A safety concept for a wind power mixed-criticality embedded system based on multicore partitioning
和上文基本一样的标题, 只是 railway 换成了 wind power.

contribution
* definition of safety certification strategy,
  for IEC-61508 compilant industrial MCS,
  based on multicore partitioning

多核 + virtualization 使得软件 partition 变得可能.
软件 partition 支持 fault containment, partition isolation.

提到 partition 之间的 spatial & temporal isolation.
* spatial: 使用 MMU (但是对于 hypervisor 自身不行)
* temporal: 更加 challenging, 因为 COTS processor 的 performance 并不稳定

IMA: 航控领域, **单核上**, 不同应用封装如不同 partition, 其间 temporal & spatial isolation.

还是三步: federated -> multiprocessor -> multicore.

TODO 还是看不懂

## Question
1. HW 是否是 MC 的? 若是, 内存是否是 MC 的?

2. 什么是 safety concept? 其中是否有 address isolation? 是否是我们的半边读半边写?

3. multicore partition 是什么?



# An Embedded Hypervisor for Safety-Relevant Automotive E/E-Systems
问题: AUTOSAR 不能很好支持 mixed integrity 的 sw partition 间的 full seperation.
方法: virtualization; micro-kernel based hypervisors; MMU-less;

把 ECU 继承到强力的多核平台 e.g. DCU 上.

hypervisor 实现共享 hw 访问; VM 间交流;

MMU: 使得应用地址被隔离.

也就是, 原来 ECU (supervisor + user) 上跑了 AUTOSAR OS, 现在让多个 AUTOSAR OS
跑到 DCU 上, 但是 supervisor 被 hypervisor 占用了. 因此需要 porting.

## DESIGN
把多个 ECU 集成到一个 DCU (上面有一样的 ECU, 因此减小 porting effort),
但是会有一些 seperation 失效 e.g. 原来是串口通讯的分割开的, 现在是 shm mc processor sys.
可能破坏 freedom from interference.

需要将 error/failure (iso26262) 封装到 VM 内部. 方法是 micro-kernel like.
hypervisor 设置 spatial & temporal isolation.

它们的 RTA-HV hypervisor 是 type-1, 使用了 microkernel 思想. 所有的 VM 运行在 user mode.

## IMPLEMENTATION
硬件:
* 三个核 (TriCore). 没有专门针对虚拟化设计. 有 MPU 而没有复杂 MMU.

虚拟化支持不好 -> paravirtualization.

spatial isolation:
* 只有 4 个 MPU protection region.
* 它说的 MPU virtualization 是什么?
  - AUTOSAR OS 在 baremetal 上有 supervisor mode, 现在只能在 VM 里面.
  - region 0 给 hypervisor, 1 给 VM 的 trusted, 2 给 VM 的 untrusted.
    region 0,1 被 hypervisor 设置, region 2 可以被 VM 修改, 但是需要在其静态被分配的范围内
  - hypercall 用来让 VM 切换 region 0 和 1 (相当于同一个 privlege 中不同的段)


## Question
1. 如果提到 MC, 是硬件是 MC 还是任务是 MC?
任务一定是 MC 的. 硬件提到了 "wearing out"...
硬件错误也考虑了. 解决方法包含 HW watchdog, AUTOSAR end2end protection.

> We concentrate on sporadic faulty states, which were not detected and could
> appear during system runtime. These are often caused by hardware, but
> erroneously-engineered software can also be a source.

III.B: 硬件产生 intermittent fault,
到 error 可以通过 HW (MPU / watchdog) 和 SW (AUTOSAR end-to-end protection),
到 failure 可以被应用逻辑或者 additional control mechanism 处理.

2. Hypervisor 为了适应 MC 吗? 做了什么?
全文没有细提 mixed criticality, 重点是 ECU 到 DCU 之后 VM 的 porting,
以及不同 VM 间的 isolation. 但是 isolation, 就算不是 mixed criticality, 也要做.


# Distributed architecture for developing mixed-criticality systems in multi-core platforms
问题: Partitioning, in heterogeneous scnearios.
方法/贡献:
* Virtualization 用于隔离 & middleware 用于通讯.
* 针对 partitioned 分布式系统中 multicore platform 上, 发现 commmunication
  的挑战以及提出解决方法.

XtratuM hypervisor: IEC-61508 compilant hypervisor 的那篇提到的.
* 空间隔离: 是不同 partition 间的. 它们分配到不同内存区域中.
  对共享内存限制非常严格.
* 通信: ARINC653 中 sampling & queueing.
* 故障隔离: hypervisor 中一个模块来检测/处理 fault.

## ARCHITECTURE
* spatial isolation: 是指, 如果数据在 partition 之间传送, 必须是 authorized.
  - ARINC-653 的 channel

* partition 间的 communication 的实现
  - Hypervisor 提供的 Virtual Network

## Oth
在 第 5 章中提到了 adaption of
* OS: 访问硬件要用 hypercall 而非 native hw.

## 背景
* middleware in distributed systems:
  - 提供 OS 以外的其他服务, 使得分布式系统中的诸组分沟通.

* DDS standard in distributed systems
  - 定义: <匿名, 异步, (producer/comsumer解耦合的)> decentralized middleware architecture
  - publisher 提供新数据. middleware 将数据 *透明* 地传播到 subscribers.

## Questions
1. 考虑了那些问题, 尤其是有没有考虑 hw failure. 并且是如何处理这些问题的?
没有提到, 不过它基于 XtratuM 所以考虑了. 参见 XtratuM.

2. 考虑到 MCH 以及 isolation, 系统软件做了那些设计?
没有提到 MCH. 关于 isolation 还是 spatial isolation 一文中提到的.

3. communication 是什么 communication?
inter/intra-partition 软件 communication.



# An introduction to Functional Safety and IEC 61508

## Functional Safety
术语
* EUC: Equiment under control
* EUC Control System: 接受输入型号和操作员指令, 发送输出信号给 EUC
* Safety-related system: 实现一系列 safety function, 需要达到期望的 safety integrity
* EUC risk: risk without enforcing any protective measures.
* PES: Programmable Eletronical System = E/E/PE

* Harm: 实际给人造成的伤害, 间接或直接
* Hazard: Harm 可能的源头
* Safety: freedrom from unacceptable risk
* Safety function: E/E/PE 系统实现的功能, 用于保证 EUC 的安全状态
* Risk: 包含 severity 和 likelyhood
* Residual risk: 所有保护手段应用后, 余下的 risk

* safety integrity: safety-related system 正常完成 safety function 的概率
  - software / hardware safety integrity
  - safety integrity 的值就是 SIL (safety integrity level)

## IEC 61508 overview
标准的 scope 是帮助系统设计者完成 safety management.

functional reliability 不等于 safety. 因此需要分离 safety requirements/design 和 functional requirements/design.

safety requirement 被一系列 safety function 满足. 这些 safety function 实现在
safety-related systems 中.  当然具体的 system 可能是软件的, 硬件的,
甚至非电气的.

certification 也可以看成是 safety assessment, 通过 building safety case 来展示 safety claim is valid.

safety case 是一个 logically presented argument, 需要被 documented 支持.
并且 documentation 必须是 structured, 顶层是 safety argument.

## 一个工业中实际例子
> The bursting disc is considered to be 100% reliable but its operation is not
> considered desirable for environmental and public relations reasons.

即使有一些 safety function 可以依赖, 有时并不希望完全依靠它们 -- 类似我们的硬件 MMU.

## QUESTIONS
1. Maojj 在 Introduction 写了一大堆 functional safety, 那么我们做的真的和
   functional safety 有那么大的联系吗?
能够说明一些基本概念和硬件保护为什么不够.

2. 相关的开发过程是有限制吗?
根据 risk assessment 的结果和 SIL 要求, 确定 development process.
因为通常开发的 systematic failure rate 是不可测定的, 所以通过 sound engineering 减少 failure rate.

3. certification 过程如何?
TODO

4. 如何结合软硬件的保护手段的叙述?
TODO


# Towards the Design of Fault-Tolerant Mixed-Criticality Systems on Multicores

## 基本信息
问题: 已有的 MCS 设计关注时域隔离的scheduling, 但是 functional safety 忽略.
方法: 提出设计 methodology, 保证 schedulability 以及 safety, 在 SMP 上.
  将两者合并起来 i.e. fault-tolerant scheduling.

## 方法
* 将 safety requirement 用 pfh 表示
* 使用 task re-execution / replication 以及 system reconfiguration

fault tolerenace 的两种方法: 论文中均使用
* 运行时 sanity check
* task replication

system reconfiguration:
* critical 任务重试若干次均失败: drop / downgrade noncritical 任务

引入 task killing 和 service degradation. 提高 safety 和 schedulability.

## 问题
1. 是否考虑硬件错误?
考虑. hardware / software transient errors, dual criticality platform.

2. 是否考虑单体软件隔离?
不考虑. 说的是 fault tolerant scheduling.

3. 它所说的保证 safety 具体是通过什么? regular checks 吗还是 fault tolerant?
re-execution, replication, reconfiguration.

> For functional safety,  various stresses (e.g. hardware/software errors) [21]
> need to be mitigated through different hardening techniques including task
> re-execution and replication [9,  19].

保证方法: taskre-execution / replication


# Fault-Tolerant Platforms for Automotive Safety-Critical Applications
## 基本信息
automotive 中的 chip fault tolerance, 在单个 chip 上

## 内容
* FCR: fault containment region, 一系列 component, 无论 region 外有什么电子/逻辑错误, 都正常运行.

SOC 上的 fault tolerance, 多核:
* lock-step dual processor
  - 不能判断是那个 processor 出错

* loosely-synchronized dual processor:
  - RTOS 在双核上运行, 软件检查错误
  - critical task 是 replicated 并且 regularly checked

* TMR architecture:
  - majority vote. 内存加 ECC.

* dual lock-step: fail-operational


## 问题
1. 有什么能够写到 related work 中
它的几种 failure resolution.



# Software Composability and Mixed Criticality for Triple Modular Redundant Architectures

## 基本信息
TMR + (MCS, Composability).

## 背景和概念
* composability: 将系统分解为 FCR. 减少 certification 的限制.
* TMR: 三个 FCR.<F3>

* fault containment module:
  TMR 的概念. 注意和传统说法 fault containment region (FCR) 不同.
  - 相关概念:

* failure containment region:
  composability 的概念. 和前面的概念是正交的.
  - 相关概念: non-interference

## Software Composability
* composability 的组成部分
  - FS 对 IE 的限制: FS contract.
  - non-interference i.e. 同 IE 中不同 FS.

通常最有效地 non-interference 通过每个 FS 是独立的 FCR 完成.

完成 non-interference 称为 failure containment.
failure 指的是某个 FS 的非正常工作.

# TMR + Composability
* TMR: 主要为了防止 hardware random failure

* TMR 为了处理 common cause failure (单个 fault 导致两个 module 失败),
  需要不同 module 间的 fault containment.

* TMR 需要完成 recovery 保持持续运行. 从 2 个对的重建那个错了的.

fault 指的是 hardware failure.

提出
* static TMR composable arch. 提高 utilization.
* dynamic TMR composable arch. 没有 FS 到 board 的静态划分.

## 相关工作
几个部分:
* MC + composability
* HW composability
* separation kernel vs virtualization
* SFT (software fault tolerance).

当代 TMR:
* software time redundant: 编译时候做 instruction replication + voting.
  - 针对 hw transient fault. 无需硬件支持. 效率低.
* hardware lock-step TMR:
  - hw transient / permanent fault. 用不了 COTS 硬件. 利用率低.
* software incremental / lockstep TMR:
  - 在不同 board 上运行 middleware, 它做 synchronization + voting


## 问题
* 是否和我们的相关 (软件隔离, 单体软件分割)

不一定有单体软件隔离, 但是有软件系统设计.

* 所谓 composability 是什么和我们有什么关系?

似乎就是 modular certification

> Should parts of the system change, substantial effort is necessary for
> re-certification, as the corresponding certification process has to be repeated
> for the whole system to demonstrate that safety is still guaranteed.
> Composability aims to overcome this limitation of certification.

* 似乎主要讲的是 composability + TMR, MC 在子节标题都没出现?
  MC 的意义何在?

  不同 component 是 MC 的要 compose 起来.

* compose 之后如何隔离?

  硬件上的就不说了. 基本上通过对系统设计来隔离. 没有特别提到如地址空间隔离.


# misc. websites

[link](https://community.arm.com/iot/embedded/b/embedded-blog/posts/announcing-the-arm-cortex-a76ae)

> next generation ADAS and autonomous driving systems will consist of multiple
> applications running at different levels of safety criticality (i.e.
> different levels of ASIL).

Cortex-A76AE, 相对 A76, 有 functional safety 和 added application flexibility.
也是针对自动驾驶.

* dual core lock step:
  - ASIL D hardware diagnostic coverage requirements
* memory protection
  - 在 cache 中, ECC
* RAS (reliability, availability, serviceablity)

两种 mode: performance mode (SMP) / safety mode (lock-step). 所以称为 flexibility.


------------------------------------------------------------------------------

[link](https://www.arm.com/solutions/automotive)

自动驾驶的等级
1. L0: no automation
2. L1: Driver assistance
3. L2: Partial automation
4. L3: Conditional automation
5. L4: High automation
6. L5: Full automation

Functional safety

ASIL

------------------------------------------------------------------------------

[link](https://www.zhihu.com/question/27719391)

Functional safety 的含义
> Absence of unreasonable risk due to hazards caused by malfunctioning behavior of E/E systems.

即, 即使系统失效, 也应当 operate safely

ISO 26262: 汽车 E/E 系统. 对汽车研发, 生产, 测试等要求.
* V 型开发模型

ASIL (automotive safety integrity level), 从低到高是 ABCD, 另有非安全相关的 QM.
如果失效可能很小, ASIL 就高.

------------------------------------------------------------------------------

The Functional Safety Imperative in Automotive Design: White Paper

faliure: systematic / random hw faults.

------------------------------------------------------------------------------



# 概念
## Safety Function

> Accordingly, the standard defines the safety function as a function of a
> machine whose failure can result in an immediate increase of the risk
>
> The SISTEMA Cookbook 6
