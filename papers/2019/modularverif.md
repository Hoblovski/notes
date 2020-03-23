# Modular Verification for Computer Security

# Stance
*Modularity principles* enabling verification of *system security components*.

而且还强调是 end-to-end, modular, deductive program verification.



# Argumentation
## Background
论文把普通应用和底下的 *system security components* (forth SSC) e.g. compiler, OS, crypto 区分.

通常, 软件的 trusted base 很大: 从库和 OS 直到硬件 netlist.
但是这些 base 是模块化地组织起来的, 所以
* 全面, 端到断, 模块化的的系统验证是可行的
* 验证需分不同层面: 如 program, algorithm
* 高阶函数式语言用于 spec 是有效的
* 为了简化验证的链接, 应使用一个通用逻辑

为了保证可执行代码的正确性, 有几个 proof:
* compiler correctness
* refinement proof
* declarative proof

## Refinement Proof
静态分析只考虑 safety / liveness, 但是对 system security component 不够.
SSC 还需要 functional correctness.

这一层主要是 refinement proof.
因为 functional spec 和 declarative proof 之间验证相对容易.

如何写 spec? 传统是一阶语言, 而且通常是逻辑语言.
* 论点: **用函数式语言**
  - 即使 verbose, 但是 experience 证明有效.
* 理由: 函数式语言容易分析
  - 因此作为 domain specific property 和 implementation 的交接线
  - 交接线两边可以独立进行证明, 模块化
* 例子: CertiKOS (Gallina), HMAC (Gallina), seL4 (HOL-fun), Raft (Gallina).

如何表示复杂的系统 i.e. 不能简单的被函数建模, 包含内部状态的系统?
* 论点: **语言要支持高阶逻辑**
* 理由: 用函数式语言中 dependent record, 内部状态用 ADT 和 existential quantification 表示**
  - c.f. representational state 以及 representational invariant
* 理由: 因为用 existential quantification 实现, 所以需要对类型和谓词量化
  - 例如
```
exists T : type (* of hidden representation*) .
exists P : predicate (* of representational invariant*) .
forall x : T . P(x) ->
exists y (* after some interface function *) : T .
    P(y)
```

结论: 使用 functional language + higher order logic.

### Correctness-by-construction
除了证明 impl 对应 functional spec, 另一种方法是从 functional spec 抽取 impl.
这种方法就是 program synthesis, 或者 proof extraction.

* CakeML:   直接把证明编译成 executable binary, 编译器是被验证了的.
* CertiCoq: 把证明翻译成 C light, 然后用 Compcert.
* Cogent:   linear-typed 函数式语言, 所以不需要 **垃圾回收**

## Compiler verification
最终可执行代码还不完全对应程序, 程序验证不能完全对应可执行代码的验证.

一种方法是 ad-hoc validation, 例如 seL4 使用 gcc 和 custom C semantics.
* 论点: 编译器的 interface 应该是 AST + SOS (小步语义).
  - 理由: 发展成熟, 更好的 modularity
* 论点: validation 是个例都不一样, CompCert 更通用
  - 事实: 而且效率更高, 因为 validation 其实是 model checking
  - 推论: CompCert 的方法更好.
  - proof by reflection: 把证明正确的程序用到其他证明里
* 论点: 被链接的证明应当使用同一逻辑和证明系统
  - 理由: seL4 Isalella/HOL vs. CompCert coq

导出问题: C 语言的形式化.
* C 缺乏形式化定义
  - C lawyer 的出现
  - 语义用 limitus test
  - UB 导致不期待的结果

## Others
函数式 spec 的缺点: 不好处理 randomness.
以及逻辑限制不好处理 intrinsic sharing + mutability.

关于硬件
* RTL 到 netlist 在硬件层面验证用的很多.
* ISA 到 RTL 还不多, ISA 通常都是 informal.
* timing channel 是另外的问题

# Retrospect
## Remaining confusions
* 讨论了很多题目, 但是题目之间的联系是什么?
  - 上面梳理了逻辑

* 如果说 modular verification, 那么什么是 nonmodular verification?
  - 验证了这个, 那个, 放到一起又要重新验证
  - seL4 + CompCert? :(

## Future Work and Weakness of Proposed Solution
* higher-order logic: 逻辑学似乎研究较少?

* functional language for spec: 是否 verbose?

当然 future work 就是 full-scale end-to-end.

## My main take-away
主要的论点: higher-order logic 和 functional language for spec.
