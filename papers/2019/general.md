# 不同的锁的区别

考虑 spinlock, mutex, 以及一般的 sem.

## 参考
* [proc4](https://people.mpi-sws.org/~druschel/courses/os/lectures/proc4.pdf)

## spinlock
和 mutex 一样用来控制对临界区的访问.
临界区可以看成一种共享资源, 共享的是代码, 任何时刻只能有一个线程访问这个资源.

线程试图进入锁上一个 spinlock 时, 要是 spinlock 已经被锁上,
那么线程在这里循环, 一直尝试锁上 spinlock.
直到时间片耗尽才被换出, 解锁不会唤醒线程.

属于忙等策略, 常用于对小的临界区的保护.

在单核机器上使用 spinlock 没有意义,
因为除非等待 spinlock 的线程被换出,
否则永远不会有其他线程来解锁这个 spinlock,
使得等待 spinlock 的线程继续进行.

通常使用特殊指令实现, 如 `lock cmpxchg` (x86), `ll; sc` (mips).

## mutex
和 spinlock 一样用来控制对临界区的访问.

mutex 可以用来实现 sem.

线程试图进入锁上一个 mutex 时, 要是 mutex 已经被锁上,
那么线程被换出, 当 mutex 被解锁时才被唤醒.

需要线程切换, 所以用于较大的临界区的保护.

spinlock 和 mutex 功能是一样的, 区别只在于加锁失败时,
是忙等还是被换出.
现代操作系统中通常有混合方式, 即先 spin 一段时间, 要是这段时间内线程没有得到锁,
那么被换出.

## semaphore
sem 用来控制对共享资源的访问.

sem 可以用来实现 mutex, 但是不能用来实现忙等的 spinlock.

线程试图 down 一个 sem 时, 要是 sem 已经是 0,
那么线程被换出, 当另外线程 up 这个 sem 时被唤醒.

单核系统中, 在 up 和 down 中关中断;
多核系统必须硬件支持.

