
# 你应该知道的工具

这是在执行Rust和WebAssembly开发时,应该了解的精选工具列表. 

## 开发,构建和工作流程编排

### `wasm-pack`\|[github](https://github.com/rustwasm/wasm-pack)

`wasm-pack`欲成为构建和使用Rust生成的WebAssembly的一站式商店,这样你可以通过Web或Node.js与JavaScript进行相互操作. `wasm-pack`帮助您构建和发布Rust生成的WebAssembly到npm注册表,以便与您在工作流中已经使用的其他JavaScript包一起使用. 

## 优化和操作`.wasm`二进制

### `wasm-opt`\|[github](https://github.com/WebAssembly/binaryen)

该`wasm-opt`工具将WebAssembly作为输入读取,在其上运行 转换,优化 和/或 检测,然后将转换后的WebAssembly作为输出发出. `rustc`会让它,与`.wasm`LLVM 合作生成二进制文件,通常这使得创造的`.wasm`二进制文件既小又执行得更快. 这个工具是`binaryen`项目的其中一部分. 

### `wasm2asm`\|[github](https://github.com/WebAssembly/binaryen)

该`wasm2asm`工具将WebAssembly编译为"大多数 asm.js". 这非常适合支持没有WebAssembly实现的浏览器,例如Internet Explorer 11.此工具是`binaryen`项目的其中一部分. 

> 注意: 计划将此工具重命名为`wasm2js`但是,在撰写本文时,重命名仍未发生. 

### `wasm-gc`\|[github](https://github.com/alexcrichton/wasm-gc)

垃圾收集WebAssembly模块并删除所有不需要的导出,导入,函数等的小工具. 这实际上是一个WebAssembly的链接器标志`--gc-sections`. 

您通常不需要自己使用此工具,原因有两个: 

1.  `rustc`现在有一个`lld`足够新的版本,它支持`--gc-sections`WebAssembly的标志. LTO构建会自动启用此功能. 
2.  该`wasm-bindgen`CLI工具为你自动运行`wasm-gc`. 

### `wasm-snip`\|[github](https://github.com/rustwasm/wasm-snip)

`wasm-snip`替换WebAssembly函数的主体,通过用一个`unreachable`指令. 

也许您知道某些函数永远不会在运行时调用,但编译器无法在编译时证明这一点? 剪断它!然后再次运行`wasm-gc`,它传递调用的所有函数 (也可能永远不会在运行时调用) 也将被删除. 

这对于在非调试的生产版本中,强制删除Rust的恐慌基础结构非常有用. 

## 检查`.wasm`二进制

### `twiggy`\|[github](https://github.com/rustwasm/twiggy)

`twiggy`是一个对`.wasm`二进制文件代码大小分析器. 它分析二进制的调用图来回答如下问题: 

-   为什么这个函数首先包含在二进制文件中? 即哪些导出的函数是可传递的呢?
-   这个函数的保留大小是多少? 即如果删除它以及删除后成为死代码的所有函数,将节省多少空间. 

使用`twiggy`让你的二进制文件变得苗条!

### `wasm-objdump`\|[github](https://github.com/WebAssembly/wabt)

打印关于`.wasm`二进制及其每个部分的基本详细信息. 还支持反汇编成 WAT文本格式. 就像是`objdump`,不同的是为WebAssembly服务的. 这是WABT项目的一部分. 

### `wasm-nm`\|[github](https://github.com/fitzgen/wasm-nm)

列出`.wasm`二进制文件中定义的导入,导出和私有函数符号. 就像是`nm`,不同的是为WebAssembly服务的. 
