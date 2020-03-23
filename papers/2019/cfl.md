# CFL-Reachability and Context-sensitive Integrity Types



# 问题背景
`neg <: poly <: pos`
禁止信息从 pos 流到 neg.
e.g. `mutable <: const`, `untainted <: tainted`.

如上 information flow type 限制信息流动.
integrity type 是一种 IFT, 用来限制秘密泄漏.
(其实 misnomer, 应为 confidentiality)


# 方法
## CFL
用 CFG 定义. 建图, 在图上跑 reachability.
没细看.

## Type
两个 qualifier: `safe <: tainted`.

另有 `poly` 表示 context sensitivity.
其具体值依赖于 context, 可能是 safe 可能是 tainted.
使用 viewpoint adaption: "from the viewpoint $q$, adapts $q'$ to $q''$".
当然 viewpoint 就是 context, 如 `obj.fie` 中 `obj` 就是 context.
写作 $q\triangleright q' = q''$, 规则如
$$
\begin{align*}
  \_ \triangleright tainted &= tainted\\
  \_ \triangleright safe &= safe\\
  q \triangleright poly &= q\\
\end{align*}
$$

adapt 的时机: 域访问 或 方法调用.

adapt 的对象:
* 被访问的 field
* 被调用方法的参数和返回值

参见问题 2. 的回答.

他写 type system 的时候给出了 rule, 并且一条一条 rule 的解释.


# 验证结果
就写了半栏加一张图, 举了一个例子而已?



# 背景
## CFL reachability
Context Free Language reachability.

把信息泄漏作为图上的可达问题. 包括
* {call,structure}-transmitted dependencies



# Questions
1. integrity type 和 qualified type 有何关系?
给 type 加上 qualifier, 变成 qualified type, 是实现 integrity type 的最简单的方法.

2. 他说的 *context-sensitive* 和我想的是否是一样的?
似乎和我的是有点像.

它的 context sensitive 指的是, `obj.field` 的 qualifier 不仅仅和 `field` 有关,
还和 `obj` 有关.  同一个 field / method, 可能在 safe object 中就是 safe 的,
而在 tainted object 中就是 tainted.

也就是说它的 context 是 receiver, 但是我们的是 calling function.

3. 它的 typing rule 是什么?
除了标准 rule 以外, 为了支持 viewpoint adaption, 还针对
T-ASSIGN, T-ASSIGN_TO_FIELD, T-ASSIGN_FIELD_TO, T-CALL_METHOD
特别加了 rule.

