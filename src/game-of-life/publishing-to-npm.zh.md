# 发布到 npm

现在我们有一个运行中的，快速，*和*小尺寸的`wasm-game-of-life`包，我们可以将它发布到 npm，以便其他 JavaScript 开发人员可以重用它，若是他们需要现成的 Game of Life 实现。

## 准备

第一，[确保你拥有一个 npm 账号](https://www.npmjs.com/signup)。

其次，通过运行此命令，确保您在本地登录到您的帐户:

```
wasm-pack login
```

## 发布吧

确保`wasm-game-of-life/pkg`为最新版本，通过在`wasm-game-of-life`目录运行`wasm-pack`构建:

```
wasm-pack build
```

花点时间查看一下`wasm-game-of-life/pkg`现在的内容，这就是我们下一步要发布到 npm 的内容!

当你准备好了，运行`wasm-pack publish`，就可以将包上传到 npm:

```
wasm-pack publish
```

这就是发布到 npm 所需要的一切!

...除非，其他人也在做这个教程，所以`wasm-game-of-life`名字会在 npm 重叠，以致于最后一个命令不起作用。

打开`wasm-game-of-life/Cargo.toml`，并将您的用户名添加到`name`，用个人的方式消除包歧义:

```toml
[package]
name = "wasm-game-of-life-my-username"
```

然后,重新生成,并再次发布:

```
wasm-pack build
wasm-pack publish
```

这次,它应该工作啦!
