# All You Ever Wanted to Know About Dynamic Taint Analysis and Forward Symbolic Execution (but might have been afraid to ask)



# INTRODUCTION
Dynamic analysis include dynamic taint analysis, forward symbolic execution etc.

Taint analysis can be used to detect defects, while symbolic execution generate concrete test inputs.

**Contribution**: formalize DTA and FSE, using formal run-time semantics of the language.



# Language
* Intermediate language (SIMPIL).
  - Paradigm: imperative, untyped.
  - Constructs: updatable variables, only goto and no functions calls.

* Uses natural operational semantics. Disallow self-modifying code.



# Dynamic Taint Analysis
Value depending on taint source is *tainted*, otherwise *untainted*.

* Taint policy: how taints flow.
  - how taints are introduced
  - how taints propagate
  - the taint checking rules

* *Overtainting / Undertainting*: false positive / false negative.

* Extend SIMPIL to incorporate taint status into a value.

* $P_{\text{op-check}}$: is it safe to perform op *if op involves tainted value*?
  - Possible that even if op involves tainted value, op is always safe.

## Subtleties
1. Treat memory as tainted or untainted by default?

2. Are taint status of the address of pointers and their values independent?

3. *Taint sanitization*: when can taint be removed?
  - ​		e.g. `a xor a`
  - cryptographically secure functions always gives untainted values.
  - when the program itself performs sanitization.



# Forward symbolic execution
* Introduce path predicate $\Pi$ into execution environment.

## Subtleties
* Symbolic memory
  - Point-to all possible, alias analysis, SMT solver

* External interactions
  - Models; concolic

* Path selection (e.g. loops?)
  - Random, DFS, concolic, heuristics



# QUESTIONS
* [ ] Why is forward symbolic execution related to dynamic taint analysis?

Whey are both frequently used dynamic analyses in security research.

* [x] What are taint analysis's applications?

* Find unknown vulnerabilities.
* Malware analysis.

* [x] What's formal *runtime* semantics, of which language?
  Is it operational semantics?


