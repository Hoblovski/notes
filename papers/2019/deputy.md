# Dependent Types for Low-Level Programming
ESOP'07

# Problem
## Research Question
集成依赖类型到 C 等低级命令式语言中.

## Existing Solutions, Their Troubles, c.f. Proposed Solution
已经有的依赖类型: DML (POPL'99), Xanadu (LICS'00), Cayenne (ICFP'98)

表示程序性质:
* 静态分析
* 动态检查: fat pointers 等. 运行开销, 修改程序不兼容
  - CCured (Necula, 同一帮人), Cyclone

SafeDrive (OSDI'06): 同一波人


# Proposed Solution
## General Idea
C 中需要依赖类型表达的性质最突出的就如下.
* 数组 (也含指针) : 长度
* union: 那个种类

所以其实需要求解的命题是 $e_1 \diamond e_2$, 其中 $\diamond$ 是比较算子.

Deputy 分三步
* [Infer Annotation] 推断局部指针的长度
* [Instrumentation] 每次用一个指针前, 插入 assert 检查其长度约束
* [Optimization] 用数据流分析消除大部分 assert

这里的依赖类型是流不敏感的.

加 dependency 也就是说
* 源代码里是 $x : \text{arrptr}\, T$, 推断后变成 $x:\text{arrptr}\, T\, l$.
  - 加入 arrptr 依赖的长度.
* 论文里面 $\Delta$ 是说, 如果 $\tau :: T_1 \Rightarrow T_2 \Rightarrow *$ 并且 $x : \tau\, e_1, e_2$ 那么 $\Delta(x) = \langle e_1, e_2\rangle$

## Main Obstacles and Resolution
Mutation:
命令式语言允许值的变化. 类型可以依赖值, 所以赋值语句可能导致类型变化.
这对于流不敏感的类型系统是不允许的.
* 特别处理赋值语句的类型规则;
  - 现在 $\Gamma\vdash e:T$ 是有条件的了, 例如访问内存需要保证指针有效
* 类型所依赖的值中, 只能出现 *局部变量* e.g. `int * count(n) arr`
  - 不能出现内存访问 e.g. `int * count(*p) arr`
  - 赋值只有在左手边是 *局部变量* 的时候才需要 assert


# Evaluation
## Metric
在 CIL (OCaml 那个) 实现, 18000 loc.

有两个度量:
* (因为要求手动 annotate) 需要改动多少代码
* (因为插桩了动态检查) 运行时开销

## c.f. Existing solutions
只是提了一句比 CCured 快, 但是 CCured 比它强.


# Retrospect
## Remaining confusions

## Future Work and Weakness of Proposed Solution
e.g. Performance? Labor? Economicalness? Elegance? Compatibility? Usefulness?

## My main take-away


# Others
作者用 $\Gamma\vdash c\Rightarrow c'$ 表示插桩.

$$
    \frac
        { a : \text{arrptr}\, T\, l}
        { a + l' : \text{arrptr}\, T\, (l-l')}
$$
