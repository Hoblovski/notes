# Pluggable Type Systems

传统 mandatory (要编译, 必须符合程序的 typesys) 有害:
* 损害 expressiveness
* 大家依赖它, 但是实际中 typesys 复杂, 难以 sound.

optional type system:
* 不改变运行时语义
* 不要求 type annotation

e.g. theoretical lambda calculi: typesys 只是否定某些非 well-typed 的程序.
但是不改变语义.

可以支持一个语言上多个 typesys.
