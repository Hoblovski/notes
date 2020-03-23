# Liquid Types
PLDI'08

# Problem
## Research Question
(最终目的是证明程序安全性质, 和依赖类型一样)
依赖类型完成类型推断.

predicate abstraction + HM 类型推断实现依赖类型.

## Existing Solutions, Their Troubles, c.f. Proposed Solution
* 正确性: Hoare Logic etc
  - 也算一种方法

* 依赖类型: Coq, DML
  - 需要标注太多, 毕竟逻辑不完备

## My Proposed Answer
* 没有


# Proposed Solution
## Background
Dependent type: 使得 type system 可以表示更 fine-grained 的约束.
但是已有方案 (DML) 缺点是需要的手动标注太多 (因为 type-checking 已经不自动了).

* base refinement: $\{ \nu: B \;|\; p(\nu)\}$:
  - 一种 dependent type system 的方法, 本文基础
  - 其中 $\nu$ 称为 value variable, $p$ 成为 refinement 

* liquid type variable:
  - unknown type refinements, 其实是 refinement predicate
  - 用 $\kappa$ 表示

* logical qualifier: 包含 PVar, $\nu$ 以及 $*$ 的命题.
  - $\mathbb{Q}^*$ 是所有 logical qualifier 的集合
  - qualifier 的形式是*受限制*的!

## General Idea
三步
1. 先 HM 跑出 $B$, refinement predicate 还不知道所以用 $\kappa$ 代替
2. 生成子类约束
3. 约束求解

## Main Obstacles and Resolution



# Evaluation
## Metric
减少程序证明需要的标注.

## c.f. Existing solutions



# Retrospect
## Remaining confusions

## Future Work and Weakness of Proposed Solution
e.g. Performance? Labor? Economicalness? Elegance? Compatibility? Usefulness?

## My main take-away


# Others

