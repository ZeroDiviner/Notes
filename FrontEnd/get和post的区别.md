# Get和 post 的区别

1. get 的参数使用 url 后面拼接的方式，而 post 使用request body 进行传参
2. get 的参数有长度限制(浏览器限制一般为2k，而服务器限制一般为64k)，而 post 没有
3. get 的参数只能是 ASCII 形式的，而 post 没有限制。
4. get 的参数会被浏览器历史记录保留，而 post 不会
5. get 的 url 是可以被浏览器书签储存的，而 post 不可以
6. get 请求会被浏览器主动 cache，而 post 除非手动设置过，否则不会缓存
7. get 比 post 更不安全，因为参数直接暴露在 url 上



而其实 get 和 post 并没有本质上的区别，让 Get 增加 request body，让 post 的 url 上增加参数，技术上来说也是可行的。GET 和 POST 只是 HTTP 给其的一个标准，而具体实现则还是 TCP，因此 GET 方法不使用的就是TCP 的 entity body 的部分。



