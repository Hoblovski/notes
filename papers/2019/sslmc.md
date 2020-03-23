# Finite-State Analysis of SSL 3.0

# Problem
## Research Question
渐进式地, 使用 model checking, 验证 SSL 3.0 协议.

## Existing Solutions, Their Troubles, c.f. Proposed Solution

## My Proposed Answer



# Proposed Solution
## Background
Murphi
* 是一个 "FSM 验证器"
  - 用来做 finite state analysis 即模型检查
  - 验证过缓存一致性协议
  - 可以验证 safety 条件
* 基于 transition 而非 process
  - 状态是全局变量, 局部变量
  - 计算模型: asynchronous, interlaving
  - scalability: 限制 server / client 的数量

SSL 3.0 handshake
 * 目的是在通信双方 (C/S) 间建立共享密钥

## General Idea
渐进式验证:
* 从简单而不安全的协议触发, 每次改正 MC 发现的攻击, 最后达到 SSL 3.0 的近似.

步骤
* 形式化协议
* 加入入侵者
  - 窃听消息, 重发消息, 如果有 key 那么解密, 截断消息,
    (从初始知识和听到的消息中) 生成消息.
* 写安全条件: e.g. 入侵者不能知道秘密
* 运行检查模型
  - 检查输出是否是真的攻击, 改进模型

TODO

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

