## 微信OAuth2授权流程详细说明
微信 OAuth2.0 授权登录让微信用户使用微信身份安全登录第三方应用或网站，在微信用户授权登录已接入微信 OAuth2.0 的第三方应用后，第三方可以获取到用户的接口调用凭证（access_token），通过 access_token 可以进行微信开放平台授权关系接口调用，从而可实现获取微信用户基本开放信息和帮助用户实现基础开放功能等。

如果一个网站要使用微信登录，必然是要去微信公众号后台申请 appid 的，并且在申请的时候，**`还要填写一个获取 code的域名`**，而微信后台也会给你一个 **`appsecret ，appid，secret，code，域名`**，所有这一切，在整个认证流程里起了什么样的作用呢，我们下面详细来分析。

首先 OAuth 2.0 是有 4 种认证流程的：**Authorization Code、Implicit、Resource Owner Password Credentials、Client Credentials**。微信 OAuth2.0 授权登录目前支持 **`authorization_code 模式`**，适用于拥有 server 端的应用授权。
### 登录流程详细分析
认证过程如下：
- **第一步**：微信用户 请求登录 第三方应用；
- **第二步**：第三方应用携带 **`appid`** 以及 **`redirect_uri`** 通过<font size=4 color=blue>**重定向的方式**</font>请求微信OAuth2.0授权登录（最常见的就是生成一个二维码给微信用户扫描），<font size=4 color=orange>**注意这一步没有发送appsecret**</font>；
	- 注意：**`此时微信开放平台是无法确定第三方应用身份的`**，因为这时微信开放平台只有一个appid，但没有任何手段来确认 第三方应用使用的是自己的 appid；
	- 用户授权后，微信会立即发送 code 和 state(你自己设定的字段) 到 redirect_uri 中，如：
		```java
		http://test.com/test/pay?code=CODE&state=STATE
		```
- **第三步**：微信用户 在 微信开放平台 上认证身份（扫码认证），并统一授权给 第三方应用；
- **第四部**：微信用户 允许授权 第三方应用 后，微信 会 302 跳转到 第三方网站  的 redirect_uri 上，并且带上授权临时票据 **`code`**（**也就是 authorization code**）；
	- 按 OAuth2.0 的协议约定，<font size=4 color=blue>**该 code 通过浏览器的 302 重定向发送给 第三方应用，这意味着 code 值从浏览器就能看到了，非常危险**</font>。
- **第五步**：第三方应用 拿 **`code 以及 appid、appsecret`** 换取 **`accsess_token`** 和 **`openid`**；
	- 首先，这个过程是<font size=4 color=orange> **第三方应用 后台**</font> 对 <font size=4 color=orange> **微信开放平台 后台**</font> 的，<font size=4 color=orange>**不依赖浏览器，所以access_token不会像 code 那样会暴露出去**</font>。
	- 其次，**`第三方应用 需要提供自己的 appsecret`**，这样就为 微信开放平台 提供了一种验证 第三方应用 的机制。

