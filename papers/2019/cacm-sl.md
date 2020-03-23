# Separation Logic

处理指针很困难, 因为逻辑这个工具基本上还是基于 "语法" 的

> The main difficulty is not one of finding an in-principle adequate
> axiomatization of pointer operations; rather there is a mismatch between
> simple intuitions about the way that pointer operations work and the
> complexity of their axiomatic treatments. ... when there is aliasing, arising
> from several pointers to a given cell, an alteration to a cell may affect the
> values of many syntactically unrelated expressions.

SL 最开始是处理堆的, 现在演化成 general theory for modular reasoning.

> "To understand how a program works, it should be possible for reasoning and
> specification to be confined to the cells that the program actually accesses.
> The value of any other cell will automatically remain unchanged."

并且 SL 还有个好处是可以自动化. e.g. smallfoot 和 verifast.

基于 SL 的实际工作有 FSCQ, certiuos.

> An assertion talks about a heaplet rather than the global heap, and a spec
> {P} C {Q} says that if C is given a heaplet satisfying P then it will never
> try to access heap outside of P (other than cells allocated during execution)
> and it will deliver a heaplet satisfying Q if it terminates.

Hoare 本来也有 disjoint parallel rule, 不过用的是 `/\` 而非 `*`.

assertion 表示程序状态. 状态其实分为 store 和 heap, 不过可以写在一起.
我的理解中, assertion 如下

$$
\begin{align*}
    \diamond ::= & \land | \lor | \to \\
    \mathbf{x} ::= & x | y | z \ldots \\
    \mathbf{v} \in &\mathbb{N}\\
    \mathbf{e} \text{is }& \text{expression}\\
    P ::= & \lnot P \;|\; P \diamond P \;|\; \top \;|\; \bot \\
          & P * P \\
          & P -\!* P \\
          & \mathbf{e} = \mathbf{e} \\
          & \mathbf{e} \mapsto \mathbf{e}\\
\end{align*}
$$
