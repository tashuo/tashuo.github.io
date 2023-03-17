---
title: OAuth授权与CSRF攻击
date: 2017-02-09 10:02:53
categories:
- 笔记
tags:
- oauth
- csrf
---

## OAuth简介
> OAuth是一个关于授权（authorization）的开放网络标准，在全世界得到广泛应用，目前的版本是2.0版。

OAuth在"客户端"与"服务提供商"之间，设置了一个授权层（authorization layer）。"客户端"不能直接登录"服务提供商"，只能登录授权层，以此将用户与客户端区分开来。"客户端"登录授权层所用的令牌（token），与用户的密码不同。用户可以在登录的时候，指定授权层令牌的权限范围和有效期。
"客户端"登录授权层以后，"服务提供商"根据令牌的权限范围和有效期，向"客户端"开放用户储存的资料。

## OAuth授权流程
![授权流程](http://ogslmd0ql.bkt.clouddn.com/bg2014051203.png "OAuth授权流程")

1. 用户打开客户端以后，客户端要求用户给予授权
2. **用户同意给予客户端授权**
3. 客户端使用上一步获得的授权，向认证服务器申请令牌
4. 认证服务器对客户端进行认证以后，确认无误，同意发放令牌
5. 客户端使用令牌，向资源服务器申请获取资源
6. 资源服务器确认令牌无误，同意向客户端开放资源

不难看出来，上面六个步骤之中，2是关键，即用户怎样才能给于客户端授权。有了这个授权以后，客户端就可以获取令牌，进而凭令牌获取资源。

## 获取授权的四种模式
客户端必须得到用户的授权（authorization grant），才能获得令牌（access token）。OAuth 2.0定义了四种授权方式。

* 授权码模式（authorization code）
* 简化模式（implicit）
* 密码模式（resource owner password credentials）
* 客户端模式（client credentials）

<!-- more -->

授权码模式（authorization code）是功能最完整、流程最严密的授权模式，也是目前大多数平台所使用的方式。它的特点就是通过客户端的后台服务器，与"服务提供商"的认证服务器进行互动。下面就只记录该模式的逻辑。

## 授权码模式（authorization code）
### 流程
![授权码流程](http://ogslmd0ql.bkt.clouddn.com/bg2014051204.png "流程")

步骤如下：

1. 用户访问客户端，后者将前者导向认证服务器。
2. 用户选择是否给予客户端授权。
3. 假设用户给予授权，认证服务器将用户导向客户端事先指定的"重定向URI"（redirection URI），同时附上一个授权码。
4. 客户端收到授权码，附上早先的"重定向URI"，向认证服务器申请令牌。这一步是在客户端的后台的服务器上完成的，对用户不可见。
5. 认证服务器核对了授权码和重定向URI，确认无误后，向客户端发送访问令牌（access token）和更新令牌（refresh token）。

### 参数
#### A: 客户端申请认证的URI
* response_type：表示授权类型，必选项，此处的值固定为"code"
* client_id：表示客户端的ID，必选项
* redirect_uri：表示重定向URI，可选项
* scope：表示申请的权限范围，可选项
* **state：表示客户端的当前状态，可以指定任意值，认证服务器会原封不动地返回这个值**

例：

    GET /authorize?response_type=code&client_id=s6BhdRkqt3&state=xyz&redirect_uri=https%3A%2F%2Fclient%2Eexample%2Ecom%2Fcb HTTP/1.1

    Host: server.example.com

#### C: 服务器回应客户端的URI
* code：表示授权码，必选项。该码的有效期应该很短，通常设为10分钟，客户端只能使用该码一次，否则会被授权服务器拒绝。该码与客户端ID和重定向URI，是一一对应关系
* **state：如果客户端的请求中包含这个参数，认证服务器的回应也必须一模一样包含这个参数**

例:

    HTTP/1.1 302 Found

    Location: https://client.example.com/cb?code=SplxlOBeZQQYbYS6WxSbIA&state=xyz

#### D: 客户端向认证服务器申请令牌的HTTP请求
* grant_type：表示使用的授权模式，必选项，此处的值固定为"authorization_code"
* code：表示上一步获得的授权码，必选项
* redirect_uri：表示重定向URI，必选项，且必须与A步骤中的该参数值保持一致
* client_id：表示客户端ID，必选项

例: 

    POST /token HTTP/1.1
    
    Host: server.example.com
    
    Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
    
    Content-Type: application/x-www-form-urlencoded
    
    
    grant_type=authorization_code&code=SplxlOBeZQQYbYS6WxSbIA
    &redirect_uri=https%3A%2F%2Fclient%2Eexample%2Ecom%2Fcb

#### E: 认证服务器发送的HTTP回复
* access_token：表示访问令牌，必选项
* token_type：表示令牌类型，该值大小写不敏感，必选项，可以是bearer类型或mac类型
* expires_in：表示过期时间，单位为秒如果省略该参数，必须其他方式设置过期时间
* refresh_token：表示更新令牌，用来获取下一次的访问令牌，可选项
* scope：表示权限范围，如果与客户端申请的范围一致，此项可省略

例: 

    HTTP/1.1 200 OK
    Content-Type: application/json;charset=UTF-8
    Cache-Control: no-store
    Pragma: no-cache

    {
      "access_token":"2YotnFZFEjr1zCsicMWpAA",
      "token_type":"example",
      "expires_in":3600,
      "refresh_token":"tGzv3JOkF0XG5Qx2TlKWIA",
      "example_parameter":"example_value"
    }

从上面代码可以看到，相关参数使用JSON格式发送（Content-Type: application/json）。此外，HTTP头信息中明确指定不得缓存。


在做 OAuth 的时候，开发者的安全意识不高的话，很可能会忽略 **state** 参数，从而导致出现 **csrf漏洞**, 出现的原因如下

### 忽略state参数导致的CSRF漏洞
#### 攻击逻辑
![攻击逻辑](http://ogslmd0ql.bkt.clouddn.com/687474703a2f2f3778727a6c6d2e636f6d312e7a302e676c622e636c6f7564646e2e636f6d2f63737266312e6a7067.jpeg "攻击逻辑")

授权码流程中的第三步`假设用户给予授权，认证服务器将用户导向客户端事先指定的"重定向URI"（redirection URI），同时附上一个授权码`， 举例来说， A用户登录 [比目官网](http://www.bmob.cn) 之后， 点击绑定 [github](https://www.github.com) 的按钮， 同意授权之后github会跳回比目链接， 链接带有授权码， 比目服务器会根据拿到的code来调取github接口获取A用户的github平台的openid， 并且将该github账号绑定至该比目账号， 这个是大略的绑定流程。

问题在于github授权完成跳回bmob的带有code的重定向URI对于bmob服务器来说是无状态的， 该code标识的只是A用户在github平台的唯一性， 所以如果A用户在拿到重定向URI之后立即中断浏览器的执行而将该链接发给B用户， 如果B用户恰好也已经登录了bmob， 那bmob服务器依然会正常执行绑定流程， 即通过该URI中A的code调用github接口获取到A的openid， 结果就是使用A的github账号绑定了B的bmob账号， 即现在A可以通过自己的github账号登录B的bmob账号，从而完成了CSRF攻击。

#### 使用state参数防止CSRF攻击
上述攻击的关键就在于bmob服务器无法判断由github重定向回的URI是否是当前登录用户B所发起的授权，而state参数就是为此服务，即通过state参数来判断code是否合法。

具体做法是bmob在授权码流程第一步中生成一个 **唯一的随机数(state)** 存入当前用户的session中，并传给github接口，回调时该state参数会随code一起返回给bmob服务器，bmob服务器则可以通过github回调传回的state参数和当前登录用户session中的state参数对比来判断是否合法。比如上面那个例子中A将回调的URI发给B，B用户打开此链接之后比目服务器会判断到用户B的session中并不存在state值或不等于回调的state值，从而阻挡了CSRF攻击。

即：**回调时对用户session中state值和回调URI中的state参数做判空操作和对比**

#### 其他
如上可知其实针对oauth做CSRF攻击条件也是比较苛刻的

* 攻击者必须了解第三方网站可能的 **绑定特性**，否则攻击极可能失败。绝大多数网站都是一个网站账号只能绑定一个OAuth提供方账号（比如微博帐号）。这种特性，导致这个漏洞在绝大多数网站根本无法快速撒网，只能定向劫持未绑定的用户到攻击者OAuth账号上，而攻击一次后，这个账号必须解绑才能用于别的攻击，导致利用难度增大不少。
* 攻击者还必须了解被定向劫持用户的上网习惯。在这个漏洞中，绝大部分都需要受害者在第三方网站上处于 **登录状态**，否则攻击基本失效。
* 攻击者还必须了解第三方网站、以及用户在该第三方网站上存在的利益。现在的攻击有许多都是带有利益的，如果是电商类的话，网站本身的金钱利益驱动可能存在，但又需要判断该用户在该网站是否存在高价值，这就增加了额外的工作量；如果只是娱乐类网站，除了言论相关和用户所拥有的网站管理权，我还想不出有什么可以吸引攻击者去定向攻击。

另一方面即说明 **利用OAuth进行CSRF攻击不用发生在第三方登录上面**，不过遵守OAuth协议规则使用state参数也是一个好的编码习惯，任何场景都有此必要。


<br/><br/><br/><br/><br/><br/><br/>
参考：
> [理解OAuth 2.0 - 阮一峰](http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html)
> 
> [OAuth2：忽略 state 参数引发的 csrf 漏洞 - pzxwhc](https://github.com/pzxwhc/MineKnowContainer/issues/68)