![在这里插入图片描述](https://img-blog.csdnimg.cn/a485037116f34c37ac71c797e1f0c3c9.png?x-oss-process=image,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5LiA5Liq5bCP56CB5Yac55qE6L-b6Zi25LmL5peF,size_20,color_FFFFFF,t_70,g_se,x_16)
### 参数说明
|     参数      |                             说明                             |
| :-----------: | :----------------------------------------------------------: |
|     appid     |      应用唯一标识，在微信开放平台提交应用审核通过后获得      |
|    secret     |   应用密钥 AppSecret，在微信开放平台提交应用审核通过后获得   |
| redirect_uri  |                           回调地址                           |
|    ErrCode    | ERR_OK = 0(用户同意) ERR_AUTH_DENIED = -4（用户拒绝授权） ERR_USER_CANCEL = -2（用户取消） |
|     code      |   用户换取 access_token 的 code，仅在 ErrCode 为 0 时有效    |
| access_token  |                         接口调用凭证                         |
|    openid     |                       授权用户唯一标识                       |
| refresh_token |                    用户刷新 access_token                     |
### 获取 access_token 后可以进行哪些操作？
#### 操作一：获取用户个人信息 & UnionID 机制 和 OpenID机制 的区别
> 此接口用于获取用户个人信息。

- 开发者可通过 OpenID 来获取用户基本信息；
- 特别需要注意的是，**`如果开发者拥有多个移动应用、网站应用和公众帐号，可通过获取用户基本信息中的 unionid 来区分用户的唯一性`**，因为只要是同一个微信开放平台帐号下的移动应用、网站应用和公众帐号，用户的 unionid 是唯一的；
- 换句话说，同一用户，对同一个微信开放平台下的不同应用，unionid 是相同的。请注意，在用户修改微信头像后，旧的微信头像 URL 将会失效，因此开发者应该自己在获取用户信息后，将头像图片保存下来，避免微信头像 URL 失效后的异常情况。
> 请求说明

```java
GET https://api.weixin.qq.com/sns/userinfo?access_token=ACCESS_TOKEN&openid=OPENID
```
> 对官方说明进一步解释

做微信登录时会用到OpenId和UnionId。
- OpenID：是用户和 **`应用`** 共同生成的唯一ID。
- UnionID: 是用户和 **`应用所有者`** 共同生成的唯一ID。

举例：如公司C的微信账号同时有A, B两款应用，在做微信登录时
- 用户通过 **`A应用`** 获取的OpenID和 **`B应用`** 获取的OpenID 是 **`不同且唯一的`**。
- 如果希望 **`A应用`** 的注册用户通过微信**免注册登录** **`B应用`**, <font size=4 color=orange>**则要使用 UnionID**</font>。
- 也就是说， <font size=4 color=orange>**对于公司C账号下的所有应用，同一个微信用户获取的 UnionID 是唯一且一致的**</font>。


#### 操作二：刷新或续期 access_token 使用
> 接口说明

access_token 是调用授权关系接口的调用凭证，由于 access_token 有效期（目前为 2 个小时）较短，当 access_token 超时后，可以使用 refresh_token 进行刷新，access_token 刷新结果有两种：

1. **`若 access_token 已超时，那么进行 refresh_token 会获取一个新的 access_token，新的超时时间；`**

2. **`若 access_token 未超时，那么进行 refresh_token 不会改变 access_token，但超时时间会刷新，相当于续期 access_token。`**

refresh_token 拥有较长的有效期（30 天）且无法续期，**`当 refresh_token 失效的后，需要用户重新授权后才可以继续获取用户头像昵称。`**
> 请求方法：使用/sns/oauth2/access_token 接口获取到的 refresh_token 进行以下接口调用：

```java
GET https://api.weixin.qq.com/sns/oauth2/refresh_token?appid=APPID&grant_type=refresh_token&refresh_token=REFRESH_TOKEN
```
#### 操作三：检验授权凭证（access_token）是否有效
请求说明
```java
GET https://api.weixin.qq.com/sns/auth?access_token=ACCESS_TOKEN&openid=OPENID
```


## redirect_uri 的作用分析，以及如何防止黑客篡改redirect_uri ？
**`做过微信登录开发的都应该遇到过，获取 code 一步，如果给微信服务器传的 redirect_uri 参数值不是申请 appid 时所输入的域名，那么微信会立马返回『redirect_uri 参数错误』的提示`**。不过，为什么 OAuth 2.0 <font size=4 color=orange>**要求申请 appid 的时候必须输入一个域名**</font>，并且<font size=4 color=orange>**要求 redirect_uri 必须是此域名下的地址**</font>？

举个例子，假如我在微信申请 appid 输入的域名是 test.com，申请到的 appid 是 123456，然后假设通过微信正常获取 code 的地址是 https://example/code，那么获取 code 完整的 URL 如下：

```java
https://example/code?redirect_uri=https://test.com&appid=123456&response_type=code&scope=userinfo
```

只要把这个地址给任何微信用户发过去，他们都能通过自己的微信账号，换回 code(微信带 code 参数回跳 https://test.com/?code=abcd123)，test.com 再用 code 换回 access_token ，用户用微信账号登录 test.com 成功。

但是如果微信不验证 redirect_uri 是否是 test.com，只要有攻击者稍稍修改上面的地址，把 redirect_uri 改成他的网站（假如叫 http://hack.er），受害人访问此链接，确认登录，微信生成 code 之后再通过回调的方式，便把 code 传给了攻击者的网站：**`http://hack.er/?code=abcdef...`**。拿到 code 之后，**`攻击者再把域名换回 test.com 得到`**: **`https://chrisyue.com/?code=abcdef`**，而此时 test.com 分辨不出这是微信直接回调还是有人动了手脚的地址，无差别获取了 code 对应的 access_token，**`攻击者以受害者身份登录成功`**。

所以作为 OAuth2.0 Server，redirect_uri 的域名限制是一定要做的。而作为调用者，到不必这么担心，目前大部分遵循 OAuth2.0 的服务都不会犯这个错误。

不过对于调用者来说，此限制会影响到开发过程中的调试工作，因为微信后台对 redirect_uri 域名设置的限制，**难道开发调试就只能用外网的机器了吗**？其实这事儿也好解决，如果你的开发环境就是本地 127.0.0.1，<font size=4 color=orange>**那么直接将 redirect_uri 的域名通过 hosts 文件直接指向内网就行了**</font>：

```java
# /etc/hosts
127.0.0.1 chrisyue.com
```
## appsecret 的作用是什么？为什不能在第一步就发送appsecret呢？
> **appsecret 的作用是什么？**

如果 appid 是为了告诉身份认证服务器，『我是 test.com』，那么 secret 则是为了告诉身份认证服务器，『我真的是 test.com！』。为了告诉身份认证服务你是哪家第三方服务，你总是需要 **`暴露`** 一些信息给身份认证服务器的（**`暴露是指用户能获取到，比如会在地址栏里出现`**），获取 code 的时候，那能暴露的只能是 appid，但如果某些数据交互可以不用暴露给用户，比如获取 access_token 那一步，是由第三方服务器内部直接发起的，用户并不可见，那会带上 secret。secret 为什么叫 secret，就因为他绝对不能暴露给外网。
> **为什不能在第一步就发送appsecret呢？**

第一步我们将 appid 和 redirect_uri 是通过  **`redirect 重定向的方式`** 将它们发送给 微信认证服务器，**`是非常不安全的`**，因为我们在地址栏就能看到。
- redirect是服务器端根据逻辑代码返回一个状态码，告诉浏览器去访问最新的url地址。
```java
//1 生成微信扫描二维码
@GetMapping("login")
public String getWxCode() {
    // 微信开放平台授权baseUrl  %s相当于?代表占位符
    String baseUrl = "https://open.weixin.qq.com/connect/qrconnect" +
            "?appid=%s" +
            "&redirect_uri=%s" +
            "&response_type=code" +
            "&scope=snsapi_login" +
            "&state=%s" +
            "#wechat_redirect";

    //对redirect_url进行URLEncoder编码
    String redirectUrl = ConstantWxUtils.WX_OPEN_REDIRECT_URL;
    try {
        redirectUrl = URLEncoder.encode(redirectUrl, "utf-8");
    }catch(Exception e) {

    }

    //设置%s里面值
    String url = String.format(
                baseUrl,
                ConstantWxUtils.WX_OPEN_APP_ID,
                redirectUrl,
                "atguigu"
             );

    // 请求微信地址获取二维码
    return "redirect:"+url;
}
```

## code的作用是什么？为什么要在第四步先返回一个code呢，而不是直接返回 accsess_token 呢？
> <font size=3 color=blue>**首先我们需要知道 第三方应用是如何获取到code的，也就是微信开放平台是通过什么方式将code传给我们的？**</font>

- 这个问题在第一节中就已经提到了，**`code 是通过浏览器重定向获取的，你在浏览器地址栏就可以看到`**，如果这一步不返回code而是直接返回 access token，那么这个token其实已经暴露了；
- 而 **`第三方应用`** 拿到 code 以后换取 access_token 和 openid 是 **`第三方应用后台`** 对 **`认证服务器的访问`**，<font size=4 color=orange>**不依赖浏览器，access token不会暴露出去**</font>。（**重要 重要 重要**）
> <font size=3 color=blue>**其次如果我们在第四步直接返回access_token，那么在第一部就需要将 appsecret 提供给微信认证服务器，前面分析过了，这么做是非常不安全的。**</font>

> <font size=3 color=blue>**那么code就没有暴露的可能吗？假如被攻击者获取到了 code，类似与之前讨论 redirect_uri 是否可以不检查域名时所做的一样，直接用获取到的 code，访问 test.com 获取 access_token 的地址来登录受害人的账号，该怎么应对这种情况？**</font>

针对这个问题，微信 OAuth2.0 协议其实对此是有处理方式的：
- 首先，code 只能被使用一次；
- 其次，若是攻击者比正常用户先用了 code 也没事，<font size=4 color=orange>**因为如果同一个 code 被用了两次，之前通过此 code 获取的access token 将被撤回**</font>，而因为普通用户本来就是要访问拿 code 换 access token 的地址，code 是一定会被用的。

**也就是说，攻击者最多让正常用户有点困扰，可能会出现登录意外失败，或者明明看起来登录成功但还是获取不到用户信息的情况（access token 已经失效），但攻击者依旧拿不到数据。**
## 拿到 access_token 后，第三方应用 在后续的请求里，access_token 还是直接发送给 认证服务器 的，这个时候就不能截取了吗？
这里其实是 **`通过 HTTPS 保证安全性的`**，虽然 OAuth2.0 协议没有明文要求 OAuth 认证服务的厂家 使用 HTTPS ，但实际上 OAuth 认证服务的厂家 提供的接口都是 HTTPS 的，作为 APP 开发者可以留意一下，看看有哪个提供 OAuth 认证服务的厂家不用 HTTPS 的。
## 最容易被忽视的state字段其实很重要
state 是最容易被忽略的一个参数，因为几乎所有的 OAuth2.0 服务提供商的文档，都没有解释这个参数到底存在的目的是什么，加上本身这个参数又可以为空。但是虽然 state 参数可以为空，OAuth2.0 官方文档里标注的可是 Recommended。那官网到底因为什么要推荐使用此参数呢？

简单一句话：<font size=4 color=orange>state **参数是为了保证申请 code 的设备和使用 code 的 设备的一致 而存在的。**</font>

那么什么是『保证请求设备一致性』呢？不知大家是否了解<font size=4 color=orange>**登录 CSRF**</font>？大家又是否了解为什么建议登录的时候需要添加 CSRF 保护呢？CSRF 就是典型的保证登录请求发起设备一致性的手段。
参考：[https://blog.csdn.net/qq_36389060/article/details/124068809](https://blog.csdn.net/qq_36389060/article/details/124068809)


> <font size=4 color=blue>**用 A 来代表合作方，用 B 来代表鉴权方。在授权流程中，A 发起授权时，会向 B 传一个 state 参数，而 B 对这个参数只做一个事情，就是在 302 跳转返回的时候把它原封不动的返回。为什么要这么做呢？这背后是 登录 CSRF 攻击。**</font>

<font size=4 color=green>**与 CSRF 攻击类似，如果 state 参数为空，作为攻击者：**</font>

- 先申请一个新的，专门用于攻击他人的账号；
- 然后走正常流程，跳到微信上去登录此账号；
- 登录成功之后，微信带着 code 回跳到 test.com，这个时候，攻击者拦截自己的请求让他不再往下进行，而直接将带 code 的链接发给受害者，并欺骗受害者点击；
- 受害人点击链接之后，继续攻击者账号的登录流程，**`不知不觉登录了攻击者的账号`**。

受害者如果这个时候没察觉此账号不是他本人的，传了一些『果照』啥的，攻击者立马就能通过自己的账号看到。

<font size=4 color=green>**而 state 参数如果利用起来，当作 CSRF Token，就能避免此事的发生：**</font>
- 攻击者依旧获取 code 并打算骗受害者点击；
- 受害者点击链接，但因服务器（比如 test.com）**`分配给受害者的设备的 state 值`** 和 **`链接里面的（分配给攻击者的）state 值不一样`**，服务器（test.com）直接返回验证 state 失败.

state 或者说 CSRF Token 这种跟设备绑定的随机字符串，只要稍微复杂一点，攻击者根本就不可能猜得出来，而设置一个让攻击者猜不到的，跟设备或者说浏览器绑定的 state （CSRF token 同理） 值，就是解决 CSRF 攻击的关键。
> <font size=4 color=blue>**假如说在中间人拿到了 code 之后，控制响应不再到受害人设备，而是自己使用了 code，这种情况怎么办？**</font>

**实际上这也是典型的`『保证请求设备一致性』`的范畴**。

只要利用好 state 这个参数，攻击者是没有办法使用 code 的。<font size=4 color=orange>**当攻击者带着偷来的 code 和 state 参数的请求到第三方服务器，第三方服务器发现当前设备绑定的号根本不是 state 里面的号，直接返 403 拒绝完事儿。**</font>

> <font size=4 color=blue>**这里再重点说明一下容易搞混的点：**</font>

- state（以及登录 CSRF） 要防止的是<font size=4 color=orange>『攻击者骗受害者访问攻击者自己的账号』</font>，而不是<font size=4 color=orange>『攻击者登录受害者的账号』</font>，也不是中间人攻击的事，虽然state也可以方式中间人攻击。
- 至于为什么攻击者要骗受害者登录攻击者自己的账号。简单来说，让受害者以为登录了自己的账号做了一些操作，比如发私房照，却因实际登录账号是攻击者的，完全被攻击者尽收眼底。大家可以参考：[https://blog.csdn.net/qq_36389060/article/details/124068809](https://blog.csdn.net/qq_36389060/article/details/124068809)

<br>
<br>

参考文章：[https://www.zhihu.com/question/27446826/answer/127367856](https://www.zhihu.com/question/27446826/answer/127367856)
参考文章：[https://blog.popon.top/security-issue-about-oauth-2-0-you-should-know.html](https://blog.popon.top/security-issue-about-oauth-2-0-you-should-know.html)
参考文章：[https://www.cnblogs.com/blowing00/p/14872312.html](https://www.cnblogs.com/blowing00/p/14872312.html)


