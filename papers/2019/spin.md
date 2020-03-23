# Extensibility,  Safety and Performance in the SPIN Operating System
------------------------------------------------------------------------------

# 问题和背景
## 动机和目的
* 不同的应用有不同的需求.
  通用系统可能各种应用都能跑, 但效率都很差,
  故需要 dynamic specialization 来类似做一个 system tuning 的工作,
  使得系统特别适合某一种应用.

* (dzy) 又如专注运行数据库 (让数据库自己处理磁盘调度, 改进分页)
  或网路服务器 (sendfile, poll) 的系统.
  又比如像 tlp, 自称安装了这个 package 就让系统变得省电.

* 传统通用系统一定程度上可以 dynamic specialize,
  但是非常麻烦 (可能他们认为 `EXPORT_SYMBOL` 暴露的接口不好用吧)
  易错 (比如垃圾驱动搞崩整个系统) 而且容易顾此失彼.
  (dzy: 并且效果也不好, 自称省电的 tlp 给我的系统 dynamic specialize 之后续航毫无提升)

* 希望是, SPIN 自身可以是一个通用系统,
  但可以很好的 dynamic specialize.
  这里是通过给 kernel 加上 extension 的方式.
  (dzy: 可以理解为类似 kernel module 的, 具体后述)

## 关键问题
* 可拓展性: 以完成 dynamic specialization
* 安全: 保证 specialization 这个过程不会损坏系统
* 效率: specialization 若开销很大就可能抵消其好处


# 工作介绍
## 基础技术
为了实现可拓展性, SPIN 的方法是 内核定义一系列接口,
然后 extension 实现这些接口.
(dzy: 接口就类似回调函数, 这样内核就可以主动调用 extension 中代码了)

因此, 接口需要细粒度, 而且需要权限管理.
(dzy: 防止 faulty extension 崩掉整个系统)

* Extension 动态链接入 kernel. 目的是低代价.
  也就是说 kernel 内部有一个 linker.

* 要求 extension 使用 Modula 3 编写.
  通过编译器来完成严格的模块化, i.e. 模块边界的保证.
  (当然用户应用可以用任何语言)

* Extension 的保护隔离通过 logical protection domain,
  实际上就是一个命名空间, 包含接口 (代码和数据).
  跨域通信的代价降到一个函数调用.

* 发生 "事件" 的时候, 才会调用 extension.
  extension 可以监听事件. 事件在内核提供的接口中定义.
  动态分派的代价也是一个函数调用.

## Modula 3
SPIN 依赖 Modula 3 的安全和模块封装措施
* 类型安全: 这一块内存保存的数据的类型是 A, 访问就必须按照 A 来访问.
  (dzy: 似乎这也包含了 按名访存 的思想? 的确 按址访存 很不安全...)
* 内存安全: 自动内存管理 (dzy: GC 完成)
* 模块和接口


# 架构
第 3, 4, 5 节讲的就是架构.

SPIN 内核有两个组成部分
* core services: 可以认为是控制硬件资源分配复用的服务
* extension services: 应用提供的, 为了 dynamic specialize 到自己的需求.
它们在同样的内核地址空间运行.

连接这两部分的软件 infrastructure 包含
* Protection model: 高效细粒度的访问控制
* Extension model: 如何让 extension 能够集成到内核中

