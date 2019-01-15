# WebAssemblys 是什么?

WebAssembly(wasm)是一个简单的机器模型和可执行格式[广泛的规范][extensive specification]。设计成便携，紧凑，并以原生速度或接近原始速度执行。

作为一种编程语言，WebAssembly 由两种表示相同结构的格式组成，尽管方式不同:

- 1.  该`.wat`文字格式(称为`wat`为"**W**eb**A**ssembly **T**ext")使用[S 表达式][s-expressions]。与 Lisp 语言系列有一些相似之处。比如 Scheme 和 Clojure。

- 2.  该`.wasm`二进制格式是较低级别的，旨在由 wasm 虚拟机直接使用。它在概念上类似于 ELF 和 Mach-O。

作为参考，这里是一个阶乘函数`wat`:

```
(module
  (func $fac (param f64) (result f64)
    get_local 0
    f64.const 1
    f64.lt
    if (result f64)
      f64.const 1
    else
      get_local 0
      get_local 0
      f64.const 1
      f64.sub
      call $fac
      f64.mul
    end)
  (export "fac" (func $fac)))
```

如果你对`wasm`文件好奇，你可以使用[wat2wasm demo]看上面的代码.

## 线性内存

`WebAssembly`有一个非常简单的[内存模型](https://`WebAssembly`.github.io/spec/core/syntax/modules.html#syntax-mem)。 目前，一个`wasm`模块可以访问单个"线性内存"，它本质上是一个固定数字类型的普通数组。 这个[内存成长性](https://`WebAssembly`.github.io/spec/core/syntax/instructions.html#syntax-instr-memory)是页面大小 (64K) 的倍数，并且不能缩小。

## WebAssembly 是否仅给到 Web?

虽然它目前在 JavaScript 和 Web 社区 中受到关注， 但并没有限制它的运行环境。 因此，认为在不久的将来， *wasm*将可能成为在各种环境中， 重要的"便携式可执行"格式 (我们将花一些时间仔细研究一下*wasm*便携性功能，待本书进一步说明)。

_今时今日_，总得来说，*wasm*主要与 JavaScript 有关，它有很多种类 (包括浏览器和 `Node.js`) 。 由于 JS 广泛且易于访问， 我们将主要关注使用这些平台来运行 Rust 生成的*wasm*，但 其他语言的编译 可能会在不久的将来发布。

[memory model]: https://webassembly.github.io/spec/core/syntax/modules.html#syntax-mem
[memory can be grown]: https://webassembly.github.io/spec/core/syntax/instructions.html#syntax-instr-memory
[extensive specification]: https://webassembly.github.io/spec/
[value types]: https://webassembly.github.io/spec/core/syntax/types.html#value-types
[node.js]: https://nodejs.org
[s-expressions]: https://en.wikipedia.org/wiki/S-expression
[wat2wasm demo]: https://cdn.rawgit.com/WebAssembly/wabt/aae5a4b7/demo/wat2wasm/
