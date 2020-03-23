Singularity: Rethinking the Software Stack
------------------------------------------------------------------------------

# INTRO
操作系统的实现中, 很多决定都来源于上世纪 60/70 年代的遗留.
不能匹配现代的硬件, 架构, 需求.

三个选择
* 安全语言
* 可靠的验证程序	sound program verifier
* 改进的架构, 缓解错误传播


# FOUNDATION
三个关键的架构
* software isolated processes: 进程执行不受外部干扰
* contract based channels: 高效安全的进程通信
* manifest based programs: 约束 SIP 对应的程序

## SIP
提供进程的 context.

SIP 包含
* 一系列内存页, 包含数据和代码
* 一堆线程
* 一个 security identity + security attributes

各个 SIP 之间不共享数据 (至少可写数据不共享), 
通过 channel 通信. channel 被 contract 约束.

SIP 代码是 sealed $\to$ 没有 动态链接和代码生成

最简单的模型可以是内核和一系列 SIP 在同一实地址空间运行

SIP 有
* 低通信代价
* 低创建/销毁代价
* 低切换代价
$\to$ 可以用来做细粒度的隔离和拓展

## CBC
CBC 是双向 channel. 有两端, 称为 Imp 和 Exp.

方向是, out 用感叹号表示, 从 Exp 到 Imp; in 用问号表示, 从 Imp 到 Exp.

Contract 包含
* 消息描述: 方向+参数
* channel 状态及转移: 状态 x 消息 $\to$ 新状态

NIC 例子:
* NIC 驱动暴露两个接口 NicDevice 和 NivEvents 给网络协议栈


## MBP