## Protection Model
* **Capability**:
  内核资源, 如设备 / 内存区域, 用 capability 表示.
    (dzy: 也可以称为 handle / access token / ticket)
  Capability 就是一个引用 / 指针, 非常轻量级, 指向内核资源.
  但是类型安全的编译器保证引用的用法是安全的,
  也不能有人随便就造出一个 capability 出来 (不可伪造).
    (dzy: e.g. through some insane pointer arithmetic)
  这个概念似乎来自 mach, [参见](https://www.gnu.org/software/hurd/capability.html).
	一个 capability 包含了两重意思: 用来标识对象, 以及访问控制.
	如果将标识对象和访问控制分开 (如文件名和权限), 就会有问题.

* **Protection Domain**:
  针对一个执行流, 包含一系列可用的名.
  只有有权访问该域的代码才能访问这些名.
    (dzy: c.f. 传统按址访存的系统的 protection domain 就是地址空间)
  不同 domain 可以互相交叉, 完成共享.
    (dzy: c.f. 传统系统的 shared memory)
  编译器检查.
	参见 Figure 1, 它很好的说明了 Protection Domain 和 Capability.

* Domain 由目标文件 object file 表示,
  安全的目标文件应当被 Modula 3 编译器签名.
  目标文件导出的符号是该 domain 导出的, 导入的默认不被解析.
  有相关方法 `Create(objFile): T`,  `Resolve(src, mut tgt: T)` 等.

## Extension Model
有两个关键组成部分
* Event: 系统的状态发生变化, 或者系统需要某项服务.
如 IP 模块定义了事件 `IP.PacketArrived(pkt: IP.Packet)`, 
* Handler: 接受处理 event 的函数/过程

实现中, event 是接口函数 (原型), handler 实现对应函数 (实现), 
这样, 无论是表示系统状态变化的事件, 还是系统内部请求一个服务, 
都可以直接调用这个接口函数 (event), 然后分派到 handler 执行.

(dzy: 这样命名让人觉得是强行面向事件, 简单的 interface / service function 就可以了啊)

权限控制就延续 protection model 的方法
* 要有权限处理某 event, 一个 extension 必须有权限实现它所在的接口
* 要有权限引起 (raise) 某 event, 一个 extension 必须有权限调用其所在的接口

这种架构当然还需要 dispatcher, registeration 等等概念.
一个 event 能有多个 handler, 默认有一个主 handler, 在其 default impl 模块中.
其他 extension 可以请求替换它或者注册额外的 handler.
主 handler 负责批准或拒绝这些请求,
以及给这些请求加上约束如这个额外 handler 是异步的,
需要在一定时间内执行完成等等.

## Core Services
管理处理器和内存资源, 向 extensions 暴露细粒度的**接口** (i.e. event).

管理内存包含地址分配和映射.
有三个底层接口分别是实页框分配, 虚页分配, 以及虚实映射控制.
这些接口由内核实现 (dzy: 显然为了安全),
用户可以提供 service (不是 extension 而是 core) 在其之上构建高层语义, 
如 Unix 地址空间, Mach task 等.

管理处理器就是进程管理. 由简单的 Strand 接口完成, 
用户应用可以提供自己的线程模型/线程实现 
(如 C-Thread, Modula-3 Thread, goroutine)
和自己的 scheduler.
当然为了安全, 应用的 scheduler 只管用户态的线程,
进到内核态之后切换还是内核来做.

Core Services 应当是可信的, 它们需要符合 Spec.
如线程管理中 CheckPoint 之后不切换是违反语义的.
不过即使它们出错, 也只有使用它们的应用会受影响.


# 效果评价
略


# 使用 Modula-3 和 C 的比较
* 曾经尝试构建使用安全的 C 子集, 后来发现好用的 Modula-3
* C 程序员可以速成 Modula-3
* 觉得很慢后来发现是偏见
* SPIN 中用 Modula-3 写的部分比 C 写的部分可靠多了容易维护多了...


# 和 RustOS 的比较
## SPIN 速成
为了方便比较, 简单地总结 SPIN 的重点设计:

* 内核有很多 interface. Extension 可以实现这些 interface, 内核调用这些 interface 然后就可以 dynamic specialize.

* 要防止 extension 破坏 kernel, 就要控制 extension 对于 kernel 里面的东西的访问. 我的理解就是, extension 不能随便改 kernel 的数据, 不能随便调用 kernel 的函数. 当然, 合理的改数据和调函数还要允许.

* 不能乱调函数: 通过 protection domain 完成. 
(dzy: 但是这样也只能控制 extension 能调*哪些*函数, 不能保证 extension *以正确的 contract* 调用这些函数? 不过要要求后者的话感觉会很复杂, 说不定就变成了对着 spec 做形式化验证了...)
不能乱改数据: 导出的 capability 类似一个 opaque handle, 必须通过导出的方法才能访问. 也就是只能通过正确的方法改数据.

* 因为 extension 动态链接进 kernel, 在同一个地址空间内所以开销很小.

## 比较
首先 RustOS 和 SPIN 目的是不同的, RustOS 完全没有考虑 dynamic specialize. 所以总体来讲我觉得不同大于共同.

SPIN 的保护我觉得还是保护 core kernel,
或者说 kernel 的其他部分不被有错的 extension 影响.
(Extension 有错就只会搞掉自己)
而且 Extension 不是完全被信任的,
(装 extension handler 的时候主 handler 都要检查)
所以 SPIN 的安全有点 security 的味道.

相反现在的 RustOS 还是一个整体, 没有 kmod 或者 extension.
RustOS 的每个部分都是完全被信任的 
(使用 Rust 是为了防止无意的错误?) 
(显然如果 RustOS 内部要是有恶意的话,
	随便用几个 unsafe 系统就 compromise 了)
因此 RustOS 更多地是 safety 的味道.

以上是我觉得它们最大的不同点.
而它们最大的相同点,
在我看来就是靠 "按名访问" 而非通过裸指针访问来提高安全性和隔离.  

就是说调一个函数 (访问变量同理), 不能像 C 一样 `((void(*)()) X)();`,
其中 `X` 是任何一个函数的地址.
要检查 `X` 是不是指向这个上下文里面不允许调用的函数,
这件事情基本不可能. 这就是我理解的 "按址访问".

而 "按名访问" 就是, 要想调某个函数, 就要明明白白地把函数名写出来.
这样检查当前上下文里面这个函数允不允许被调用就简单多了.

上面是说的安全. 至于效率, 我觉得没有太多可比性.
SPIN 的效率是说通过链接,
把内核不同部分放到同一个地址空间里跑来减少跨域通信的开销.
而 RustOS 就只是想说高级语言特性比如 ownership, 闭包等是零代价抽象.
另外 RustOS 没有 GC, 而 SPIN 的 Modula-3 有 GC. 这可能也造成部分效率差别.


## 不足
对我们的启发

* 没有改硬件
* 如 await async 在内核中的应用
