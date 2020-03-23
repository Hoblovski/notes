# This Architecture Tastes Like Microarchitecture
ISA 的趋势是暴露而非抽象 microarch 的实现细节, 还被 RISC 促进.

## MIPS
早期 RISC 的代表, 让软件能够直接了解 microarch 的细节包括流水线阶段等.

作者意见
* 硬件不能信任软件来保持自己的内部完整性
* 没有 microcode 不能保持向前兼容 ? microcode 可以帮助 "正确性", "安全", "兼容" ?

设计错误
* delay slot: 现在公认是错误决定, 一般都用透明的实现.
* branch hint: 动态的 branch prediction 比静态的 hint 更好.
* SIMD vs RISC: SIMD 定长, 之后新的长度需要改动指令编码.
* ...

其他设计
* TTA (transport triggered architecture): 指令说明数据在 functional unit 中的传输
  作者: nonportable
* RLA (register less archicture): 反正都有 cache, 而且还比较快,
  为什么不去掉 reg 呢?

更高层的抽象
* ISA 应当直接暴露 microarch, 抽象应当和底层无关, 来最小化兼容问题 ?
