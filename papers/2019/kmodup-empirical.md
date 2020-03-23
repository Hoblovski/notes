# 研究对象
  内核接口的变化
  如何把老版本内核的 kmod 自动升级使得能在新版本内核上运行

# 已有方法

* 手动升级 kmods: 新版本内核上编译, 改, 直到没有 error / warning
  缺点: 太烦

* 自动, 基于编译 error / warning
  缺点: 就算没有编译 error / warning, 也不一定运行就正确,
        i.e. 造成运行时行为变化

* change pattern inference:
    内核接口会被用很多次, 变化之后, 用到它的地方也要变化
    多个用到他的地方, 他们的变化应该是类似的 -> 一个 change pattern
    基于程序静态分析, 接口的变化通过用法的变化来反映
  SmPL .. 各种工具的原理, 不足  :: 这一部分各种工具的原理有点没看懂

# 实验例子
  SmPL 速成了一下

## 之前工作对于输入 (code change, commits) 假设
* take manual API usage updates as the inputs and infer patterns by abstracting irrelevant details
* 或者, 依赖于编译错误

## commit 里面的原始改动特征
* 单个 commit 可能包含多个 change patterns  -> 同时改变多个功能相关的 API.
* commit 可能不涉及 API 使用的变动 i.e. 和 change pattern 都没关系
* 较少 (不到 15%) commit 只导致单个 pattern instance, i.e. 非常偏僻的 API, 只有一个地方用到
* 仅 75% 的过时 API 使用会导致编译 error / warning
