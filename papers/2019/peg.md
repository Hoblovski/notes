# Parsing Expression Grammars: A Recognition-Based Syntactic Foundation
> POPL'04



# INTRODUCTION
CFG is generative based system, while PEG is recognition based system.

PEG may be viewed as a formal description of a top-down parser.



# DESCRIPTION
* Not ambiguous, use prioritized choice operator `/` instead of `|`.

* Repetition operator `*` similar to EBNF.

* Syntactic predicates: `&e` or `!e`.
  Attempt to match without consuming input,
  and returns to starting point whether matching succeeded or failed.

> Identifier !LEFTARROW: any identifier not followed by a left arrow.

* Originally lexing and parsing are separated because CFG is unsuitable for lexing. That's not the case with PEG, resulting in a unified grammar with no distinction between lexing and parsing.


## Comparison to CFG
* **More power**: e.g. C++ template `>>` problem; Pascal like nested comments;
  dangling 'if' problem.

* **Restrictions**: left recursion;
  not able to use semantic predicates e.g. typedef vs multiplication.


## Formal definition
4-Tuple $G = (N, T, R, S)$. Rules $R$ maps nonterminal $A$ to parsing expression $e$,
written as $A \leftarrow e$.

* Define legal 'parsing expression' inductively,
  from empty string / terminals / nontermial symbols,
  formed by sequencing, prioritized choice, repetitions, and not-predicate.


## Properties
* PEL is closed under intersection, union and complement.

* Whether PEL is empty is undecidable, complete.



# Other
TODO



# OPEN PROBLEM
* Whether operands of `/` are disjoint.
* Inter-convertibility of PEG and CFG.

# RELATED WORK


