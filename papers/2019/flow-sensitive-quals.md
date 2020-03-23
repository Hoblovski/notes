# Flow-Sensitive Type Qualifiers



# 问题背景
*flow sensitive*: 一个值, 在不同的 program point, 类型不同.
flow sensitivity 限制到 qualifier, 其不包含底下的具体类型.

约束求解需要特别改变, 否则代价极大.



# 理论
call-by-value lambda calculus + ref.
* `assert(e, Q)` 将 e 的顶层 qualifier 设置为 `Q`.
* `check(e, Q)` 只有当 e 的顶层 qualifier 是 `Q` 的时候才 typecheck.

类似 Christopher00, 加入 ref 之后需要 abstract store.
$$
\begin{align*}
  e ::= & x | n | \lambda^L x:t\.e | e_1 e_2 | ref^{\rho} e \\
      & | !e | e_1 := e_2 | assert(e, Q) | check(e, Q)\\
  t ::= & \alpha | int | ref \rho | t \to^L t'\\
  L ::= & \psi | \{\rho\} | L_1 \cup L_2 | L_1 \cap L_2\\
\end{align*}
$$
其中 $\rho$ 表示 abstract location.
$L$ 表示 effect, $\psi$ 是 effect variable, 而 $\rho$ 在 effect 中表示读/写/创建 $\rho$.

另外还有
| 符号 | 含义 |
| --- | --- |
| $C_I$ | 将某地址映射到该处值的类型 |
| $\sigma$ | standard, unqualified type |
| $\tau$ | qualified type |
| $Q$ | qualifier |
| $\iota$ | linearity |
| $C$ | store |

TODO



# 验证结果
不考虑 C 中的不安全特性, e.g. type cast, varargs, bad pointer arithmetic.

度量
* 应用到某项目上, 能否发现, 发现多少 bug
* 静态分析的 false positive 问题
* tool 的运行效率

## Linux Locking
`struct spinlock` 可以是 `locked <: unk`, `unlocked <: unk`.
注意 locked 和 unlocked 之间没有关系.

false positive 问题仍然存在, 有动态语义下完全 well locking 的程序不通过 type checking.
使用 restrict 可以减少 false positive. restrict 可以推断, 作为 future work.

## C stream library
glibc 里面 FILE 那块, streamed IO.
检查了 sendmail, man 中 stream 相关的部分.

要求: open 之后检查 NULL, 之后才能读写.

只找到一个 bug, 然后声称主要关心 tool 的效率, 而非在这些经常测试的程序上找出 bug.

和 loc 相对耗时基本是线性的, scalability 尚可. 比 parsing 慢个两三倍.



# 背景
## Linear Type System
通常的逻辑下, $\Gamma, A, A, \Delta \vdash C$ 那么可以推出两个结论:
$\Gamma, A, \Delta \vdash C$,
$\Gamma, A, B, \Delta \vdash C$.
即前提出现的次数不影响结论.

在 *linear logic* 中, 即使前提成立, 这两个结论也不能被推出.
另外还有 *relevance logic*, 前提成立的话只能推出第一个结论.
它们称为 *substructural logic*.

传统地, 资源被使用的次数是无所谓的. 但 linear type 中, 资源只能被使用一次.
因此禁止了 alias.
有些说, 变量 x 在 lambda expression 中只能出现一次.



# Questions
1. qualifier 是否是 context sensitive 的?
不是, 只是说同一值可以在不同程序点有不同 qualifier.

2. Foster 喜欢用 lambda calculus 做理论, 但是实验用 C 做.
  它们怎么处理 C 语言的各种 pecularity?
这次用了 lambda calculus + ref. 但是完全没提从 lambda calculus 到 C.

3. 为什么需要 alias analysis?
flow sensitive qualifier 的存在使得一个值的 qualifier 可能会改变.
如果两个 ref 指向同一个值, 那么改变这个值的 qualifier, 就要求改变这些所有 ref 的 qualifier.
不然就 unsound 了.



# 想法
Foster99 提出 qualifier lattice, 但是实际中可能 qualifier 不是正交的,
如 read, write, readwrite, open. 这种要么抛弃 qualifier lattice,
要么引入 qualifier arithmetic and aliasing, 用 readwrite 作为 read/\write 的别名.
