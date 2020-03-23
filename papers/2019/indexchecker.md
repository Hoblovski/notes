# Lightweight Verification of Array Indexing

# Problem
## Research Question
针对数组下标, 完成高效可靠 (sound) 的编译期验证.

## Existing Solutions, Their Troubles, c.f. Proposed Solution
我:
* 动态检查: trivial, 不能 *prevent*
* z3: 似乎太重量了, 而且很慢
* liquid type: C / C++ / (Java) 没有

它:
* 动态检测: Valgrind, Purify, Dependent type for low-level programming (ESOP'07), Cyclone (ATC '02), CCured (TOPLAS'05),
  - 耗时

* 通用静态分析: FindBugs, Coverity (CACM'10)

* SMT: extended static checking, dafny
  - 不稳定

* 类型系统: liquid type (依赖类型, PLDI'08), Enforcing correct array indexes with a type system (更简单, FSE'16)
  - 利用 omega / fourier 进行判定
  - IC 的 TS 更弱, 但自称更 practical

* 抽象解释: Clousot (.net 上), cccheck
  - 抽象解释和类型是一回事 (Types as abstract interpretations)


## My Proposed Answer
搞个 index calculus 然后特别处理.


# Proposed Solution
## General Idea
只考虑这一个特别的问题. 只需要保证 `0 <= i <= a.length`.

使用 Java annotation 和 pluggable type (Checker framework),
加入表示数组长度的 type qualifier.

只考虑七种简单情况, 然后每种都用一个 type qualifier 表示.
type qualifier 只作用于 int 和数组.

程序员需要在字段和参数上手写 qualifier, 然后工具做 local type inference.
qualifier 是 flow sensitive 的.

`x` 表示任何 int 表达式, `c` 表示常量 int 表达式, `a` 表示另外一个数组.
`v` 表示被 qualify 的 int, `l` 表示被 qualify 的数组的长度.

--------------------------------------------------

* 下界, 给 int 的
```
@Positive   | @NonNegative  | @GTENegativeOne   | @LowerBoundUnknown
v >=1       | v >= 0        | v >= -1           | unk
```

* 上界, 给 int 的
```
@LTLengthOf(a, x)
v < a.length - x
```

我觉得这个东西莫名其妙, 直接搞个 range analysis 有那么慢吗?

* 常量范围, 给 int 的
```
c1 <= v <= c2
```

* 长度下界, 给数组的
```
@MinLen(c)
l >= c
```

* 长度相等, 给数组的
```
@SameLen(a)
l == a.length
```

* 简单线性不等式:
```
v >= xxxxx  -> v >= xxx
```

* 复数下标: 专门处理 binarySearch

## Main Obstacles and Resolution
处理非线性: 用 top 近似 (注意常数的已经被处理了).

不调用外部求解器, 手动实现 inference rule.

各个 TS 之间的循环依赖: 手动指定一个顺序.



# Evaluation
## Metric
Java 程序: Guava, JFreeChart, Plume-lib.

手写 annotation 然后检查.

## c.f. Existing solutions
也没什么特别可比的.


# Retrospect
## Remaining confusions
处理 array of arrays?

处理 array aliasing? 比如 a.arr 和 b.arr 是别名的?

## Future Work and Weakness of Proposed Solution
e.g. Performance? Labor? Economicalness? Elegance? Compatibility? Usefulness?

## My main take-away


# Others
软工论文喜欢写 case study 或者 threats to validity.
而且特别会写故事.

然后一堆 false positive 找各种理由说不是问题.

Type-checking is a dataflow analysis that produces sound estimates of what a program may compute.
