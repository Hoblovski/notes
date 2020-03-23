# Patina: A Formalization of the Rust Programming Language
硕士论文

# Problem
## Research Question
对 Rust TS 的整合性描述, 以及 Borrow Checker 正确性的证明.
通过给 Rust 一个形式化语义.

## Existing Solutions, Their Troubles, c.f. Proposed Solution
* Cyclone: 有 region i.e. lifetime 但是没有 affine type i.e. ownership
* 一堆人使用 linear type 等

## My Proposed Answer
NO


# Proposed Solution
## Background

## General Idea
Patina 是 Rust 的一个 small model, 包含其 memory management.
目标: 没有内存泄漏, 并且读内存总是有效的 (i.e. no uninitialized reads).

不变式
* 每个 (heap) 内存位置有唯一的 owner, 决定其什么时候被清空
* 任何 (heap) 内存位置不能同时是 mutable 和 aliased (此词比 shared 更好)

Unique Pointer 是内存的 owner
* 其创建等于内存分配和初始化, 其 drop scope 等于内存释放
* 任何 heap 内存只有一个 unique pointer
  - 防止 double free
* Patina 要求 explicit free, explicit `&*`


## Formalizations: Level 1
这一层包含
* 读的安全性: 不读 uninitialized / out-of-heap (可以写!)
* 编译期和运行时的信息一致

### Type definition
* 用 $\ell$ 表示 lifetime (注意和 $l$ 不同), $X$ 作为类型变元, 修饰符 $q$ 可以是 $imm$ 或者 $mut$

类型定义如
$$
\begin{align*}
  \tau ::=&\quad \text{int} \\
         |&\quad \sim \tau \tag{Unique Pointer}\\
         |&\quad \&\quad~l~q~\tau \tag{Reference}\\
         |&\quad \langle \tau_1\ldots \tau_n \rangle \tag{Variant}\\
         |&\quad [ \tau_1\ldots \tau_n ] \tag{tuple}\\
         |&\quad \ldots \tag{recursive, type var}
\end{align*}
$$

其中 $\sim \tau$ 类似现在 Rust 的 `Box<`$\tau$`>`.

### Concepts

| 概念 | 记号, 元变元 | 性质, 含义 |
| --- | --- | --- |
| Type | $\tau$ |  |
| 程序变元 (PVar) | $x$ | 静态的 |
| Path | $p$ | 静态的. 由一串 deref / index 等组成. 论文中有时也称 $x@p$ 为 path. |
| Typing context | $\Gamma$ | PVar $\to$ Type |
| Lvalue | $x@p$ | 静态的概念 |
| Lvalue typing | $\Gamma\vdash x@p:\tau$ | 原文称为 typing judgement |
| Integer | $z$ | |
| Cell | $c$ | 内存的独立单位, 内容要么空, 整数, 或者 uniqptr (owned), 或者 shrptr (ref). 动态概念 |
| Allocation | $\alpha$ | 对应 PVar, 动态概念. 原子地分配释放的一块内存. |
| Route | $r$ | 对应 path. 包含一串 pay (取得 variant 的 payload), index. 注意没有 deref, deref 使用 READ 实现. 动态概念 |
| Address | $\alpha@r$ 或者 $\alpha$ | 前者是 shrptr, 后者是 uniqptr. 这里认为 Address 等于 ptr. 有时候论文中也把 $\alpha@r$ 称为 route. |
| Layout | $l$ | 包含一些 cell, 加上上面的结构信息 (只有 variant 和 tuple). 读内存读到的是 $l$ |
| Heap | $H$ | Allocation $\to$ Layout |
| READ | $\text{READ}: H, \alpha@r \mapsto l$   | 读取函数 (原文是关系, 但是证明唯一性后就是函数了). 后文中写成 $\text{READ}_H(\alpha@r)$ |
| Variable map | $V$ | $\partial$ PVar $\to$ Allocation |
| Path evaluation | $V; H \vdash x@p \to \alpha@r$ | 在环境 $V, H$ 下 ($V$ 类似 store), 静态的 lvalue $x@p$ 的地址是 $\alpha@r$ |
| ADDR | $\text{ADDR}: V, H, x@p \mapsto \alpha@r$ | 我自己的. 既然可以证明 path evaluation 的 $\alpha@r$ 唯一, 所以写成函数形式好看一点. |
| Heap type | $\Sigma$ | Allocation $\to$ Type |
| Discriminant Context | $\Psi$ | $\partial$ (Allocation $\times$ Route) $\to$ Integer |
| Route typing | $\Sigma, \Psi \vdash \alpha@r : \tau$ | |
| Layout typing | $\Sigma, \Psi \vdash l:\tau$ | |
| shadow heap | | 类似 heap, 但是不保存 cell 的值, 只保存其是否初始化, 以及初始化了的话类型是什么 |
| hole | $h$ | 静态的概念. 对应 cell |
| shadow | $\sigma$ | 静态的概念. 对应 layout. |

