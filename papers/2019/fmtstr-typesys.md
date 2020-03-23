# A Type System for Format Strings



# 摘要
问题: fmtstr 不安全.

方法: fmtstr 的 typesys.

验证: java format.



# 方法
* soundness 要求:
  - fmtstr 格式对
  - 参数个数对
  - 参数类型对


用 4 个 qualifier:
* *Format*: 加到 String 上. 表示作为 fmtstr 是可以的.
* *InvalidFormat*: 加到 String 上.
* *Unknown*: *Format* join *InvalidFormat*
* *FormatBottom*: *Format* meet *InvalidFormat*, 应该是空的.

可以学习, 它限制 qualifier 到 string 上就不用讨论其他类型上的 qualifier 了.

Listing 3 的例子很好:
* 类型如 `@Format({FLOAT, INT}) String`
* 那么 `"adsa %f %d 123"` 是这个类型, 但 `"%c %d"` 不行.
* 引入了 subtyping 的概念, 让我眼前一亮.

在 String.format 调用处, 特别比较 fmtstr 的类型 (重点是 qualifier) 和参数的类型.


## 细节
如下, `format` 的类型 qualifier 依赖于 args 的值, 类似 dependent qualifier.
`public final void log(@FormatFor("args") String format, Object... args)`.

解决方法就是 *FormatFor*, 然后内部就允许 `String.format(format, args)`, 转而在 callsite 检查.



# 其他
* Java 的 checker framework 插拔式类型, 作为 annotation

* 有时静态检查不够, 再加一个动态检查 `isGoodFormatFor(fmt, obj...)`.

* false positive:
  - `format("%"+"d", n)`
  - `format("%"+width+"f", v)`
  - format 出错被 exception catch
  - fmtstr 被复杂地计算出来
