# Http 常见状态码



100 continue: options 请求一般会返回的状态码.

101 switching protocol: 一般转换至 websocket 返回的状态码

200 ok: get 或者 post 请求成功并返回了所请求的内容。

204 no content: 无内容，服务器处理成功，但是不需要返回任何信息。



301 永久重定向: 用于 http 转向 https, 或者修改文件后缀等等，例如 html 文件变成了 asp 文件，收到301请求后客户端可以缓存跳转后的地址，下次不用再次发送。

302 暂时重定向: 临时移动，客户端继续使用原来的URI

304 协商缓存

307 暂时重定向，区别是307不允许把 post 重定向到 get 上(http1.1的内容)

308 永久重定向，区别是308不允许把 post 重定向到 get 上(http1.1的内容)



400 bad request: 发送的请求里面有语法错误，服务器无法理解

401 Authorization required: 需要验证，请求未授权

403 Forbidden: 拒绝访问，通常因为服务器上文件夹或者文件的权限问题导致

404 not found: 这个资源在服务器上找不到

405 method not allowed: 请求方法(指get,post)不被服务器允许。



500 internal error服务器代码有问题

501 not implemented : 服务器不支持请求的功能

502 Bad gateway 充当网关或者代理的服务器从服务器收到一个无效请求

503 Service unavailable服务器繁忙

505 http version not supported

