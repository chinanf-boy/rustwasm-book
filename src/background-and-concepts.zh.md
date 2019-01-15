# 背景和概念

## WebAssembly

`WebAssembly` 是一个简单的机器模型和可执行格式[extensive
specification]\(广泛的定义).

虽然它目前在 JavaScript 和 Web 社区 中受到关注， 但并没有限制它的运行环境。 因此，认为在不久的将来， *wasm*将可能成为在各种环境中， 重要的"便携式可执行"格式 (我们将花一些时间仔细研究一下*wasm*便携性功能，待本书进一步说明)。

_今时今日_，总得来说，*wasm*主要与 JavaScript 有关，它有很多种类 (包括浏览器和 `Node.js`) 。 由于 JS 广泛且易于访问， 我们将主要关注使用这些平台来运行 Rust 生成的*wasm*，但 其他语言的编译 可能会在不久的将来发布。

作为一种编程语言，`WebAssembly`由两种格式组成: 二进制格式和文本格式。 两者都代表了一种共同的结构， 尽管方式不同。 文本格式 (通常称为`wat`) 使用[S 表达式-](https://en.wikipedia.org/wiki/S-expression)，与 Clojure 或 Racket 等语言有一些相似之处。 二进制格式`wasm`是一种较低级别(底层)的格式，它本身就是由 解释器运行的汇编代码.

作为参考，这里是一个`wat`格式的阶乘函数:

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

如果你对`wasm`文件好奇，你可以使用[wat2wasm demo]看上面的代码.

`WebAssembly`有一个非常简单的[内存模型](https://`WebAssembly`.github.io/spec/core/syntax/modules.html#syntax-mem)。 目前，一个`wasm`模块可以访问单个"线性内存"，它本质上是一个固定数字类型的普通数组。 这个[内存成长性](https://`WebAssembly`.github.io/spec/core/syntax/instructions.html#syntax-instr-memory)是页面大小 (64K) 的倍数，并且不能缩小。

[extensive specification]: https://`WebAssembly`.github.io/spec/
[wat2wasm demo]: https://cdn.rawgit.com/`WebAssembly`/wabt/aae5a4b7/demo/wat2wasm/
