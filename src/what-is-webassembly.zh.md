# What is WebAssembly?

WebAssembly(wasm)是一个简单的机器模型和可执行格式[广泛的规范].它设计为便携,紧凑,并以原生速度或接近原始速度执行.

作为一种编程语言,WebAssembly由两种表示相同结构的格式组成,尽管方式不同:

1.  该`.wat`文字格式(称为`wat`为"**w ^**eb**一个**ssembly**Ť**ext")使用[S表达式-],与Lisp语言系列有一些相似之处,比如Scheme和Clojure.

2.  该`.wasm`二进制格式是较低级别的,旨在由wasm虚拟机直接使用.它在概念上类似于ELF和Mach-O.

作为参考,这里是一个阶乘函数`wat`:

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

如果你对如何做好奇心`wasm`文件看起来像你可以使用[wat2wasm演示]用上面的代码.

## Linear Memory

WebAssembly有一个非常简单的[记忆模型].wasm模块可以访问单个"线性存储器",它本质上是一个字节的平面数组.这个[记忆力可以增长]通过页面大小的倍数(64K).它不能缩水.

## Is WebAssembly Just for the Web?

虽然它目前在JavaScript和Web社区中受到关注,但是wasm并没有对其主机环境做出任何假设.因此,推测ism将成为将来在各种环境中使用的"可移植可执行"格式是有意义的.作为*今天*然而,wasm主要与JavaScript(JS)有关,它有很多种类(包括Web和Web)[Node.js的]).

[memory model]: https://webassembly.github.io/spec/core/syntax/modules.html#syntax-mem

[memory can be grown]: https://webassembly.github.io/spec/core/syntax/instructions.html#syntax-instr-memory

[extensive specification]: https://webassembly.github.io/spec/

[value types]: https://webassembly.github.io/spec/core/syntax/types.html#value-types

[node.js]: https://nodejs.org

[s-expressions]: https://en.wikipedia.org/wiki/S-expression

[wat2wasm demo]: https://cdn.rawgit.com/WebAssembly/wabt/aae5a4b7/demo/wat2wasm/
