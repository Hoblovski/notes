# Leveraging Rust Types for Modular Specification and Verification

------------------------------------------------------------------------------
# Problem
## Research Question
* 做 Deductive Verification
  - 和所有 verification 差不多, 针对 imperivative language
  - 针对有指针的 systems language
  - 指针导致 aliasing, possibly uncontrolled side effect

## Existing Solutions, Their Troubles, c.f. Proposed Solution
* 有指针, 成熟的方法是 separation logic
  - 难学难用

* 有限方法, 比如 model checking 或者 symbolic execution
  - 证明能力有限

## My Proposed Answer
* 缺乏人力, 懒得学习就使用有限方法.
* 希望足够的 expressiveness 还是绕不开 separation logic.
  - 顶多加糖.


------------------------------------------------------------------------------
# Proposed Solution
## Background
Viper:
* heap-based imperative language
* 包含 assertion, pre/postcondition, loop invariant
* 堆中所有对象类型都是 `Ref`, 其用包含一个字段表示对象的值
  - 但是值的类型不能确定
  - `Ref` 表示一个 permission i.e. exclusive capability
* permission 能够被显式的增加 / 删除
* `acc(o.f)` 是一种 resource assertion, 表示当前, 对象 `o` 的字段 `f` 有 permission 访问
* `&&` 类似 SL 的 `*` 而非 $\land$
* `fold(pred)` 和 `unfold(pred)` 完成交换一个 predicate 和它的 body
* `exhale(pred)` 要求 `pred` 成立, 之后释放 `pred` 包含的所有资源
  `inhale(pred)` 正好相反. 释放之后, 其他人就可能修改那个资源了.

## General Idea
基本思想:
* 从抛去所有 assertion / pre/postcondition 的 Rust 程序, 生成 core proof.
  - core proof 只保证 memory safety, type safety 等 Rust 本来就有的性质
  这一步是完全自动的. Core proof 中包含的信息有 framing, 副作用等.
  core proof 用 Viper 框架语言写作.
* 将用户提供的 assertion 稍微变换一下加入 (interweave) 到 core proof 中
* 最后的 proof 被 Viper 框架验证

构建 core proof 的过程又分为 3 步解释: 似乎引用需要特别处理.

### Core Proof Construction without References
* owner: 是一个变元 $x$
* place: 类似 Patina 中的左值 $p ::= x \quad|\quad p.f \quad|\quad *p$
* capability: 能访问 place 中值的 "能力", 实体化为 reference

PCS
* place capability set: 某 program point 能访问的所有 place
  - 如对于 `let a = Point ...`, 如果 move 了 `a.x`, 那就不能访问 `a`, 即 unfold
* 函数入口处 PCS 只包含参数, 离开的时候除了 drop 的只能由返回值中的 place
* PCS operation 包含
  - remove: move assign 或者 fall out-of scope
  - add: 新的 owner
  - pack, unpack: 字段分解 (因为 Rust 的 ownership 是传递的)
* 利用 Rust 编译器, 自动推断每个 statement 寸前的 PCS
  - 推断的其实是 PCS operation

*dzy: PCS 事实上似乎就是 SL 中 "当前的堆"*

Core proof 实际上包含一系列的 Viper method, 每个对应原来的一个 Rust 函数.
将 Rust 函数映射到 Viper method, 包含
* memory: 内存用 Viper 的 Ref 建模
* types: 变成 resource assertion 的 predicate
  - 从这个出发, 一系列的字段可以访问而且递归满足 resource assertion
* capability: 直接把 PCS 翻译成一系列 disjoint conjucted resource assertion
  - 需要 Prusti 插入 `fold()` 和 `unfold()`
* capability transfer

构建完 core proof 之后, 直接把 user-provided assertion 丢进里面,
让 Viper 处理细节.

重点可以参考阅读 App. A. Fig. 7.

### References
TODO

mutable borrow  和 exclusive capability 类似.

borrow 的性质是可以从一个 borrow 创建 less-scope less-reachibility 的 sub-borrow.
当然, sub-borrow 的 lifetime 不能超过原 borrow 的 lifetime -->
而且, 因为是 mutable i.e. exclusive 所以只要有 sub-borrow 未 expire, 原 borrow 必须不可访问.

