# Verifying linearizability: A comparative survey

------------------------------------------------------------------------------
# INTRODUCTION
> Linearizability ensures every (potentially concurrent) execution of the
> implementation object (e.g., Fig. 1) can be explained by a sequential execution
> of a corresponding abstract object (e.g., Fig. 2).
> 
> The sequential ordering is determined by the order of linearization points in
> the concurrent execution.

图 4 不错.

# Verifying linearizability
很多都是通过 data refinement 验证的
* 利用 representation relation, 连接 abstract spec 和 concrete impl 的 observable behaviour.

但是
> Data refinement is a system-wide (i.e., global) property and a monolithic
> proof of data refinement quickly becomes unmanageable. Therefore, several
> methods for decomposing it have been developed.

## Difficulties in verifying linearizability
LP 的分类:
* fixed
* external: LP 在哪儿取决于另外某操作的执行
* future-dependent

## Simulation
分析 concrete 的每步 Op 必须保存 representation relation.
图 5.

canonical construction (CC) 证明对象 (con) 和某抽象对象 (spec) 的 refinement.

要证明对象 linearizable, 对于对象的每个方法 m, 考虑其顺序执行的 trace
* `inv m args,   s1,   s2,   s3,   ...,   sn,   ret retv`

我们要证明它是某抽象 spec 的 simulation, 这个抽象 spec 称为 CC
* `a_inv m args,   a_trans,                     a_ret retv`

即证明, 存在一个仅一个的 sk, 使得 sk 和 a_trans 是 Non-shuttering 的 simluation, 
其他任何 si 都是 shuttering simulation.

图 6 是, 按照图 5 证明 simulation 性质之后, 如何从 concrete trace 得到 canonical trace.
然后 Lynch 证明了 CC spec 是 linearizable 的, 所以 refining 对象也是 linearizable 的.
从 canonical trace 得到 linearized sequential trace 很简单.


另外, 可以直接从 concrte trace 得到 linearized sequential trace, 如图 7.
