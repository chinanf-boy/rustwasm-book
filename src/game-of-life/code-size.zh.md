# 收缩`.wasm`尺寸

对于`.wasm`，是通过网络向客户端发送的二进制文件，如，我们的 生命游戏的 Web 应用程序，我们希望密切关注代码大小。我们的`.wasm`若是越小，我们的页面加载速度越快，用户就越快乐。

## 我们怎么通过构建配置，缩小生命游戏的`.wasm`二进制文件？

[花一点时间，来看看减小`.wasm`代码大小的构建配置选项。](../reference/code-size.zh.html#optimizing-builds-for-code-size)

使用默认版本的构建配置（没有调试符号），我们的 WebAssembly 二进制文件是 29,410 字节：

```
$ wc -c pkg/wasm_game_of_life_bg.wasm
29410 pkg/wasm_game_of_life_bg.wasm
```

启用 LTO 后，进行设置`opt-level = "z"`，并运行`wasm-opt -Oz`， 结果的`.wasm`二进制文件缩小到，只有 17,317 字节：

```
$ wc -c pkg/wasm_game_of_life_bg.wasm
17317 pkg/wasm_game_of_life_bg.wasm
```

如果我们用`gzip`压缩它（几乎每个 HTTP 服务器都会这样做）我们得到的是 9,045 字节！

```
$ gzip -9 < pkg/wasm_game_of_life_bg.wasm | wc -c
9045
```

## 演习

- 使用[该`wasm-snip`工具](../reference/code-size.zh.html#use-the-wasm-snip-tool)从我们的生命游戏中的`.wasm`二进制文件，删除恐慌基础设施函数。它节省了多少字节？

- 建立我们的生命游戏箱，有没有[`wee_alloc`作为其全局分配器](https://github.com/rustwasm/wee_alloc)。我们克隆的`rustwasm/wasm-pack-template`模板，有个“wee_alloc”Cargo 特性，您可以在`wasm-game-of-life/Cargo.toml`中的`[features]`部分添加，`default`字段，启动该特性：

  ```toml
  [features]
  default = ["wee_alloc"]
  ```

  `wee_alloc`刮掉了`.wasm`二进制文件多少尺寸？

- 我们只实例化了一个`Universe`，因此，相较于提供一个构造函数，导出控制一个`static mut`全局实例的操作会更好。如果这个全局实例也使用前面章节中讨论的双缓冲技术，我们也可以让这些缓冲区成为全局`static mut`。这将从我们的生命游戏实现中，删除所有的动态分配，还有我们做一个不包含分配器的`#![no_std]`箱子。我们完全删除分配器依赖后，又移除了`.wasm`的多少尺寸？
