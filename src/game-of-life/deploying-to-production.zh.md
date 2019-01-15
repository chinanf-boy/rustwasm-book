# Deploying our Game of Life Web Application to Production

当我们对项目感到满意时,下一步是将其部署到生产服务器而不是本地开发服务器.这与Rust和WebAssembly一样,与其他任何东西一样:将文件放在通过HTTP服务器暴露给Web的某个地方!

我们通过运行确保我们的wasm模块是最新的构建`wasm-pack`来自内部`wasm-game-of-life`目录:

```
wasm-pack build
```

然后我们通过运行捆绑我们的JavaScript和HTML`webpack`在...内`wasm-game-of-life/www`目录:

```
./node_modules/.bin/webpack
```

这捆绑了整个Web应用程序 -`.wasm`二进制文件,JavaScript文件和我们的HTML  - 并将其输出到`wasm-game-of-life/www/dist`目录.其内容应如下所示:

```
wasm-game-of-life/www/dist/
├── 0.bootstrap.js
├── 357c6c6c57e15cecdc07.module.wasm
├── bootstrap.js
└── index.html
```

要从服务器使用我们的应用程序,必须正确配置它才能提供服务`.wasm`文件正确[MIME类型][mime]-`application/wasm`.

[mime]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types

> **注意**:服务器配置因操作系统而异,建议您查找特定操作系统和Web服务器的教程.这些示例假设使用Debian / Ubuntu或CentOS / Red Hat / Fedora等环境.

例如,使用nginx,[加`application/wasm wasm;`至`/etc/nginx/mime.types`
][nginx-mime].然后重新加载nginx`sudo nginx -s reload`拿起配置更改.

[nginx-mime]: https://nginx.org/en/docs/http/ngx_http_core_module.html#types

对于Apache[加`AddType application/wasm .wasm`到你的apache配置的根目录][apache-mime],可能位于两者之一`/etc/apache2/apache2.conf`要么`/etc/httpd.d/conf/httpd.conf`.重新加载Apache`sudo apachectl -k graceful`.

[apache-mime]: https://httpd.apache.org/docs/2.4/mod/mod_mime.html#addtype

最后,我们可以上传的内容了`dist`生成服务器的目录,例如使用SCP或SFTP客户端.将文件复制到Web根目录(通常是`/var/www/html`要么`/var/www`)将允许任何人看到最终产品.
