# An Implementation and Analysis of a Kernel Network Stack in Go with the CSP Style

# 目的
* 尝试高级语言写 OS (子系统)
* 尝试利用多核

# 相关工作
* MirageOS: unikernel, OCaml.
* Pycorn: python.

# tap 网络接口
虚拟网络接口, 可读写.
加上 bridge.

# Evaluation
* baseline 是 tapip, 非主流.
* 代码: GoNet 更简单. 没有定量分析, 只是给出例子.
* latency: 并发度较小时 tapip 更好, 并发度变大之后 GoNet 更好 (因为更好的并行性)
* throughput: GoNet 更好.

# Conclusions
* 指明了实验的限制
