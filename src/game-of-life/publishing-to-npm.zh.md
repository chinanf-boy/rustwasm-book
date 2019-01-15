# Publishing to npm

现在我们有一个工作,快速,*和*小`wasm-game-of-life`包,我们可以将它发布到npm,以便其他JavaScript开发人员可以重用它,如果他们需要现成的Game of Life实现.

## Prerequisites

第一,[make sure you have an npm account](https://www.npmjs.com/signup).

其次,通过运行此命令,确保您在本地登录到您的帐户:

```
wasm-pack login
```

## Publishing

确保`wasm-game-of-life/pkg`通过运行构建是最新的`wasm-pack`在 - 的里面`wasm-game-of-life`目录:

```
wasm-pack build
```

花点时间查看一下内容`wasm-game-of-life/pkg`现在,这就是我们下一步要发布到npm的内容!

当你准备好了,跑`wasm-pack publish`将包上传到npm:

```
wasm-pack publish
```

这就是发布到npm所需要的一切!

...除了其他人也做了这个教程,因此`wasm-game-of-life`名字取自npm,最后一个命令可能不起作用.

打开`wasm-game-of-life/Cargo.toml`并将您的用户名添加到`name`以独特的方式消除包装歧义:

```toml
[package]
name = "wasm-game-of-life-my-username"
```

然后,重新生成并再次发布:

```
wasm-pack build
wasm-pack publish
```

这次它应该工作!
