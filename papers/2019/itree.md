已有的方法:
操作语义: noncompositional, not executable because relational
....trace model.
itree: 指称语义.

```
CoInductive itree (E: Type → Type) (R: Type): Type :=
| Ret (r: R)                                   (* computation terminating with value r *)
| Tau (t: itree E R)                           (* "silent" tau transition with child t *)
| Vis {A: Type} (e : E A) (k : A → itree E R). (* visible event e yielding an answer in A *)
```


E : effect, interaction, event.
E 的参数是 A : answer, 表示
    A (for answer) is the type of data that the environment pro- vides in response to the event.
R : 计算返回值

(itree E) 是一个 monad.

可以用树表示, 叶子是 Ret, 中间平凡边是 Tau, 分叉中间点是 Vis.
bind 就是把每个 Ret 又换成 Vis.


strong bisimulation: t1 ~= t2: 有完全一样的 shape.
weak: ≈ 允许移除有限个 tau. i.e. eutt eq.

    If we have t1 : itree E A and t2 : itree E B and some relation r : A → B → Prop
    , we can define eutt r (“equivalence up to Tau modulo r ”), which is the same
    as ≈ except that two leaves Ret a and Ret b are related iff r a b holds.

也就是说, event 是一样的, 而且

eutt bisimulation

mu 是最小不动点, nu 是最大不动点


证明 itree 等价性, 但是 coinduction 太复杂, 所以用一堆 monad / structural / congruence law 来代替


# 3
itree 包含了一堆 events.
To add semantics to the events of an ITree, we define an event handler, of type E ~> M for some M.

E A |-> M A:
interaction, environment returns answer |-> computation, argument

例子: 把 itree 理解为 state monad


# IMP Compiler

指称语义:
    denote_expr, denote_imp : 把 expr/imp 映射成包含 ImpState(就是 Imp Event) 的 itree

state monad 语义
    interp_imp : 把包含 ITree

简单语义: env 到 env 的函数
    imp -> env -> (env * unit)

现在:
    -- Definition stateT (S:Type) (M:Type → Type) (R:Type) : Type := S → M (S * R).
    -- stateT S M R ::= S -> M (S * R)
    -- imp -> stateT env (itree F) unit
    imp -> env -> itree F (env * unit)
    -- c.f. imp -> env -> (env * unit)

最终的语义就是
    s |> denote_imp |> interp_imp

证明 program transformation 的正确性: denote_imp c = denote_imp' c.

`-< 是 subevent`
```
Infix "≅" := (eq_itree eq) (at level 70) : itree_scope.

Infix "≈" := (eutt eq) (at level 70) : itree_scope.

Infix "≳" := (euttge eq) (at level 70) : itree_scope.

```

