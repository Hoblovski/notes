# Program Synthesis

# INTRODUCTION
* program synthesis: 自动生成某语言的符合用户意图的程序.

# APPLICATIONS
* 数据清洗/转变 (data wrangling)

* 图形学: 生成图形

* 代码修复 (code repair):
  给定有 bug 的程序 $P$ 和 spec $\phi$, 要求计算出一套修改 $M$ 使得新的程序 $P'$ 满足 $\phi$.
  - SemFix, AutoProf

* 代码建议

* 建模

* Superoptimization: 如纯函数优化.

* 并发程序

# CONSTRAINT SOLVING
基本思想
* 将程序语法加上 spec 编码到公式中, 寻找公式模型.

## Component Based Synthesis (Jha et al.)
* 目标: 生成 loop-free 程序.
  - 使用一系列基本 component. component 的语义 $\phi(\vec{I}, O)$ 给定.
  - 最终函数就是 component 的组合
  - 当然, 程序的 spec 也需要给出 $\phi(\vec{I}, P(\vec{I}))$.

* oracle: 检查生成的程序是否符合 spec. 很难写.
  (最差就是人工检视, 但是就不能自动化了)
  改成 I/O oracle: 给定任一输入, 给出有效输出.
  oracle 即是, 任意输入, I/O oracle 和生成程序输出相同.

* 算法大意
```
CBSynthesis( I/O oracle I, validation oracle V, components L )
  loop
    io constraints = constraining the satisability of <the set of examples> within power of <L>
    if some candicate program = get model of <io constraints>
      distinguish constraint = constraining the satisfiability of
          <the set of examples> but the program should be different from <candidate program>
      if some distinguishing input = get model of <distinguishing constraints>
        add <distinguish input output pair> to <the set of examples>
      else if <candicate program> is valid querying <V>
        report <candidate program> as answer
      else
        report: L is insufficient to satisfy V
    else
      report: L is insufficient to satisfy V
```

* 假定每个 component 函数只使用一次 -- 总可以包含同一函数的副本
  - 于是构成一个 DAG, 节点是 component 函数
  - 具体叙述中的 line number 就是 DAG 线性化之后的编号.
  - line number 相同那么值相同, 如 $LI_{12}=LO_2 \to I_{12}=O_{2}$

| 符号                 | 类型           | 含义                                            |
| ---                  | ---            | ---                                             |
| $I_{ij}$             | $\mathbb{B}^5$ | $f_i$ 的第 $j$ 个参数的值                       |
| $O_i$                | $\mathbb{B}^5$ | $f_i$ 的输出的值. 要求有 $o_i = f_i(\vec{I_i})$ |
| $I_{ij}^L$           | $\mathbb{N}$   | $f_i$ 的第 $j$ 个参数在什么地方被定义           |
| $O_i^L$              | $\mathbb{N}$   | $f_i$ 的输出在什么地方被定义                    |
| $\hat{I}_n, \hat{O}$ | $\mathbb{B}^5$ | 程序输入和输出                                  |
| $\hat{I}_n^L$        | $\mathbb{N}$   | 输入总是在最开始的若干行被定义                  |

约束分为
* Well foundedness: 设行数为 $N$ i.e. 一共定义 $N$ 个值; 输入有 $M$ 个 i.e. $|\hat{I}| = M$.
  - $0 \le I_{ij}^L < N$
  - $M \le O_i^L < N$
  - $I_{ij}^L < O_i^L$: 无环
  - $O_i = f_i(\mathbf{I_{i}})$, 其中 $f_i$ 的定义确定了.
  - $\hat{I}_i^L = i, \quad 0 \le i < M$
  - $\hat{O}^L = N-1$

* Data flow constraints: 把前面的输出和后面的输出对应
  - $I_{ij}^L = O_k^L \to I_{ij} = O^k$: 若 $f_i$ 的输入 $j$ 在某行定义, 此行定义的是 $f_k$ 的输出, 则两者相等.

## Solver Aided Program Synthesis
基本思想:
* 不是使用最 general 的 SMT, 而是使用 program synthesis 特别的 SMT.
* 例如, 在 C 中加入 symbolic hole 以及 generator

实现: Rosette (Racket)
* `define-symbolic`
* 加入符号运算 symbolic evaluation

## Inductive Logic Programming
* 通过一系列 example 和 background knowledge 推导出一阶规则和约束



# Programming by Examples
