# Local Rely-Guarantee Reasoning

RG: specify interference

RG 的问题;
* 线程私有的状态
* 线程如果只访问共享状态的一部分
* 动态分配的资源
* 线程组私有的状态

Contribution
* 通过 frame rule 和 hiding rule 给 RG 实现 modularity
* simplify RG



关键思想

RG and CSL
* parallell composition

RG
* stability: non-interference

CSL:
* stability: non-interference



transisition 的 disjoint conjection 
$$
    (\sigma, \sigma') \models a_1 * a_2
    \Longleftrightarrow\\
    \exists \sigma_1\, \sigma_2\, \sigma'_1\, \sigma'_2.\,
    \sigma = \sigma_1 \uplus \sigma_2 \land \sigma' = \sigma'_1 \uplus \sigma'_2 \\\land
    (\sigma_1, \sigma'_1) \models a_1 \land
    (\sigma_2, \sigma'_2) \models a_2 \land
$$

为什么引入 invariants? 为了 useful modularity, 希望的是
$$
    \frac
        {
         p \;\text{Sta}\; a
         \qquad
         p' \;\text{Sta}\; a'
        }{ 
         p * p' \;\text{Sta}\; a * a'
        }
$$

但是不行: 可能 $a'$ 改 $p$ 而 $a$ 改 $p'$ 所需需要 precise action.

$ I \rhd a $ iff
* $I$ 是 precise
  - 描述唯一的 substate, 这个 substate 是 $a$ 应该关心的
* $[I] \Rightarrow a$
  - (当然 state 都满足 $I$ 的前提下) identity 满足 $a$
* $a \Rightarrow I \ltimes I$
  - $a$ 保存 $I$


