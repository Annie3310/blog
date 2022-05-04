## 什么是 OAuth
> OAuth (Open Authorization[1][2]) is an open standard for access delegation, commonly used as a way for Internet users to grant websites or applications access to their information on other websites but without giving them the passwords.[3][4] This mechanism is used by companies such as Amazon,[5] Google, Facebook, Microsoft and Twitter to permit the users to share information about their accounts with third-party applications or websites.

> OAuth (Open Authorization) 是一个开放授权的开放标准, 通常作为一种在网络上不提供密码, 但允许其他网站或应用访问他们的信息的方式

试想象, 如果 A 网站需要访问你的微信头像, 而访问头像却需要你提供你的微信密码, 由 A 网站去登录你的微信去获取, 这显然是不能接受的. 
![](https://annie3310.coding.net/p/img-bucket/d/image-bucket/git/raw/master/Without-oauth.png?download=false)
于是 OAuth 应运而生.
**注意 OAuth 是一种标准, 而不是具体的实现**

## OAuth 的流程
这里以 GitHub OAuth 举例
![](https://annie3310.coding.net/p/img-bucket/d/image-bucket/git/raw/master/OAuth2.0Flow.svg?download=false)
## 实现
### 创建 GitHub OAuth App
#### 1 创建 App 并获取 Client ID
访问该地址来创建一个 GitHub OAuth App: [https://github.com/settings/applications/new](https://github.com/settings/applications/new)
![](https://annie3310.coding.net/p/img-bucket/d/image-bucket/git/raw/master/OAuthAppRegistration.png?download=false)
#### 2 获取 Client Secret
点击 `generate a new app secret` 来创建一个 `client secret`, 它用于请求 `access token`, 记住该 `secret`, 你再也无法看到它, 如果忘记只能重新生成.
![](https://annie3310.coding.net/p/img-bucket/d/image-bucket/git/raw/master/GenerateASecret.png?download=false)
### GitHub OAuth 流程
#### 1 请求用户授权
申请好 App 之后, 就可以使用了, 首先需要访问
```
GET https://github.com/login/oauth/authorize
```
该请求需要携带以下参数
|参数|说明|
|:---:|:---:|
|client_id|**必填** 客户端唯一标识, 与 GitHub App 唯一对应, 可以在设置中找到|
|redirect_uri|在用户确认授权后, 响应返回的地址, 如果不填, 则返回到 App 设置中默认的地址|
|scope|该授权可以访问的信息, 默认只可以访问 `user`, `repo`, 如果还想要访问其他的内容, 参照 [官方文档](https://docs.github.com/en/developers/apps/building-oauth-apps/scopes-for-oauth-apps) |
|state|用于标识该次请求, 最好是一串随机字符串, 用于防跨站请求伪造攻击, 这里会在下面介绍|

**还有两个不常用的参数, 一般不填, 如果有需要可以参考下方官方文档**
#### 2 请求授权码
有了授权码就可以访问网站信息了.
上一步中的请求在用户确认授权后, GitHub 会向 `redirect_uri` 中传入的地址发起一次请求, 并附带一个参数 `code`, 该参数是一串随机字符串, 用于 GitHub 认证服务器确认身份, 也是下一步请求的参数.
如果上一步中请求携带了 `state` 参数, GitHub 的请求也会将此参数原封不动的带回, 这样就可以确定 GitHub 的这次请求是来自于用户的授权请求, 而不是第三方在得知 Client ID 后伪造的请求. 如果没有 `state` 参数或再次请求的值不匹配, 则可以确定该次请求来自第三方伪造.
在拿到 `code` 参数之后, 需要发起如下请求
```
POST https://github.com/login/oauth/access_token
```
该请求需要携带以下参数
|参数|说明|
|:---:|:---:|
|client_id|**必填** 同上|
|client_secret|**必填** OAuth 设置中申请的 secret|
|code|**必填** GitHub 请求中的参数|
|redirect_uri|在发起授权后用户会被送往你的网站中的该地址|

该请求会返回如下格式的字符串
```
access_token=gho_16C7e42F292c6912E7710c838347Ae178B4a&scope=repo%2Cgist&token_type=bearer
```
如果想让该请求返回其他格式的字符串, 也可以在上一步的请求头中加入 `Accept: application/json` 或 `Accept: application/xml` 来让该请求返回 `json` 或 `xml` 格式的文本

#### 3 访问信息
上一步的请求中拿到了 `access_token`, 这就是授权码.
访问
``` 
GET https://api.github.com/user
```
该请求需要在请求头中加入 `Authorization: token OAUTH-TOKEN` 其中 `OAUTH-TOKEN` 就是上一步的 `access_token`, 该请求就会返回用户的信息, 此时, 一次 OAuth 认证就完成了.

## 总结
可以看出 OAuth 其实和 TCP 的三次握手有些相似
TCP 的三次握手中
第一次: 客户端确定了自己的发信能力, 服务端确定了自己的收信能力和客户端的发信能力
第二次: 客户端确定了自己和服务器都可以收发, 服务端确定了自己的发信, 收信能力和客户端的发信能力
第三次: 客户端和服务端都互相确定了双方的收发能力, 通信建立.
而 OAuth 本质上也是用户-网站-与资源持有方互相交换凭证确认关系的一种方式.
## 参考文档
1. [GitHub OAuth 文档](https://docs.github.com/en/developers/apps/building-oauth-apps/authorizing-oauth-apps)
2. [OAuth Wikipedia](https://en.wikipedia.org/wiki/OAuth)