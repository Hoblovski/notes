# Multiprogramming a 64 kB Computer Safely and Efficiently

# Introduction
为了支持进程抽象, 硬件需要提供
- HW privilege levels
- MMU
- (以及足够的内存)

在低功耗嵌入式场景下, 计算力高度被限制.
导致例如 OS 和用用编译到一起, 这样高效, 但是缺乏隔离, 也缺乏动态特性如分配内存.

内核使用应用内存 -> grant


# Background
* 软件平台: 其上需要支持多个独立的, 可动态加载的应用. 当前嵌入式 OS 不支持.

* 嵌入式环境下, 计算力特别受限制的计算力, 资源尤其是内存亦如此, 并且增长也不快.

* 最近的 MCU 提供了一个 "内存保护单元 MPU", 提供内核/应用和应用/应用的隔离.
  并非虚存/分段的方法. MPU 的保护很有限.

## 嵌入式 OS
有如下五个目标, tock 希望达成这 5 个目标, 以尽量小的 tradeoff.
* 并行
  - 原因: energy efficiency.
  - 已有方法: run-to-completion (simple + starve) / preemptive.

* 可依赖性
  - 原因: 运行环境不方便开发者介入.
  - 内存耗尽: 确定性的内存使用情况, 内存耗尽的可靠性.

* 故障隔离: 主要是 memory fault.

* 内存高效
  - 支持动态加载/分配内存.
  - 灵活, 可靠, 高效的 tradeoff

* 运行时应用更新


# ARCHITECTURE
代码分两种
* capsule: 内核内组成部分. 被语言限制. cooperatively scheduled.
* process: 外部进程. 硬件内存隔离. preemptively scheduled.

## Threat Model
角色有
* board integrator: 硬件设计, 提供访问硬件的方法/代码

* kernel component dev: 内核组成部分通过 capsule 提供.
  任何 capsule 代码可以被 board integrator 检视.
  capsule dev 不是完全可靠. (e.g. 不能非授权的硬件访问, 可以 cpu starve)
    dzy: i.e. capsule 开源而且是 rust 的?

* app dev: 提供 process.
  通常 app 的代码不能被检视.
  app 可能是恶意的, 更低的信任, i.e. 完全不可靠.
    dzy: 私有, 或者奇怪的语言?

* 用户: 安装/更新/使用应用.
  也不是完全信任的 e.g. 敏感的内核数据不能泄漏给用户.

## Capsule
* 一个 capsule 是 rust 的一个 struct 实例.
* capsule 不能动态加载.
* 所有 capsule 共享一个栈 -> cooperatively scheduling.
* cooperatively scheduling -> 可能 capsule 会耗尽 CPU 资源
  故 capsule 不应当执行耗时操作.
* 普通 capsule: 不被信任, 必须 type safe.
  不能访问其他 capsule, 不能访问应用内存.
  e.g. syscall capsule: 用户 syscall 翻译成内核服务.
* 少数信任的 core capsule: 直接和硬件交互.
  允许 type unsafe.
  e.g. MMIO -> 对应的 struct. 调度其.
* 不同 capsule 通过 type system 和 module system 隔离.
  编译期检查使得隔离代价很小.
* 内核调度是事件驱动的. 事件如硬件中断, 或 process 产生的 syscall.
  capsule 不能产生事件, 只通过普通的控制流转移.

## Process
* 硬件隔离. 包含自己的堆, 栈, 全局变量.
* 可用任何语言写. 类似传统的进程.
* 没有虚存
* 因为是 preemptively scheduling, 所以允许执行耗时的任务.
* 允许动态内存分配. 允许动态加载.
* MPU: 设置多达 8 个内存区域的 RWX 位.

## Syscall
* 非阻塞. 针对事件驱动.
* command: $process \xrightarrow[\text{整数类型的参数, 不用验证}]{\text{请求}} capsule$
* allow: $process \xrightarrow[\text{buffer, 需要验证}]{\text{请求}} capsule$
* subscribe: $process \xrightarrow[\text{callback closure}]{\text{监听事件}} capsule$
* yield: $process \xrightarrow{\text{等待 callback 唤醒}} core$
* memop: $process \xrightarrow{类似 sbrk} core$


# GRANT
* 问题: capsule 需要动态分配内存, 因为要服务 process 的请求.
* 已有方法: 静态确定 (不灵活); 全局内核堆 (不可靠, 不好错误隔离)
* grant: 内核堆中放到 process 的内存中
  - 一个 process 耗尽内核堆不影响其他 process.
    (dzy: 有点像 Lin Zhong 的 state spill?)
  - 简化并且可靠 process 退出后内核堆的回收
  - process 不能访问 grant 内存, 这个通过 MPU 实现
  - capsule 访问 grant 内存也要受限制 --
    grant 内存的生命和 process 一样长, 但是 capsule 的生命通常都要长于 process,
    故 grant 中的引用不一定有效, 需要限制指向 grant 的引用.
  - grant 内存是有类型的,
    一个在 grant 内存中的 `T` 类型的对象只能通过对应的 `Owned<T>` 访问
  - capsule 操作 grant 内存只能在一个作为 `Grant.enter` 的参数的闭包中操作,
    `Grant.enter` 调用闭包的时候, 把对应的 `Owned<T>` 传入.
    这是 capsule 唯一得到 `Owned<T>` 的方法,
    也就是 capsule 唯一访问 grant 内存的方法.
  - `Grant.enter` 的闭包外, 不会有指向其参数 `Owned<T>` 的引用.
  - 这样保证了, 在闭包内, grant 内存总是有效而且类型安全的.
    并且一旦 process 退出, 可以安全的释放其所有 grant 内存,
    不用担心有指向其 grant 内存的引用.
* 进程内存布局: 进程能使用的内存是在其启动式就确定分配的一块, 大小是确定的.
  底下开始向上生长的是 process code/data, 顶上开始向下生长的是 grant.
  两者如果交汇, 禁止接下来的任何内存分配.


# IMPLEMENTATION
* MPU 操作: 进入某 process 时,
  - 允许访问 flash 上的 process code
  - 允许访问 sram 中的 process data/stack/heap (但无 grant)

* Grants: 参见前文 Grant. Figure 6. 很好用!

# EVALUATION
* capsule 隔离的代价: 相对没有隔离, 单体系统, 隔离加入了通讯代价.
  - 时间: 节能睡眠模式 + 忙询模式.
  - 空间: ram + flash.
  - 应用: helloworld + realistic.

* capsule vs process 隔离: capsule 之间通信代价很低.
  - 空间: 共享的栈; process 的元数据;
  - 通讯: 一个函数/函数指针 vs 上下文切换

* grant 内存:
  - overhead 指的是 `Owned<T>` 和 `Grant<T>` 的元数据的 overhead.

* grant 导致的额外 overhead:
  - e.g. 时钟中断时, 必须遍历所有进程的 Grant

# 其他
> [reddit](https://www.reddit.com/r/embedded/comments/7rrtlo/multiprogramming_a_64kb_computer_safely_and/)
主要是三个方面
* 将 core kernel 和驱动隔离: capsules, 利用 Rust.
* 将 kernel 和应用隔离: process, 利用 MPU.
* 支持并发动态的应用: 新的内存管理, grant.
