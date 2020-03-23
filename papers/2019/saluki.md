# Saluki: Finding Taint-style Vulnerabilities with Static Property Checking



# 问题背景
二进制分析.
检查 taint-style security properties,
e.g. read(2) 的结果不能直接喂给 system(3).

property: 某条路径上符合的 predicate.

基本 workflow: 找出 data dependency, 检查它们是否符合 policy.
然后用一个逻辑检查器来检查 数据/控制依赖 满足 security properties.



# Questions
1. 是否使用 typing 以及 qualifiers?
它使用 logic engine. 自称也可看成一种 Model checking.

2. 检查的性质是什么?
用户指定的 security policy, 如 read-system 例子.

3. 他所说的 *context sensitivity* 指什么?
引用 Context-Sensitive Program Analysis as Database Queries.

考虑执行上下文 e.g. 在哪里, 整个 caller chain 上有那些元素.
所以 context-sensitive data dependency 就类似:
如果从 `f0()` 调用过来, 那么 `a` 会依赖 `b`.


