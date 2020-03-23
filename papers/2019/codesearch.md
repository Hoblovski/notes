# How Developers Search for Code: A Case Study: FSE 2015.
研究 Google 程序员, 工作在 monorepo, 里面的 CS 行为.



# RELATED WORK
之前的研究方法有
* 直接观察程序员
* survey 调查
* 分析搜索日志
* 在 wild 或者 controlled environment

survey 可能的 bias 是, 人的记忆出错.

已有的 tool 如简单的 grep,
* Koders, Krugle etc 采用方法签名和关键词检索.
* 其他检索方法如, API 使用 (Exemplar), 类型 (S6), 测试 (S6, CodeGenie), 模式信息 (Sourcer)



# 结果
  看起来结果就是列举 finding.

## Search 的目的: Survey
让人回答问题, 然后用 open card sorting 完成分类.

大部分 60% 是 howto (怎么用, 怎么做, 给个例子) 以及 what (它做什么).

不少其他的, 如 "为什么不 work", "对象什么时候初始化的"...

## Search 的上下文: Survey
上下文指的是

* 做什么 (explore, solve...)
* 有多熟悉

做相关性分析

## Search 的性质: Logs
* 关键词平均个数: 1.85.

* 30% 的加了 file: 限制

* 连续的 SEARCH 间, search term 的 Levistein 距离有 76% 是 1
  - 就是增加 / 删除 / 修改一个词
  - 连续 SEARCH 间间隔时间不畅, 8 sec.

## 通常的 search session 怎样: Logs
* 就是 search 的 pattern, 主要是动作的序列: SEARCH, CLICK.

* 25% 的 session 没有 CLICK: 没找到可能有用的, 或者直接 SEARCH 结果给出了答案.

## 上下文和 search pattern 的相关性
* 连接 Survey 结果和 log 分析结果

* e.g. 熟悉代码那么通常是 S+C+



# DISCUSSION AND RESULTS
> Developers search for examples more than anything else



# 问题
1. 当然, 怎么 search?
  - CS 的 ctx?
  - ctx 的目标是回答什么 问题?

2. 使用了研究方法是什么
  - survey
  - log analysis

3. 这种软工的研究一般做什么, 是什么样子的?

这篇论文结构是
1. Introduction, Related Work:
  - 常规

2. Study:
  - 说明研究对象和研究 methodology
  - 因为软工是工程研究学科

3. Results:
  - 列出 study 的结果, 并且细节地解读

4. Discussions:
  - study 结果的 *高层次* implication

5. Threats to validity:
  - 实际是 limitation
  - 因为软工目的是 present 结果, 而非系统的 present design,
    所以说明的不是 design limitation 而 results validity

6. Conclusion, Future Work:
  - 常规

其中占比最大的是 Results 使用 5 页, 篇幅 50%.
其他的基本都各自一页, 包括 Study.
