# A fork() in the road
* fork 最初在 PDP-7 上 27 行汇编即可实现. 好处是简单.

## Arguments against fork
* 不再简单:
  - 每个系统功能都要在 fork 出现, 如 lock timer aio tracing...
* 不 thread-safe:
  - 即使父进程有多线程, 子进程也只有一个线程
* 不安全:
  - inherit by default, non-default hide
* 慢, 即使使用 COW
* scalability:
  - 不 commute
* (Linux 中) 导致 memory overcommit
* 限制 single address space 应用
* 限制用户态抽象:
  - e.g. DPDK 中不可复制 NIC 状态
* 限制人们的思想:
  - unix way of thinking

在 MS 的 research kernel 里面的确遇到了这些问题.

# alternative
* posix_spawn:
  - 应用 porting 耗费人力
* vfork:
  - 不安全, 强制共享地址空间
* cross-process calls:
  - 例如 syscall 可以修改不仅当前进程, 而是所有子进程的状态
  - future work
* clone
  - 基本和 fork 同样的问题


