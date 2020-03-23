# REPT: Reverse Debugging of Failures in Deployed Software
> OSDI'18 best paper
> Weidong Cui, Xinyang Ge, Baris Kasikci, Ben Niu, Upamanyu Sharma, Ruoyu Wang, and Insu Yun. 2018. REPT: reverse debugging of failures in deployed software. In Proceedings of the 12th USENIX conference on Operating Systems Design and Implementation (OSDI'18). USENIX Association, Berkeley, CA, USA, 17-32.

# INTRODUCTION & BACKGROUND
## 问题
怎么做生产环境下的 debug?
* log: 有用但开销大
* memory dump: 信息有限, debug 能力弱

## 已有工作
* automatic root cause diagnosis:
	自动发现导致错误的语句.
	光是语句还不够, preconditions etc.
	可能需要修改源文件.
* failure reproduction:
	符号执行, model checking etc.

## reverse debugging
* 即 record/replay.  已有的 full r/r 代价高.

在 REPT 中
* 硬件辅助来做 control flow tracing.
* 二进制分析 + memory dump 来恢复 data flow.
	**恢复的数据不一定精确, 希望尽量精确.**

特别注意, REPT 不要求绝对精确 -- 那是不可能的. 所以有些方法其实是启发式的.

## 挑战
主要是执行过程中不可避免的信息丢失.

* **不可逆语句**: 如 `xor %eax %eax`.
	使用前向执行而非后向执行来恢复信息.

* **unknown memory writes**: 有的数据是无法恢复的.
	对不可恢复的数据指向的地址写, 就是 "不知道写到那里了".
	使用 error correction 技术.

* **并发的写内存**: 共享内存被并发地写.
	如果值是从并发的写中取得的, 限制它的使用.


# DESIGN
* 执行流 (动态) 是执行的指令序列 $I_i$.
* 状态包含内存和寄存器, $S_i$ 表示 $I_i$ 执行后的状态.
	$I_i$ 能修改的所有东西都在状态中被包含. 即有
$$
	S_i = f_{I_i}(S_{i-1})
$$
* 可逆指令: 若 $f_I$ 可逆, 则称 $I$ 是可逆指令.

## 单个执行流, 只含不访存的可逆指令
状态被限制到有限的寄存器中.
从 $S_n$ 一直向前, 能够精确计算任意 $S_i$.

## 单个指令流, 只含不访存的指令
参见 Figure 1.
结合前向和后向分析, 最初在所有 $S_i$ 中将所有寄存器标记为未知.
然后从 $S_n$ 执行后向分析, 然后从 $S_0$ 执行前向分析, 直到达到定点.

归纳易证该过程正确, 且收敛.

## 包含内存访问的指令
**PAPER 的重点**

* 寄存器指令是直接寻址 i.e. 访问哪个寄存器, 直接在指令编码中有
* 访存指令是间接寻址, 可能连访问的哪个地址都不知道

保守的方法是, 如果写内存指令的目标是未知的, 那么任何内存地址都可能被写 -- 但是这过于保守.

REPT 的做法是, 如果写内存指令的目标是未知的, 那么不管这条指令, (当它没有执行一样) 继续如前分析.
直到发现冲突, 因为我们忽略了一个写内存操作. 发生冲突的时候, 后向分析的结果优先.

### 推断干涉图
* 结点表示某时刻的内存位置和寄存器, 如 `rax@In`, `[rcx]@In` 等.
	结点有附属的值, 最初是未知, 推断将值计算.

* 边 $u \to v$ 表示, (后向分析中) $u$ 的值可以用于推断 $v$ 的值.
	如果 $v$ 的所有前趋节点均被推断, 则 $v$ 可以被推断.
	- vertical edge: 不同指令间结点的边, 只会从后面指向前面 (因为主要是后向分析)
		e.g. DU chain 中 U 指向 D 或者之前的 U.
	- horizontal edge: 同一指令结点间的边.
	- paper 中还区分了 address edge 和 value edge. 用来
    1. 计算 dereference (confidence) level;
    2. invalidate 结点的时候, 对于 address edge, 不仅它要设为 unknown,
      而且它所有的 vertical edge (DU chain) 都要清空.

* 因为信息的缺乏, 干涉图可能是错的, 即冲突, 包含
	- edge conflict: 确定了某个未知的内存位置, 造成内存位置上的某条 edge 失效.
		具体地, 就是打断了某个 DU chain.
	- value conflict: 推断出来的结点值和已存在的值不等.
    只有当推断出来的 confidence level 更高的时候才认为发生了 value conflict.
	- 冲突会发生, 因为在推断分析的过程中, 信息越来越多,
		之前对未知信息的处理是忽略, 现在发现之前的某个处理错了.

* 发现冲突的时候, 停止推断分析.
	发现冲突说明之前的分析有些错误, 需要 invalidate 某些信息.
  - 对于 (寄存器中的) 值, 直接设为 "未知"
  - 对于地址, 不仅要设为 "未知", 而且还要删除所有的 vertical edge.

## 处理并发
也就是有多个执行流了.

利用硬件特性: 硬件在指令序列中会插入 timestamp.
两个 timestamp 之间的 chunk 称为 subsequence,
所有的 subsequence 关于 time stamp 构成了偏序.
(其中 interleaving subsequence 没有序)

将这个偏序全序化, interleaving 之间的序随便定.
如此将问题归约为单执行流的情况.

但是需要限制, interleaving subsequence 之间的写内存结点,
不参与任何 vertical edge. 因为我们认为它们的顺序事实上是不确定的.

# IMPLEMENTATION
* 针对用户态程序实现.
* 硬件支援用 Intel PT, 实现成一个 driver.
* 实际中, 还依赖控制信息来推断数据, 如 brcond.
* syscall 的处理:
  - volatile register 设置为 unknown.
  - 不处理内核写内存的情况, 内核写内存当成 missing memory writes.
* offline binary 分析作为 WinDbg 的部分.


# EVALUATION
也是用回答问题的方式来做
* (micro evaluation) 恢复数据的 accuracy
  - 需要 ground truth: 可以慢, 但是必须可靠.
  - 多数情况 accuray > 90%, 但是有一个例子中是 75%, 因为 dump 之前有众多内存重写.
  - 基本上, 从 failure 其实越远, accurary 越低, 不过也要取决于 workload.
* (micro evaluation) 效率开销
  - 执行时 tracing 的: 低 (2%), 有前人研究.
  - offline binary 分析的: 通常挺快的.
* (macro evaluation) 多大程度上帮助 debug
  - 不太清楚这部分写的什么.
* 部署情况如何?
  - 讲了个两年老 bug 在几分钟内被修复的故事


# DISCUSSION
## 主要的限制
* 不能完全恢复
* trace 不够长
* 移动设备无法用 -- 没有 Intel PT.

## 未来可能的方向
* 现在只处理用户代码, 内核代码还要考虑中断等.
* 只在单机上做了, 那分布式系统...?

## 其他的想法
* 好像可以通过 REPT 提供的执行历史来做 automatic root cause 分析,
  不过执行历史会有错, 所以还需要处理这些错误.


# RELATED WORKS
## Automatic root cause diagnosis
* 统计方法来取得 root cause
* 处理 missing memory writes 和 concurrent memory writes: 递归地运行测试

## Record / Replay
* 前人工作有使用 full r/r, 针对 deployed system 代价很大
* Castor: 插桩 + 硬件 tracing
* 是否假设 bug 可以被重现


# Best Paper 分析写作
1. Introduction
	- 问题是什么
	- 现有什么解法, 都有什么问题
	- 我们的解法是什么
	- 简要地, 我们怎么做到的
	- 我们遇到的最大 challenge 是什么
	- 简要地, 我们如何解决了这个 challenge
	- 我们实现了怎样的系统, 做了什么 measure, 效果如何

2. Overview
	- Problem statement
	- Design Choices
	- Challanges

3. Design
	- 先比较形式化地描述术语
	- progressively 描述系统, 先是最简单的情况,
		越来越复杂 (+不可逆指令, +内存访问, +并发)
		注意它这里三个增加的特点就是之前提到的三个 challenge
  - 前三个都给出了例子.
  - +内存访问是文章的重点, 包含干涉图和错误恢复的概念.
    篇幅较大, 所以继续分成两个小节.

4. Implementation
  - 系统组成部分清楚地分成 tracing 和 binary 分析, 所以实现也单独说这两个.
  - 用了什么技术, 代码量多少, configuration 怎么样
  - 实际实现和之前介绍的模型设计有什么区别? e.g. syscall 的处理

5. Evaluation
  - 有哪些 metric
  - 每个 metric 具体是怎么计算的
  - 数字结果是什么, 如果有奇怪的数据要解释为什么系统不能很好处理

6. Discussion
  - 指出自己方法的局限
  - 包含了一些其他的想法, 如 future works, possible applications etc

7. Related work
