# A Theory of Type Qualifiers
> Jeffrey S. Foster,  Manuel Fähndrich,  and Alexander Aiken. 1999. A theory of type qualifiers. In Proceedings of the ACM SIGPLAN 1999 conference on Programming language design and implementation (PLDI '99). ACM,  New York,  NY,  USA,  192-203. 

除了简单的静态系统, 语言对于程序员知道的 invariant, 如 nonnull, positive 等支援很少.

贡献
* 说明给语言增加 qualifier 很简单, 给 lex / parse / typecheck 改动很小
* 说明 qualifier 处理可以从 type system 中分离
* 提出 qualifier 多态, qualifier 推断   (dzy: TODO / 没看)

作者认为 qualifier 其实就是加了一种简单的 subtyping,
要么 $q \tau <: \tau$, 要么 $\tau <: q \tau$.
如果 $\tau$ 能被安全地类型转换成 $q \tau$, 那么就 $\tau <: q \tau$.
如 $\tau <: \mathrm{const} \tau$, 而 $\mathrm{nonnull} \tau <: \tau$.

qualifier 多态的例子是 C++ 中函数需要两种版本: const 和 nonconst, 如
$\mathrm{const} \mathrm{int}* \to \mathrm{const} \mathrm{int}*$, 
$\mathrm{int}* \to \mathrm{int}*$ 是不同的类型 (相同就是互为 subtype), 不能缩减成一个.


# 系统记号
* $TVar$: 类型变量集合
* $\Sigma$: 类型构造器集合, 每个元素 $s$ 都接受一定参数, 然后构造出类型 $s(t_1, t_2, t_3\ldots)$.
* $Typ$: 所有类型
* $PVar$: 程序变量的集合
* $A$: 类型环境, 是一个 $PVar \to Typ$ 的映射

然后定义了 positive qualifier 为满足 $\tau <: q \tau$ 的 $q$,
negative 的可以看成缺失的 positive. 这样一系列的 $q$ 就构成了一个 (qualifier) lattice, 如 Figure 2.

lattice 上面的序实际上可以 trivially 定义一个 subtype 关系, 如 $Q \le Q' \Rightarrow Q \tau <: Q' \tau$.

2.2 中
* $l e$ 表示给 $e$ 加上一系列的 qualifier $l$. 称为 qualifier annotation.
  和 C / C++ 经验一致.
* $e|_l$ 表示要求 $e$ 的 top level qualification 必须符合 $l$. 称为 qualifier assertion.
  top level 的含义是, 
  比如 `const nonnull PtrType(nonnegative volatile int)` 的 top level qualifier 就是 `const nonnull`.

## 对比
没有加入 qualifier 时, 类型如
$$ 
Typ ::= \alpha \, |\,  c(t_1, t_2 \ldots) \quad \alpha \in TVar, c \in \Sigma, t_i \in Typ
$$

即类型可能是
* 类型变量
* 构建出来的类型 (具体类型在此列中)

加入 qualifier 时, 类型如
$$
\begin{align*}
QTyp &::= Q \tau\\
\tau &::= \alpha \, |\, c(qt_1, qt_2 \ldots) \quad \alpha \in TVar, c \in \Sigma, qt_i \in QTyp\\
Q &::= \kappa \, |\, l\quad \kappa \in QVar, l \in L (\text{qualifier lattice})
\end{align*}
$$
即 qualifier 不仅仅包含 lattice 上元素, 还包含新的 $\kappa$, 即 *qualifier variable*.

(*dzy: 深入理解, 为什么类型系统需要 polymorphism 和 type/qualifier variable?
  后者是不是因为它用了 constraint based typing)

关键结果在图 4 中.

## const 例子
const 这种比较特殊: 只有 lvalue 的 const 才有意义.
为了加入 const, 需要新的语句 `e1 := e2`,
而且还要引入引用类型 (值是 immutable 的, 引用才是可修改的).
相关规则如 (注意引用类型是 invariant 的, 所以 $\rho_1$ 必须和 $\rho_2$ 等价)
$$
\frac{\vdash Q_1 \le Q_2, \quad \vdash \rho_1 = \rho_2}{\vdash Q_1 \mathrm{ref} \rho_1 <: Q_2 \mathrm{ref} \rho_2}
$$

const 的语义这里简单地就是, 赋值的 lhs 是引用的同时, 还必须不能被 const qualifiy.
* 可以修改源代码来完成,还可以简单地
$$
\frac{A\vdash e_1:\lnot \mathrm{const}\,  \mathrm{ref}\,  \rho, \quad A\vdash e_2: \rho}{A\vdash e_1 := e_2 : \bot \mathrm{unit}}
$$


# 深入
## 类型推断

## 多态, soundness
TODO

## const 推断
把 C 的变量变成 ref, 出现在 rvalue 的变量自动解引用.

只考虑指针和简单类型情况下, 把 C 类型转换为 qualified 类型的方法 $f$ 如

$$
\begin{align*}
  f(q\, \text{int}) &= q\, \text{ref}\; \bot int\\
  f(q\, \text{ptr}(\rho)) &= q\, \text{ref}\; f(\rho)\\
\end{align*}
$$


## 实验
这种纯粹介绍理论, 不为提升某种指标的论文很难写实验评价. 可以学习他的.

在 C 上实验了 const 推断.
* Figure 6: 即使对于尝试多用 const 的项目, 他的 const 推断也能指出很多能用 const 而没用的地方
* 没有特别地优化时间的情况, 推断时间是编译时间的 1~2 倍 (无 qualifier 多态), 或 2~4 倍 (有 qualifier 多态)
* qualifier 多态的 const 推断较无多态的, 能推断出更多能用 const 的地方

只考虑指针的 const, 而且只考虑指向内存的 const. (C 按值传参, 参数如 `const int a` 毫无意义)


# 其他
1. 什么是 binding time 分析?
binding time 分析将程序中的值分为两类,
一种是编译时就可计算的 (类似 `constexpr`), 用 `static` 表示.
一种是运行时才能计算的, 用 `dynamic` 表示.
它用在称为 partial evaluation 的编译优化中.

2. 速成 lattice 理论: lattice 可以认为是满足如下性质的偏序集 $(L, \le)$: 
$$
\forall a, b \in L\;:\; \sup(a, b) \, \text{and}\, \inf(a, b) \, \text{exist in}\, L
$$
例子如幂集 $(2^S, \subset)$ ($S$ 是最大值, $\emptyset$ 是最小值), 自然数 $(\mathbb{N}, \le)$, 因子格 $(\mathbb{Z}^+, |)$.<br>
通常也把 sup 称为 join, 其实就是并集; 把 inf 称为 meet, 其实就是交集. 一些显然的性质如
* $a \le \sup(a, b), \quad \forall b$
* $\inf(a, b) \le a, \quad \forall b$
* 格不一定存在最大值 $\not \exists m\;:\;\forall a\;:\; a \le m$, 也不一定存在最小值.
* 有限格 ($L$ 有限) 一定有最大值和最小值, 称为 top / bottom element, 记为 $\top, \bot$.
* 有限格画图是很好理解的方法, 见 Figure 2.

3. 记号
| 记号                   | 含义                                          |
| ---                    | ---                                           |
| $\alpha$               | 类型变量                                      |
| $c$                    | 类型构造                                      |
| $q$                    | 单个 qualifier                                |
| $L$                    | 所有 qualifier 组合形成的 lattice             |
| $l$                    | qualifier lattice 中某元素, 如 $\bot$, $\top$ |
| $\kappa$               | qualifier 变量                                |
| $\le$ 或 $\sqsubseteq$ | lattice 上的序                                |
| $<:$ 或 $\preceq$      | 子类型关系                                    |
| $\tau$                 | 未 qualified 类型                             |
| $\rho$                 | qualified 类型                                |
| $[x \mapsto t_1]t_2$  | 将 $t_2$ 中的 $x$ 替换成 $t_1$ |

4. 想法
  - 看起来 qualifier 实现就是子类多态 (subset polymorphism), 做得好应该是无运行时开销
    而用 marker trait 是不是也可以?
    像 sorted, 包括 nonnegative 等的 qualifier 可以直接一个 `trait Sorted`
  - 他的系统中, qualifier 和 type 完全是解耦合的. 实际并非如此.
    如 const 只能应用到 ref 上, const int 是没有意义的.

5. 其他
  - *free variables*: 在项/表达式中出现的符号变量,
    并且不是这个项/任何父项的参数.
    相对的是 bound variable, 表示这个符号被 bound 到某个集合 $S$.
    如 $\forall x\;:\;\exists y\;:\;\varphi(x, y, z)$ 中,
    $z$ 是 free variable, 但 $x$, $y$ 是 bound variable.
  - *closed term*: 不包含 free variable 的项.
