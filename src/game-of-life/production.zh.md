
# 生产和部署

当我们对项目感到满意时,下一步是将其部署到生产服务器,而不是我们的开发服务器. 

第一步是运行: 

    npm run bundle

这将创建我们的Rust代码的发布版本,会使用webpack将它与我们的JavaScript和HTML捆绑在一起,并一起输出到`dist`目录. 

要从服务器使用我们的应用程序,必须正确配置[MIME类型][mime]-`application/wasm`,以正确的方式提供wasm文件. 

[mime]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types

> **注意**: 服务器配置因操作系统而异,建议您查找特定操作系统和Web服务器的教程. 这些示例假设使用 Debian/Ubuntu 或 CentOS/Red Hat/Fedora等环境. 

例如,使用nginx,[添加`application/wasm wasm;`到`/etc/nginx/mime.types`
][nginx-mime]. 然后重新加载`sudo nginx -s reload`,nginx配置更改. 

[nginx-mime]: https://nginx.org/en/docs/http/ngx_http_core_module.html#types

对于Apache[添加`AddType application/wasm .wasm`到你的apache配置的根目录][apache-mime],可能位于`/etc/apache2/apache2.conf`要么`/etc/httpd.d/conf/httpd.conf`. Apache重新加载`sudo apachectl -k graceful`. 

[apache-mime]: https://httpd.apache.org/docs/2.4/mod/mod_mime.html#addtype

最后,我们可以上传`dist`生成服务器的目录,例如通过使用SCP或SFTP客户端. 将文件复制到Web根目录 (通常是`/var/www/html`要么`/var/www`) , 现在任何人都看到最终产品了. 
