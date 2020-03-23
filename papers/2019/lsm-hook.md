# Using CQUAL for Static Analysis of Authorization Hook Placement



# 问题
LSM 插 hook 是手动, 容易出错 (位置不对, hook 种类不对).

目标:
* complete mediation i.e. LSM 认证在任何操作前执行.

> verify that an LSM authorization hook is executed on an object of a
> controlled data type before it is used in any controlled operation.

* complete authorization i.e. LSM 认证对于要执行的操作是足够的.

> Each controlled operation requires prior mediation for a set of authorization
> requirements. The verification problem is to ensure that those requirements
> have been satisfied for all paths to that controlled operation.

贡献:
* 用 CQual 验证 LSM hook placement 的正确性



# 方法
* qualifier: `checked <: unchecked`. (Foster99 的方法).
  多步 check 的就是 `checked12 <: checked1, checked2 <: unchecked`.
* 所有 controlled object 初始都是 unchecked,
  通过 authorization function "洗白" 成 checked.
* 自动化标注过程

保证的性质:
* 前提: 所有 controlled type 都是 struct.
* 保证如果读写某 controlled type 的实例的域, 则实例须为 checked.

标注的位置:
* 函数参数, 如果参与 controlled op, 则标注为 checked.



# 结果评价
标注 Linux 然后总结 524 个 type error, 将 error 分类.

使用 type checking 的方法, 但是还有 false positive.

不足:
> Unfortunately, CQUAL cannot yet reason that a field extracted from a checked
> structure is also checked.

讨论:
* 易用? 运行时间
* inheritance: 有些 struct field 可能需要继承



# BACKGROUND
## MAC
Mandatory Access Control.

Linux 默认是 DAC, 用户认证. 最大的问题是 root (以及 setuid 程序),
一旦 compromise 整个系统就 compromise 了. 并且用户可以改文件权限等.

MAC: *operation* by *subject* on *object* 被 policy 检查.
只有 security policy admin 才能改 policy, 用户不能改.

## LSM
Linux Security Modules.

> LSM inserts "hooks" (upcalls to the module) at every point in the kernel
> where a user-level system call is about to result in access to an important
> internal kernel object such as inodes and task control blocks.
>
> -- Wikipedia

目的是给 Linux 实现 MAC, 并且不要大改内核.




# QUESTIONS
* 它做的是 variable access control, 但是 scope 只限定在函数参数的检查.

* 它的 rule: 就是 context insensitive 的, 和传统完全一样的 type checking + qualifiers.
只考虑 controlled type 的实例.
初始化后是 unchecked, 直到 authorization 函数洗白成 checked.
函数参数如果是 controlled type, 那么根据有没有在函数中参与 controlled operation, 参数标注为 checked / unchecked.
之后 type checking 主要检查传参.

* 基于什么原则标注 checked / unchecked?
标注只需要标注参数. 其他实例的 qualifier 是自动生成的.
根据参数有无参与 controlled operation.
