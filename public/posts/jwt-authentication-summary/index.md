# Jwt 认证简述


## 前言
很久以前我记得我写过 jwt 相关的东西，但是今天想翻看的时候发现不见了。既然如此就再写一次好了。
原理相关的东西，我就不写了。因为也写不好，看官网，或者看我主要参考的这篇文章[https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html)。阮一峰老师的东西都很好。值得一看。
其他的参考链接
* [https://www.jianshu.com/p/576dbf44b2ae](https://www.jianshu.com/p/576dbf44b2ae)
* [https://learnku.com/articles/28909](https://learnku.com/articles/28909)
* [https://www.ibm.com/docs/zh-tw/was-liberty/base?topic=uocpao2as-json-web-token-jwt-oauth-client-authorization-grants](https://www.ibm.com/docs/zh-tw/was-liberty/base?topic=uocpao2as-json-web-token-jwt-oauth-client-authorization-grants)
* [https://blog.csdn.net/missingwhy/article/details/88887170](https://blog.csdn.net/missingwhy/article/details/88887170)

那么我写什么么？主要说一下自己的理解。以及记录一些字段的说明。以供我后期翻看。

## 正文
### 相关字段定义。
```
* iss (issuer)：签发人
* exp (expiration time)：过期时间
* sub (subject)：主题
* aud (audience)：受众
* nbf (Not Before)：生效时间
* iat (Issued At)：签发时间
* jti (JWT ID)：编号
```
```
* iss: jwt签发者
* sub: jwt所面向的用户
* aud: 接收jwt的一方
* exp: jwt的过期时间，这个过期时间必须要大于签发时间
* nbf: 定义在什么时间之前，该jwt都是不可用的.
* iat: jwt的签发时间
* jti: jwt的唯一身份标识，主要用来作为一次性token,从而回避重放攻击。
```

可以看到，关于字段的定义，我从两面的文章复制了两份。可以帮助我们理解每个字段的作用。
上面的字段都是在 `payload` 的部分。除了上面的字段，还可以放置一些我们自己的数据。注意：别放敏感信息字段，因为数据会被解密的。敏感数据还是根据我们设置的字段，在服务端计算或获取。所以自定义的部分，我们可以防止 uid、前端需要的字段、前端权限判断字段等。但是后端也要做好相关验证工作，前端一切的东西都不可信。都要经过过滤才可以。

### 哪些是我们主要用到的字段
再说说上面这些官方定义的字段主要用的是那些，目前最重要的就是 `exp` 字段，这个字段定义了过期时间。我们根据这个字段来判断是否过期。以及是否需要续期（自动续期的我们在后面具体说）。
另一个可能需要用到的字段是 `jti`。这个字段存一个唯一的标识，上面说的说法 `主要用来作为一次性token,从而回避重放攻击`。那么具体怎么用呢，下面我们就用这两个字段具体说说我的理解。

### jwt 续期
jwt 过期的时间在 `exp` 中定义。如果超期了，就说明这个 jwt 无效了。当然验证这块也得自己实现，或者看使用的框架是否验证了这部分。所以 jwt 很自由。我们不仅要验证过期，也要考虑当这个 jwt 临期了，是否需要续期的问题。这个有助于使用体验。如果设置了时间很短，用户没经过多久就要重新登录，那么体验就会很差。当然高时效性的东西，不需要考虑我上面所说的那些。那么什么时候续期呢，我们可以定义一个阈值，当请求时间跟过期时间小于阈值了，我们就需要重新生成一个 token。返回给前端。让前端刷新 token。供给后续使用。再说说 jwt 的有效期设置多久呢，还是看项目具体需求，不过从我身边的朋友，以及我自己的使用经验来看，如果不是那种高时效性的东西，设置7天也没什么问题。

### jti 的用法 (JWT ID)
这里面保存了一个唯一的字段，每个 jwt 的 jti 的值都是不同的。可以看到这个的全称是 JWTID。可以跟 sessionId 一样理解。因为 jwt 很长，我们要在后端做一些操作的时候，存储一个短的 jwt Id 就会比保存完整的节省空间。那么怎么用呢，我们可以把这个字段用来把一些 jwt 过期。比如某个人的 jwt 被别人拿走使用了。我们通过手段判断出来后，就可以吧这个 jti 放到一个黑名单中，用户请求过来的时候，我们就可以拒绝这个 jwt 的访问。官方示例 [https://github.com/auth0/express-jwt#revoked-tokens](https://github.com/auth0/express-jwt#revoked-tokens)

## 说点其他的
网上对于 jwt 的说法有很多，有说好的有说坏的。但是就我自己来说，就看这个东西怎么用。服务端一定要做好各种防范以及兜底验证方案。保证自己的安全，当然了，我自己写的项目肯定不会太严格，毕竟就是自己用，或者用来验证一些东西。另外这个东西就是跟 session 一样的东西，鉴权是另一套东西了。都得结合起来用才能保证安全。总之不信任前端任何东西，以及合理的权限配置。我们不能保证服务足够安全，那么就要把损失降到最低。


## 总结
好了，今天的说明就到这了，上面就是我粗浅的理解，如果后续有了更多想法，我在继续补充。

---

> 作者: [M1racleHao](https://github.com/crazyhl)  
> URL: http://localhost:1313/posts/jwt-authentication-summary/  

