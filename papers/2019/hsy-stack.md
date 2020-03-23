# A Scalable Lock-free Stack Algorithm

# Problem
## Research Question
并发访问的 stack.
* linearizable
* nonblocking
* scalable

称为 elimination backoff stack.

## Existing Solutions, Their Troubles, c.f. Proposed Solution
* combining funnels:
  - linearizable
  - 常数太大, low load 不好

* Treiber stack
  - 简单常数小而且 linearizable
  - contention 导致 high load scalability 不好

## My Proposed Answer
NO

# Proposed Solution
## Background

## General Idea
TODO

考虑 push

* 尝试使用 cas 压入栈 (链表), 如果成功那么 ok
  - 否则失败, 因为 contention

* 将 op 放入 elimination array : loc[thrid] 中.

* 随机选择 loc[him], 如果 him 的确正在进行 pop (并且也因为 contention 失败), 
  那么尝试结合 push 和 pop.
  - cas ( & loc[thrid], 这个 op, NULL)
  - cas ( & him, 原来的 q, 这个 op)

-> linearization point 可能在其他线程里面.

$(\sigma, (U, \theta))$ 其中 $U$ : thread -> aop

lin(t): $U$ 中线程 $t$ 的 aop 执行

## Main Obstacles and Resolution



# Evaluation
## Metric

## c.f. Existing solutions



# Retrospect
## Remaining confusions

## Future Work and Weakness of Proposed Solution
e.g. Performance? Labor? Economicalness? Elegance? Compatibility? Usefulness?

## My main take-away


# Others
