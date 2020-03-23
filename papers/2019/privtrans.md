# Privtrans: Automatically Partitioning Programs for Privilege Separation
------------------------------------------------------------------------------
USENIX Security'04.



# INTRO
*privileged programs* 用 root 跑, 包括 setuid. 程序很多, 易受提权攻击.
使用 *privilege seperation* 保证安全, 把程序分成 monitor 和 slave.
前者被信任, 有完全特权. 两者不同进程, 靠进程隔离.

> The monitor exports only a limited interface to the slave. As a result, a
> compromised slave can execute only a limited number of privileged operations.
> Without privilege separation, a compromised slave may be able to run
> arbitrary instructions with the elevated privileges.



# MODEL
所有 priv 的资源, 包括相关的数据, 以及资源的值, 都只能被 monitor 访问.
`P ref int` 解引用是 `P int`, 只有 `P ref int` 的值才能是 `P int`.
否则需要用 hashing 来解决, `ref int` 要和 `P int` 产生关系, 需要前者是后者的哈希.

一个限制是, 要求访问 priv 资源是通过某一个函数访问. 就是说, 如果 `a` 是 priv
资源, 那么不允许 `a = 5` 以及 `int b = a`, 只能 `f(a)` 或者 `a = f(c)`.

包含三个部分
* 划分资源的方法, 以及指定资源访问 policy 的方法
* monitor 和 slave 两个不同进程间通信, 需要的 RPC 机制
* monitor 中保存 slave 状态的机制

划分资源, 使用 priv / unpriv 两个 qualifier, 基于权限和 policy 标注.
priv 只能读写调用 priv, 而 unpriv 只能读写调用 unpriv.

policy 指定了 slave 能向 monitor 发送什么样的请求们,
其实就作为 monitor 逻辑写到 C 代码里.

downgrading 让 priv 数据变得不再 priv, 然后就可以流动到 slave 中.
upgrading 类似 untainting, 而 downgrading 作者称为 cleansing.

channel 它是通过 RPC 实现, 自然地形成了 request-response 双向 channel.
需要 channel 只有 call-site (c.f. 所有 priv 资源都只能通过函数访问).
* callee 是 priv
* 有参数是 priv
* 函数返回值赋给 priv
这种情况下, 称为这个 call-site 是 priv 的.
`a=f(x)` 变成了类似 `privwrap(&a, f, 'x)` (x 若是 priv 则需要 pass-by-name)

也做了多态, 函数 (而非 call-site) 是 polymorphic 的. 但是其实还是 monomorphic 的,
如果 call-site 是 priv 的那么是 `privwrap(&a, f, 'x)`, 不然还是 `a=f(x)`.
就是 `f` 在 monitor 和 slave 中都有.

另外还有动态检查, 因为类似指针可能指向 priv/unpriv 的.
因此一个 call-site 运行时可能动态地被确定是 priv 还是 unpriv 的.
静是一个态分析保守所以认为是 priv, 但是动态检测可以减少 RPC 优化效率.

# EVALUATION
别人只要不到 10 个 user annotation...


# QUESTIONS
1. 它使用什么方法完成 isolation?
将程序划分为两部分之后, 两部分跑在不同进程之间.
隔离使用进程, 而通讯使用 RPC.

2. type checking / type inference 的 rule 是什么?
2.3 节: 标注为 priv 的函数/变量只能被 monitor 访问.
其他都放到 slave 中.
那么 rule 就是, priv 的只能读写 priv 的, unpriv 的只能读写 unpriv 的.

我可以借鉴, 也是只用标注 crit 和 both.

3. 基于什么原则, 将数据/函数标注为 priv?
* **被迫**: 如果访问需要权限, 那么肯定要标注位 priv.
* **基于 policy**: 如密钥资源相关的数据/函数都标注为 priv.

4. 所谓使用动态信息优化是什么意思?
call-site 可能是 priv 和 unpriv 的, 某些 call-site 的 privi 只能在运行时确定.
静态分析标注它们为 priv 的, 但是太保守了, priv-call 的代价是可以被优化的.