所以创建 sub-borrow 就是
删除原 borrow 的 capability, 增加 sub-borrow 的 capability
然后 sub-borrow 过期就是
restore 原 borrow 的 capability

RefMut 的类型编码类似 Box

删除和增加那个就是
RefMutRoute(r)
变成
RefMutPoint(res) && (RefMutPoint(res) -- * RefMutRoute(r))



至于 shared reference:

把 PCS 分成两类: shared  (upgrade->) (<-downgrade) exclusive  // >

之后利用有 fractional permission 的 SL 变种:
capability 1 可写
capability 0<x<1 可读 //>>
capability 0 不可访问

这里用 symbolic read permission 实现.

从 exclusive capability 生成 SRP 使用 exhale.


至于 pledge... TODO


### 实现, 验证
实现为  Prusti, 是 rustc plugin, 在 type checking 之后一个 phase.

依赖 viper, viper 依赖 symbolic execution verifier 依赖 z3.

处理库:
#[trusted]
手动对 library e.g. Vec 加 wrapper 包含必要的 specification.




## Main Obstacles and Resolution


------------------------------------------------------------------------------
# Evaluation
## Metric
验证主要有几个问题
1. Core Proof 是否能自动生成
2. 无需人力的 Core Proof 上的 e.g. panic check overflow check
3. functional specification: 其实只做了很少几个代码

以及我看了他们的源代码中的例子:
* binary search
  - 证明的是算法正确性
  - 冗长的 requires, ensures, invariant

* knight's tour:
  - 证明的是 panic absence, 也就是 knight 不会出棋盘界
  - 似乎它不能处理全局变量

## c.f. Existing solutions



# Retrospect
## Remaining confusions
* 这篇文章和 Rust 类型系统到底有什么联系, 能不能用到 C
  - Rust 是处于中心位置还是只是平台?

Rust 处于中心位置. 有指针的时候, 验证 alias 都很困难. c.f. seL4 怎么做的?
Rust 提供的 exclusive ownership & capability 正好匹配的分离逻辑中的 explicit aliasing.

* 验证的 infrastructure 是 viper, 有什么特点? c.f. Dafny
  - Dafny 的 infrastructure 是 z3, 理论是 SMT (是 SAT 和 theory-specific solver)

* 验证的自动化程度有多高
  - 至少不可避免地需要手写循环不变式: deductive verification 的基线
  - 可能还要看 Viper 的性质
  - Core proof generation, 以及 overflow / panic checking

* 文中提到了下面一句话 (Sect. 2), 我想知道来源
> This is in stark contrast to most existing verification techniques, which
> require a substantial initial effort to set up predicates, invariants, or ghost
> state and to verify memory safety, before programmers can turn to the
> properties they care about most.

一个例子是 C 程序的验证, 另外还如 type system based verification.
不过还需仔细检查.

* 它怎么处理全局变量的?
  - 论文中没提.

* 为何 modular?
  - 因为 "verify against specification interface instead of implementation"


## Future Work and Weakness of Proposed Solution
e.g. Performance? Labor? Economicalness? Elegance? Compatibility? Usefulness?

## My main take-away
* SMT 进步, 大家都开始用 z3
* Viper 很强, 甚至它把 spec 直接扔到 Viper 里面
* aliasing 和 mutability 在一起会导致很多问题.
  SL 和 Rust 都是解决这个问题的.


# Others
代码在 github, ETH 也有主页.
* [例子](https://github.com/viperproject/prusti-dev/tree/master/evaluation/artifact/examples).
* [项目主页](https://www.pm.inf.ethz.ch/research/prusti.html).
* [Viper 项目](https://www.pm.inf.ethz.ch/research/viper.html)

* Working paper 中提到的 Knight's tour 的 ghost code 在 OOPSLA'19 中已经没了.
  源代码里面也没有.

* 其实 Fig. 1 的例子, `shift_x` 改成 `p.x += s+1` 更好,
  这样如果有 aliasing 那么 assertion 就不会成立了.

* 其实是一个弱点: 它也需要学习成本, 因为 spec 编写的语言是 Viper 而非 Rust
  - 所以出现 `p1.x != p.x && p1.y != p.y && p1.x >= 0 ...` 而不是 `valid(p1) && p1 != p`
  - 不过只是工程上的弱点
