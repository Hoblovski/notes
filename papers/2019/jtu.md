# 面向 Linux 设备驱动的内核服务需求特征研究
> [1]茅俊杰,陈渝.Linux设备驱动的内核服务需求特征[J].清华大学学报(自然科学版),2015,55(08):911-915.
------------------------------------------------------------------------------
# 本文中内核服务
* [ibm](https://www.ibm.com/support/knowledgecenter/ssw_aix_61/com.ibm.aix.kernextc/kern_svcs.htm)

> Kernel services are routines that provide the runtime kernel environment to programs executing in kernel mode. Kernel extensions call kernel services, which resemble library routines. In contrast, application programs call library routines.

> Callers of kernel services execute in kernel mode. They therefore share with the kernel the responsibility for ensuring that system integrity is not compromised.

* [freebsd](https://www.freebsd.org/doc/en/books/design-44bsd/overview-kernel-service.html)

Syscall 也是内核服务.

* 一般来说, 就是内核本身给内核模块暴露的借口, 包括各种 struct 定义, 系统库函数等等.

------------------------------------------------------------------------------
# 目的
* 在新兴操作系统中复用 Linux 的设备驱动
* -> 内核中实现 Linux 设备驱动需要的内核功能 (内核服务), 形成 Device Driver Environment 层
* -> DDE 的手工实现难度太大, 需要辅助设计工具
* -> 需要深入理解 Linux 内核驱动对内核服务的需求特征

------------------------------------------------------------------------------
# 驱动对内核服务的需求特征
## 内核服务分类
* **通用**: 同步互斥, 内存分配, 调度, 延迟...
* **设备子系统**: ...

## 考虑的服务
* 锁
* 等待队列
* 内存管理
* 系统时间
* 延时操作
* 进程调度
* 中断管理、DMA 操作与总线访问
* 设备子系统

## 特征
* 设备驱动仍然倾向于使用 Linux 的通用功能. 对于锁, 就是 spinlock, mutex 等, 而非特别为某设备设计的
* 如果分类标准是 "使用了那个设备子系统", 同类设备驱动对内核服务的需求具有相似性.

------------------------------------------------------------------------------
# 内核服务接口的更新规律
最终目的是, 能够自动 patch, 自动升级.

## 接口函数的变动
* 新增 / 删除接口函数
* 参数 / 返回值类型变化
* 函数重命名
* 使用函数封装对数据结构的访问 (e.g. `struct foo_t a` 变成 `get_a() -> struct foo_t`)
* 改变函数语义

特点是, 绝大部分仅仅是函数原型的改变, 满足一个固定的模式, 不涉及接口语义.

## 结论
能够抽取这些变化的模式, 形成所谓的 semantic patch, 然后用于 out-of-tree 的模块更新, 尤其是外部驱动.
