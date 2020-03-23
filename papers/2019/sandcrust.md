# PLOS'17 Sandcrust: Automatic Sandboxing of Unsafe Components in Rust

* Rust 的内存安全代码和 C 链接到一起的时候, C 代码可能破坏 Rust 的内存安全.

* 让 C 代码以及 Rust wrapper 在隔离的另外一个进程中运行,
  利用 RPC 调用 remote rust wrapper.

* 用 macro 保证编程的简单, 不过也可以用 syntax extension.

* 用 unix pipe 和 serde 完成 RPC.
