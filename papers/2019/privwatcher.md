# PrivWatcher: Non-bypassable Monitoring and Protection of Process Credentials from Memory Corruption Attacks

# 问题背景
保护 process credentials (定义 process 身份, 如描述是否特权的数据) 不受 corruption (不是硬件随机失效, 而是恶意攻击) 的损害.

# 方法
1. 把 process credentials 和其他数据分开处理.
2. 一个从内核隔离开的 execution domain 上跑 monitor, 利用 MMU 把所有 credentials 放到内核不可写的区域.
  防止内核随意改动 privilege.

# QUESTIONS
1. 是否使用了 type 来做 isolation / access control?
没有.
因此和我们的调研无关.

2. 如果做了分割, 按照什么依据?
略.
