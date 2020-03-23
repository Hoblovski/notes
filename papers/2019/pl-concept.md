# Fundamental Concepts in Programming Languages

* 67 年最初提出的文章, 希望严格化语言中的概念

> No axiomatization without insight

# 赋值操作

> [I]t is important to resist the temptation to start with a confusingly simple example

* 赋值操作不能被简单地写成 `e1 := e2`, 需要区分 lvalue 和 rvalue.

* lvalue: location. 关键特性有
  - 有一个关联的值 rvalue
  - 可以更新 lvalue 中存储的值

* 赋值改变 lvalue 中的值, 但不能改变 name 到 lvalue 的 binding.

# Expressions and commands

* 数学中只有描述性的表述 "sum of x and y" 没有命令式的表述 "add x to y".
  也就是说, 数学中没有 lvalue.

## Expressions
* All expressions have an associated value, either lvalue or rvalue.

* Expressions are inside some environment providing value for its components.
  $\to$ where clauses, lambda expressions, initialized variables.

* Bound variables: a for `let a=5 in a+b`.

* Free variables: b for `let a=5 in a+b`. Its value is given by the environment.

* Expressions can always be written in as an applicative structure: `(+ 1 (* 2 3))`.

* Evaluation: evaluation of operator is only preliminary but not whole to applying it to its operands.
  - partial (but not complete) ordering on evaluation of subexpressions

* Introduce currying: each operator has only one operand.
  - Key insight: there's no restriction on the order of evaluating the operator and operand(s).

* *referential transparency*:
> [I]f we wish to find the value of an expression which contains a
> sub-expression, the only thing we need to know about the sub-expression is its
> value. Any other features of the sub-expression, such as its internal
> structure, the number and nature of its components, the order in which they are
> evaluated or the colour of the ink in which they are written, are irrelevant to
> the value of the main expression.

* `if x = 0 then 0 else 1/x`
  - Not `(if (eq x 0))(0, 1/x)`
  - But `(((if (eq x 0)) (a=>0, a=>1/a)) x)`.

## Commands and sequencing

> Curiously enough mathematicians tend to call these things ‘variables’ although
> their most important property is precisely that they do not vary.

* The referential transparency of mathematical immutable variables is destroyed with updatable lvalues
  - But if we associate names with 'generalized addresses' instead of enclosing rvalues...
  - Need $\sigma$ operator doing abstract load.
  - Each assignment alters (introduces a new) $\sigma$ while invalidating the old one.
  - (dzy) $\sigma$ could be thought of as a *continuation*!

## Functional abstractions
* Parameters are values.
  - lvalue $\to$ pass by reference
  - rvalue $\to$ pass by value.

* Free variables in functions:
  - as rvalues: the rvalue is frozen at the function definition
  - as lvalues: the abstraction address is frozen.

* Own variables: now called closures. `let a=a in ...`.

* Functions and routines: functions should not have side effects and thus don't break referential transparency.

* Routine and functions are by default constant, i.e. non-updatable lvalues.
  There's no way reassigning/redefining a function.

* Like constants and variables, there're
  - Free functions: use free variables by reference.
  - Fixed functions: no free variables, or only constant/fixed variables.

* Fixed functions are always the same regardless of its environment.
  Fixity is property of the rvalue lambda expression, while constancy is of lvalue.

## Functions and routines as data items
> Thus in a sense procedures in A LGOL are second class citizens—they always have
> to appear in person and can never be represented by a variable or expression

* Making functions into first class citizens:
  - The lvalue of a functions is an abstract address where the rvalue is stored.
  - The rvalue contains the rule for evaluating the expression, as well as an environment supplying free variables.
  - Note that a function rvalue, a.k.a. a closure, does not contain those information. It only points to those information.
  - Defining a function: associating a 'shorthand' for a function rvalue.

## Types and polymorphism
> We call ambiguous operators of this sort polymorphic as they have several forms
> depending on their arguments

> It is natural to ask whether type is an attribute of an L-value
> or of an R-value—of a location or of its content.

* Type of every expression can be determined at compile time: *manifest*.

* Type only determinable by running: *latent*.
  Machines at that time could not afford to run dynamic typed programs.

* Polymorphism:
  - Ad hoc: no systematic way. Several special rules.
  - Parametric

* Types of functions:
  - Are polymorphic functions first-class?

* Compound data structures:
  - Assignments: the rvalue of compound structures must give access to the lvalues of the components.
  - Shallow copy.

> Notice that the type of a pointer includes the type of the thing it points to,
> so that pointers form an example of parametric type.


# Other topics
* Semantics of programs:
  - lambda expressions with assignments and jumps
  - program string manipulation
  - 