后文中同时出现的 $\Gamma, H, V, \Sigma, \Psi$ 都认为是一致的, 即
* Heap well-formedness: $\Sigma, \Psi$ 和 $H$ 相容.
  - $\Sigma$ 中的类型都是良构, closed 的
  - $\Sigma$ 中描述的 Allocation typing 和从 $H$ 读到的 typing 一致
  - $\Psi$ 的确描述了 $H$ 中的 variant layout
  - uniqptr 的确是 unique 的
$$
\frac
  {
    \vdash \Sigma(\alpha) \\
    \vdash \Sigma, \Psi \vdash H(\alpha): \Sigma(\alpha) \\
    \vdash \text{READ}_H(\alpha@r) = \langle \Psi(\alpha, r) : \_\rangle
    \\
    \vdash \text{READ}_H(\alpha_1@r_1) = ~\alpha' \;\land\; \text{READ}_H(\alpha_2@r_2)=~\alpha' \\
    \;\;\Longrightarrow \alpha_1 = \alpha_2 \;\land\; r_1 = r_2
  }{
    \vdash H:\Sigma, \Psi
  }
$$

* Variable map preserves typing
$$
\frac
  {
  \Gamma(x) = \Sigma(V(x))
  }{
  \Gamma, V\vdash \Sigma
  }
$$

### Key Theorems
从一个 lvalue 访问内存有两步:
* 计算这个 lvalue $x@p$ 在运行时的地址 $\alpha@r$
* 真正读取 $\alpha@r$ 这个地址, 使用 $\text{READ}$

* Path type preservation
  - 保证运行时某块内存的 typing 的编译期其 typing 一致
$$
\frac
  {
    \Gamma \vdash x@p : \tau
  }{
    \Sigma, \Psi \vdash \text{ADDR}_{V, H}(x@p): \tau
  }
$$

* Read Safety:
  - **只要**访问是有效的 (e.g. 保证 $\alpha\in H$, 而且不会对 $z$ 做 index 等)
  - **那么**一定读是安全的, 而且读 preserve typing
$$
\frac
{\Sigma, \Psi \vdash \alpha@r:\tau}
{\Sigma, \Psi \vdash \text{READ}_H(\alpha@r):\tau}
$$

## Formalizations: Level 2

### Concepts

| 概念 | 记号, 元变元 | 性质, 含义 |
| --- | --- | --- |
| Hole | $h$ | 静态概念. 类似 cell, 但是把具体的值去掉, 只留下类型和是否初始化 |
| Shadow | $\sigma$ | 静态概念. 类似 layout (其实我觉得也有点像 type), 同上. 可以看成一块 heap 在 init 和 type 上的 shadow. |
| INIT | $\text{INIT}(\tau) = \sigma$ | Type $\to$ Shadow |
| shadow context | $\Upsilon$ | Variable $\to$ Shadow |
| | $\text{SHALLOW}_{\Upsilon}(x@p) = \sigma$ | 计算 lvalue 对应的 shadow. |
| shadow typing | $\vdash \sigma : \tau$ | uninit 有除了 tuple 外任何类型, 其余正常. |
| DROPPABLE | | layout $l$ 是否不 own heap memory |
| | $H\vdash l:\sigma$ | |
| Expression | $e$ | 包含常量, 左值和 (新创建) 的引用 |
| Rvalue | | 可求值到 layout 的 expression |

* shallowly initialized path:
  要把 path 计算成 route 需要经过一系列的 dereference.
  如果在计算 $\text{ADDR}_{V, H}(x@p)$ 的过程中, 被 dereference 的指针一定不 uninit, 那么就称 $x@p$ 是 shallowly initialized path.

后文中认为 $\Upsilon$ 和其他 context 一致, 即
* Shadow consistency: $\Upsilon$ 和 $\Gamma$ 一致
  - PVar 的 shadow 正好有类型是 $\Gamma$ 给出的类型
$$
\frac
  {
    \vdash \Upsilon(x) : \Gamma(x)
  }{
    \vdash \Upsilon : \Gamma
  }
$$

