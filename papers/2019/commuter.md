# The Scalable Commutativity Rule: Designing Scalable Software for Multicore Processors

# INTRODUCTION
Shared memory multicore processor, with MESI-like cache coherence
* R/W of exclusive cache is scalable
* R/W of shared cache is scalable
* **Writing shared/remote cache is not scalable**

*dzy: so it means shared data?*

Simple model: operations are scalable iff they are **conflict-free memory access**.
In complex real world that's not the case, e.g. memory bus overwhelmed.

Scalable Commutativity Rule:
> commutable interface operations $\to$ implementable in a scalable way.

# THE SCALABILITY COMMUTATIVITY RULE
## Terminology
Model *system execution* $H$ as a series of *actions*. It's also called *history*.
*Action* is either *invocation* (e.g. syscall) or *response* (e.g. syscall return value).
*Invocation* and *response* are paired, in the context of some thread.
An *action* comprises
* an operation class (e.g. syscall ID)
* arguments / return value
* a context thread
* a uniqueness tag

The *specification* $\mathcal{S}$ tells whether a *history* is correct
(e.g. limiting return values). It's a prefix-closed set of histories.

Denote a reordering of $H$ as $H'$, then $H'$ must preserve any thread restricted view
$$
\forall t\;.\; H|_t = H'|_t
$$

$Y$ *SI-commutes* in $H=X Y$ iff
$$
\forall Z\;.\; X Y Z \in \mathcal{S} \leftrightarrow X Y' Z \in \mathcal{S}
$$

$Y$ *SIM-commutes* in $H=X Y$ iff
$$
\forall P \sqsubset Y'\;.\; P \, \text{SI-commutes in}\, X P
$$

SIM commutativity is
* State-dependent:
  - captures commutable operations only in some states.
  - We don't universally quantify $X$.
* Interface-based:
  - Only use specification but not implementation.
  - States only need to be indistinguishable via the specification.
* Monotonic:
  - Commutativity not only for $Y'$ but also for any of its *prefixes*.

If we define
1. Implementation *states* $S$, and an initial state
2. Valid invocations $I$, including the special action $CONT$
3. Valid responses $R$, including $CONT$,

then the *implementation* is $m\;:\; S \times I \to S \times R$.
An *implementation* *generates* a *history* if it could produce that *history*.

Implementation being correct: the response it generates always allowed by the specification **TODO: what? Rigorously?**

## Illustration on constructive proof
WTF? skip 3.4, 3.5.

## Discussion
* Constructive implementation is not practical, but provides insight.
* In some cases commutativity is impossible.
* Commutativity is only sufficient, but not necessary
  i.e. non-commutativity does not mean absolutely no scalable ssolutions.

How to make systems scale?
* Used existing techniques
  - per-core resource allocation
  - double-checked locking
  - lock-free readers using RCU
  - scalable reference counts using Refcache
  - seqlocks

* Choose data structures like arrays / hashtable.
  - Counterexample is search trees. The root node is always shared.

* Defer work & split resources.
  - e.g. inode num = real inode || cpuid

* Double checked locking

* Split work:
  - f: If exist, return the inode => f1: if exist; f2: return inode

