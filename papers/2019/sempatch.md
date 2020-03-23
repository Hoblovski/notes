# Semantic Patches

> Y. Padioleau, J. L. Lawall, and G. Muller. SmPL: A domain-specific language for specifying collateral evolutions in linux device drivers. Electr. Notes Theor.  Comput. Sci, 166:47–62, 2007.

* **目的**: 自动化外部驱动和内核保持版本同步.

* **semantic patch**: 和普通的 git patch 相对.
  但是 git patch 是基于行的,
  而 semantic patch 是基于语义的.

# 例子
* **变化**: SCSI 删除了 `scsi_host_hn_get()` 和 `scsi_host_put()`.
  之前用到 SCSI Host 的函数都是传入 `int hostno`,
  然后 `scsi_host_hn_get(hostno)` 得到 `host`,
  离开函数的时候调用 `scsi_host_put(host)`.
  现在则是传入一个 `struct Scsi_Host* host`.

* 行首 +, - 分别是增加, 删除的代码, 其余是所谓上下文.

* 一个 semantic patch 包含完整的 C 词项 (parser term), 比如改变函数参数,
  就要把函数声明完全写出来, 修改的地方再 + 或者 -.

* semantic patch 中空格, 换行被忽略

* 可以声明 metavariable, 用来匹配任何一个词项 (主要是那些不能在 patch 中死编码的参数名等).
  metavariable 的声明在两个 `@@` 之间, patch 具体内容之前.
  单个声明的格式是 `MetaVarType MetaVar [,MetaVar]*`.
  `MetaVarType` 可以是 `identifier`, `expression`, `statement`, 以及具体的 C 类型如
  `off_t`, `int`.

* 若干 metavar 的声明和一个变换模式 (transformation spec) 称为一个 rule.

* metavariable 继承: sempatch 能有多个 rule, 如
```
@ rule1 @
struct SHT ops;
identifier proc_info_func;
@@
ops.proc_info = proc_info_func;

@ rule2 @
identifier rule1.proc_info_func;
identifier buffer, start, offset, inout, hostno;
identifier hostptr;
@@

  proc_info_func (
+   struct Scsi_Host * hostptr,
    char * buffer, char ** start, off_t offset,
-   int hostno,
    int inout) { ... }
```
  其中 `rule1.proc_info_func` 就是继承 (后面的 rule 想继承前面的 metavar, 需要重新显式声明一遍).
  `rule1` 里面的变换模式没有 + 和 -, 只有一个上下文用来施加限制.
  要是没有这个限制, 任何一个函数, 只要参数数目类型和 `rule2` 匹配, 就会按照 `rule2` 被变换.
  然而我们希望, 只有当这个函数是某一个 `SHT` 的 `proc_info` 域的时候才做这个变换.

* `...` 匹配任何代码片段, 甚至可以 `- if (!hostptr) { ... return ...; }`, 但是似乎只能匹配 statement.
`<... ...>` 还匹配行内表达式.
  不过 [Coccinelle 的文档](http://coccinelle.lip6.fr/docs/main_grammar004.html) 好像有什么控制流的不同解释?

## SCSI 最终例子
```
@ rule1 @
struct SHT ops;
identifier proc_info_func;
@@
  ops.proc_info = proc_info_func;

@ rule2 @
identifier rule1.proc_info_func;
identifier buffer, start, offset, inout, hostno;
identifier hostptr;
@@
  proc_info_func (
+     struct Scsi_Host * hostptr,
      char * buffer, char ** start, off_t offset,
-     int hostno,
      int inout) {
    ...
-   struct Scsi_Host * hostptr;
    ...
-   hostptr = scsi_host_hn_get(hostno);
    ...
?-  if (!hostptr) { ... return ...; }
    ...
?-  scsi_host_put(hostptr);
    ...
  }

@ rule3 @
identifier rule1.proc_info_func;
identifier rule2.hostno;
identifier rule2.hostptr;
@@
  proc_info_func(...) {
    <...
-   hostno
+   hostptr->host_no
    ...>
  }

@ rule4 @
identifier rule1.proc_info_func;
identifier func;
expression buffer, start, offset, inout, hostno;
identifier hostptr;
@@
  func(..., struct Scsi_Host * hostptr, ...) {
    <...
    proc_info_func(
+   hostptr,
    buffer, start, offset,,
-   hostno,
    inout)
    ...>
  }
```

# 省略号的区别
*TODO*: `...` 和 `<... ...>` 具体的区别? 用例子表达.