* Shadow heap correspondence: $\Upsilon$ 和 $H$ 一致
  - 任何 heap allocation 都有一个 owner
  - PVar 对应的 layout 的确有 $\Upsilon$ 给出的 shadow
$$
\frac
  {
    \forall \alpha\, .\;\Big( \exists x\, .\; V(x)=\alpha \quad\lor\quad \exists \alpha'@r\, .\;\text{READ}_H(\alpha'@r')=\sim \alpha\Big)\\
    H\vdash H(V(x)) : \Upsilon(x)
  }{
    V\vdash H:\Upsilon
  }
$$

### Key Theorems
* Shadow perservation
$$
  H\vdash \text{READ}_H\Big(\text{ADDR}_{V, H}(x@p)\Big)\;:\; \text{SHALLOW}_{\Upsilon}(x@p)
$$

* Path progress
  - 用 $f(x) = \_$ 表示 $\exists y\, .\; f(x)=y$
$$
\frac
  {
    \Gamma \vdash x@p: \tau\\
    \text{SHALLOW}_{\Upsilon}(x@p) = \_
  }{
    \text{ADDR}_{V, H}(x@p) = \_
  }
$$

## Formalizations: Level 2.5
开始处理 Rvalue

### Concepts
| 概念 | 记号, 元变元 | 性质, 含义 |
| --- | --- | --- |
| Expression | $e$ | 包含常量, (可能导致 move 的) 左值使用 (i.e. coercion from lvalue to rvalue) 和 (新创建) 的引用 |
| Rvalue | | 可求值到 layout 的 expression |
| shadow | $\sigma$ | 除了 hole $h$ 外还包含 bank $\$$ |
| lifetime  | $\ell$ | 诸 lifetime 构成 poset, $\le$ 是被包含关系 i.e. $l \le l'$ (除非必要否则 $\le$ 指 lifetime poset 上的序. 即 lifetime $l$ 被 $l'$ 包含. 原文用 $\mathcal{L}$ 表示 lifetime 上的 $\le$, 我用 $\le_{\ell}$. |
| lifetime context | $L$ | PVar $\to$ Lifetime |
| bank | $\$$ | 包含一串 $(\ell, q)$, 这里 $\ell$ 是 lifetime, 表示一个 heap location 上一系列 stacked loans. 即 bank 包含一系列的 loan. stack 顶是更 recent 的 loan. |
| hole typing | $\le_{\ell} \vdash h: \tau \text{ controlled by } mq \le l$ | |
| shadow typing | $\le_{\ell} \vdash \sigma:\tau \text{ controlled by } mq \le l$ |  |



注意
* 如果需要实现 NLL 那么 Lifetime 只能是偏序集而不能是全序集

同样, 需要满足
* Bank coherence: 对于各个 $\$$ 说的
  - 需要满足: 更 recent 的 $(\ell, q)$ 和更 old 的 $(\ell', q')$ 保证 $\ell \le \ell'$
  - 需要满足: 如果 $\$$ 顶是 imm loan, 那么不能加入 mut loan. 但是相反可以.
  - 相当于从 $\le_{\ell} : \text{Lifetime} \times\text{Lifetime}$ 定义了 $\le_{\$} : \text{Bank}\times\text{Lifetime}$
  - $\$ \le_{\$} \ell$ 表示, 某块 lifetime 为 $\ell$ 的内存 $\sigma$ (即其 owner 的 scope lifetime 是 $\ell$) 允许其上存在 bank $\$$

### Key Conjectures
这节开始就没有 Theorem 了.

## Main Obstacles and Resolution


# Evaluation
## Metric

## c.f. Existing solutions



# Retrospect
## Remaining confusions
* Write safety 呢?
  - 似乎初始化一定是 write unsafe 的.

* 为什么要把 Path 和 Route 区分出来用 $@$?
  - **不是每个 cell 有一个 owner. 而是每个 allocation 有一个 owner, 而且不同
    allocation 的 owner 不同. 一个 allocation 包含很多 cell, 它们是和 allocation
    以某种方式 (e.g. index-by etc) 联系.**

* 对于确定的 $H$, $\Sigma, \Psi$ 是否唯一?
  - 不是唯一的. 最简单的例子就是 void.

* 对于确定的 $V, H$, 是否有 $V, H\vdash x@p \to \alpha@r$ 的 $\alpha$ 和 $r$ 的唯一性?
  - 应该是的. 对 $p$ 归纳.

## Future Work and Weakness of Proposed Solution
e.g. Performance? Labor? Economicalness? Elegance? Compatibility? Usefulness?

## My main take-away


# Others

