# 问题
使用 Go 一样垃圾回收的高级语言 (HLL) 来写 kernel 有什么好处.


# INTRODUCTION & RELATED WORK
...


# MOTIVATION
* 用 C:
	- 操作底层硬件;
	- 编译出来的汇编比较确定.
* 用 HLL:
	- 自动内存管理: 减少内存 bug
	- 类型安全
	- 运行时分派
	- 语言层级支持并行
	- GC 有代价


# DESIGN OVERVIEW
* 基本架构
	biscuit 在 runtime 上, runtime 调用一个底层 kernel 来完成 内存分配/线程调度
	没有底层 kernel 的时候, 用一个 shin layer 来实现.
* 进程
	用户进程和 kernel goroutine: 一对一
	biscuit 使用的是 go 的 runtime goroutine scheduler,
	没有自己的调度器
* 中断处理
	只把处理中断的 goroutine 标记为 RUNNABLE
	防止死锁?
* 文件系统和网络
	POSIX


# GC & KERNEL HEAP EXHAUSTION
go GC:
* 不做 compacting, 有 stop-the-world pause.
* 还是简单的 mark-and-sweep.

kernel heap exhaustion 的处理:
* 预留固定大小 M 的内存给内核堆.
* 内核堆耗尽是一个关键问题, biscuit 没有实现类似 vector 的增长.

syscall 的处理:
* 静态分析得到, 每个系统调用它执行过程中最多可能使用多少内存 
  (一些难以分析的情况是给保守估计)
* 系统调用等到能保证有那么多空内存的时候再开始
* 具体静态分析计数在 6.3, 没看.

* 内核中一个 killer goroutine 检查每个进程,
  内核堆快耗尽的时候将使用最多内核堆的进程杀死.
  快耗尽: 有系统调用在等待内核堆空闲了.


# EVALUATION
* 高级语言的特性 (defer, chan, goroutine) 对于 OS 设计有什么好处
	设计中能使用. 而且还用了 packaging.

* 能到什么程度减少 bug
	200 多个 CVE 里面, 很多是逻辑 bug, go 没法解决.
	有 31 个可以用 go 解决 (编译错误或者 panic 代替错误地继续),
	主要是内存错误.

* 效率表现
	GC 有代价但是不大. runtime 函数 prologue 代价最大.


# CONCLUSION
...

------------------------------------------------------------------------------
# Extracting parallelism in OS kernels using type-safe languages
