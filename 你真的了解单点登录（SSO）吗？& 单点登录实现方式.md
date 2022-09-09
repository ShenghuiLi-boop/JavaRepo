</br>
</br>

<font size=3 color=blue>**在程序开发中，特别是网站类开发，会接触到单点登录(SSO)，什么是单点登录？单点登录(SSO)有什么用？下面就来详细介绍一下。**</font>
## 1 什么是单点登录？

单点登录的英文名叫做：**`Single Sign On（简称SSO）`**。

在以前的时候，一般我们的系统都是单系统，也就是说<font size=3 color=orange>**所有的功能都在同一个系统上**</font>。
![在这里插入图片描述](https://img-blog.csdnimg.cn/5f72be6b7bc74814904b786a43c625c6.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5LiA5Liq5bCP56CB5Yac55qE6L-b6Zi25LmL5peF,size_10,color_FFFFFF,t_70,g_se,x_16)

后来，为了合理利用资源和降低耦合性，于是把单系统拆分成多个子系统。
![在这里插入图片描述](https://img-blog.csdnimg.cn/91427c9c4af846cf8710003a3a20f584.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5LiA5Liq5bCP56CB5Yac55qE6L-b6Zi25LmL5peF,size_7,color_FFFFFF,t_70,g_se,x_16)
比如阿里系的淘宝和天猫，很明显地我们可以知道这是两个系统，但是你在使用的时候，登录了天猫，淘宝也会自动登录。
![在这里插入图片描述](https://img-blog.csdnimg.cn/11c7c87622a1434e91605451ecf6f17f.png)
简单来说，<font size=5 color=red>**单点登录就是在多个系统中，用户只需一次登录，各个系统即可感知该用户已经登录**</font>。
## 2 单系统是如何登录的
> <font size=3 color=blue>**HTTP是无状态的协议**</font>

众所周知，HTTP是无状态的协议，这意味着服务器无法确认用户的信息。于是乎，W3C就提出了：给每一个用户都发一个通行证，无论谁访问的时候都需要携带通行证，这样服务器就可以从通行证上确认用户的信息。**`这个通行证就是Cookie`**。

如果说Cookie是检查用户身上的”通行证“来确认用户的身份，那么Session就是通过检查服务器上的”客户明细表“来确认用户的身份的。Session相当于在服务器中建立了一份“客户明细表”。

HTTP协议是无状态的，Session不能依据HTTP连接来判断是否为同一个用户。于是乎：服务器向用户浏览器发送了一个名为JESSIONID的Cookie，它其实就是 sessionID。**`其实Session是依据Cookie来识别是否是同一个用户`**。

cookie数据存放在客户的浏览器（客户端）上，session数据放在服务器上，但是服务端的session的实现对客户端的cookie有依赖关系的，这个关系通过sessionID来维护。
> <font size=3 color=blue>**所以，一般我们单系统实现登录会这样做：**</font>

- 用户第一次请求服务器的时候，服务器根据用户提交的相关信息，创建对应的 Session；
- 请求返回时将此 Session 的唯一标识信息 SessionID 返回给浏览器；
- 浏览器接收到服务器返回的 SessionID 信息后，会将此信息存入到 Cookie 中，<font size=3 color=orange>**同时 Cookie 记录此 SessionID 属于哪个域名**</font>；
- 当用户第二次访问服务器的时候，请求会自动判断此域名下是否存在 Cookie 信息;
	- 如果存在自动将 Cookie 信息也发送给服务端，服务端会从 Cookie 中获取 SessionID，再根据 SessionID 查找对应的 Session 信息；
	- 如果没有找到说明用户没有登录或者登录失效，如果找到 Session 证明用户已经登录可执行后面操作。
- 此时，当我们在该网站的不同的网页中左右横跳的时候，通过跟随request中的cookie中的 sessionID ，使用相同的session。
- 当过了一段时间后，，服务器中对应的session的生命周期过去，session中的内容写入数据库，该session被清除。
- 此时再访问页面，**`会使用cookie重新登录，服务器重新创建session，重复上述过程`**。

<font size=3 color=green>**根据以上流程可知，SessionID 是连接 Cookie 和 Session 的一道桥梁，大部分系统也是根据此原理来验证用户登录状态**</font>。

> <font size=3 color=blue>**Cookie的作用是什么?和Session有什么区别？**</font>


Cookie 和 Session都是用来跟踪浏览器用户身份的会话方式，但是两者的应用场景不太一样。

Cookie 一般用来保存用户信息，比如：
- 我们在 Cookie 中保存已经登录过得用户信息，下次访问网站的时候页面可以自动帮你登录的一些基本信息给填了；
- 一般的网站都会有保持登录也就是说下次你再访问网站的时候就不需要重新登录了，这是因为用户登录的时候 **`我们可以存放了一个 Token 在 Cookie 中`**，**`下次登录的时候只需要根据 Token 值来查找用户即可(为了安全考虑，重新登录一般要将 Token 重写)`**；
- 登录一次网站后访问网站其他页面不需要重新登录。

Session 的主要作用就是通过服务端记录用户的状态。典型的场景是购物车，当你要添加商品到购物车的时候，系统不知道是哪个用户操作的，因为 HTTP 协议是无状态的。<font size=3 color=orange>**服务端给特定的用户创建特定的 Session 之后就可以标识这个用户并且跟踪这个用户了**</font>。

Cookie 数据保存在客户端(浏览器端)，Session 数据保存在服务器端。

Cookie 存储在客户端中，而Session存储在服务器上，相对来说 Session 安全性更高。如果使用 Cookie，那么最好不要将一些敏感信息不要写入 Cookie 中，最好能将 Cookie 信息加密然后使用到的时候再去服务器端解密。
## 3 多系统登录的问题与解决 之 redis + session方式
### 3.1 问题一：Session不共享问题 & 解决方案
单系统登录功能主要是用Session保存用户信息来实现的，但我们清楚的是：多系统即可能有多个Tomcat，而Session是依赖当前系统的Tomcat，所以系统A的Session和系统B的Session是不共享的。
![在这里插入图片描述](https://img-blog.csdnimg.cn/ec3fd1681a184b9684ec4d1723815e43.png)
> <font size=3 color=blue>**解决系统之间Session不共享问题有一下几种方案：**</font>

- Tomcat集群Session全局复制（集群内每个tomcat的session完全同步）【会影响集群的性能呢，不建议】
- 根据请求的IP进行Hash映射到对应的机器上（这就相当于请求的IP一直会访问同一个服务器）【如果服务器宕机了，会丢失了一大部分Session的数据，不建议】
- <font size=4 color=orange>**把Session数据放在Redis中（使用Redis模拟Session）【建议】**</font>
### 3.2 问题二：Cookie跨域的问题 & 解决方案
上面我们解决了Session不能共享的问题，但其实还有另一个问题，即<font size=3 color=orange> **Cookie是不能跨域的**</font>。

比如说，我们请求 **`http://www.test111.com/`** 时，浏览器会自动把 **`test111.com`** 的Cookie带过去给目标服务器，而不会把 **`http://www.test222.com`** 的Cookie带过去给目标服务器。

这就意味着，<font size=3 color=orange>**由于域名不同，用户向系统A登录后，系统A返回给浏览器的Cookie，用户再请求系统B的时候不会将系统A的Cookie带过去**</font>。
> <font size=3 color=blue>**针对Cookie存在跨域问题，有几种解决方案：**</font>

- <font size=3 color=orange>**服务端将Cookie写到客户端后，客户端对Cookie进行解析，将Token解析出来，此后请求都把这个Token带上就行了**</font>；
- <font size=4 color=red>**多个域名共享Cookie，在写到客户端的时候设置Cookie的domain**</font>；
- <font size=3 color=orange>**将Token保存在SessionStroage中（不依赖Cookie就没有跨域的问题了）**</font>。

### 3.3 多系统登录问题（SSO）实现
我们<font size=4 color=red>**把Session数据放在Redis中（使用Redis模拟Session）实现Session共享，并通过设置domain解决cookie跨域问题**</font> 。并且将 <font size=4 color=red>**将登录功能单独抽取出来做成一个子系统**</font>，这里我们称为SSO系统：

![在这里插入图片描述](https://img-blog.csdnimg.cn/e56982da1a854fdf8a29c0a2b5952852.png?x-oss-process=image,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5LiA5Liq5bCP56CB5Yac55qE6L-b6Zi25LmL5peF,size_9,color_FFFFFF,t_70,g_se,x_16)
> <font size=4 color=green>**SSO系统（登录系统）中的用户登录逻辑如下：**</font>
- 根据 用户名 和 密码 去数据库进行查询；
	- 首先分局用户名查询，如果未查询到则说明用户不存在；
	- 如果用户存在，则进行密码核对，如果密码正确则登录成功。
- 判断登录成功之后，<font size=4 color=orange>**生成一个Token，然后将这个token存入redis，并设置过期时间（这里Token其实就是一个UUID）**</font>。

```java
// 登录功能(SSO单独的服务)
@Override
public TaotaoResult login(String username, String password) throws Exception {
    // 1.根据用户名查询用户信息
    TbUserExample example = new TbUserExample();
    Criteria criteria = example.createCriteria();
    criteria.andUsernameEqualTo(username);
    List<TbUser> list = userMapper.selectByExample(example);
    if (null == list || list.isEmpty()) {
        return TaotaoResult.build(400, "用户不存在");
    }
    // 2.根据查询到的用户进行核对密码
    TbUser user = list.get(0);
    if (!DigestUtils.md5DigestAsHex(password.getBytes()).equals(user.getPassword())) {
        return TaotaoResult.build(400, "密码错误");
    }
    // 3.密码核对成功则登录成功
    // 4.生成一个用户token，并存入redis
    String token = UUID.randomUUID().toString();
    jedisCluster.set(USER_TOKEN_KEY + ":" + token, JsonUtils.objectToJson(user));
    // 5.设置用户token的过期时间
    jedisCluster.expire(USER_TOKEN_KEY + ":" + token, SESSION_EXPIRE_TIME);
    return TaotaoResult.ok(token);
}
```

> <font size=4 color=green>**用户登录其他子系统时，其它子系统会请求 SSO（登录系统）中的登录方法进行登录流程（就是上面的逻辑），将返回的token写到Cookie中，下次访问时HTTP请求会自动把Cookie带上。**</font>

```java
public TaotaoResult login(String username, String password,
                              HttpServletRequest request, HttpServletResponse response) {
    //请求参数
    Map<String, String> param = new HashMap<>();
    param.put("username", username);
    param.put("password", password);
    //登录处理
    String stringResult = HttpClientUtil.doPost(REGISTER_USER_URL + USER_LOGIN_URL, param);
    TaotaoResult result = TaotaoResult.format(stringResult);
    //登录出错
    if (result.getStatus() != 200) {
        return result;
    }
    //登录成功后把取token信息，并写入cookie
    String token = (String) result.getData();
    //写入cookie
    CookieUtils.setCookie(request, response, "TT_TOKEN", token);
    //返回成功
    return result;
}
```
> <font size=4 color=green>**之后用户每次发起HTTP请求时，都会自动将Cookie都会带上，后端可以通过拦截器得到token，进而判断该用户是否已经登录。**</font>

> <font size=4 color=blue>**单点登录（SSO）流程总结**</font>

- SSO系统生成一个token，并将用户信息存到Redis中，并设置过期时间；
- 其他系统请求SSO系统进行登录，得到SSO返回的token，写到Cookie中；
- <font size=4 color=orange>**每次请求时，Cookie都会带上，拦截器得到token，判断是否已经登录** </font>。

> <font size=4 color=blue>**到这里，其实我们会发现其实就两个变化：**</font>

- 将登陆功能抽取为一个系统（SSO），其他系统请求SSO进行登录；
- **`本来将用户信息存到Session，现在将用户信息存到Redis`**。

到这里，我们已经可以实现单点登录了。

## 4 CAS（Central Authentication Service）实现用户认证思路
> <font size=4 color=green>**3.3 中的SSO实现方式其实是所有微服务都在同一个域名下，但是如果两个系统（比如天猫和淘宝）域名不同，那么该怎么实现SSO呢？**</font> <font size=4 color=red>**这时候就需要 CAS（Central Authentication Service）了。**</font>

如果已经将登录单独抽取成系统出来，我们还能这样玩。现在我们现在有三个系统以及用户 A，分别是：
- 系统A **`www.cart.com`**（购物车功能）；
- 系统B **`www.order.com`**（订单系统）；
- 认证中心SSO  **`www.sso.com`**。
> <font size=5 color=blue>**用户A访问系统A流程（此时用户还未与认证中心建立全局会话）**</font>

首先，用户想要访问系统A （www.cart.com）受限的资源(比如说购物车功能，购物车功能需要登录后才能访问)，系统A （www.cart.com）发现用户并没有登录，<font size=4 color=orange>**于是重定向到sso认证中心，并将自己的地址作为参数**</font>。请求的地址如下：

```java
https://www.sso.com?service=www.cart.com
```
sso认证中心发现用户未登录，将用户引导至登录页面，用户进行输入用户名和密码进行登录，<font size=3 color=orange>**用户与认证中心建立全局会话**（**根据用户信息生成一个token，写到Cookie中，保存在浏览器上**）</font>。
![在这里插入图片描述](https://img-blog.csdnimg.cn/0b53557640e34672be98c1a24e874556.png?x-oss-process=image,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5LiA5Liq5bCP56CB5Yac55qE6L-b6Zi25LmL5peF,size_16,color_FFFFFF,t_70,g_se,x_16)

随后，认证中心<font size=4 color=orange>**重定向回系统A （www.cart.com）**</font>，<font size=4 color=red>**并把Token携带过去给系统A**</font>，重定向的地址如下：

```java
https://www.cart.com?token=xxxxxxx
```
接着，系统A去sso认证中心验证这个Token是否正确，<font size=4 color=red>**如果正确，则系统A和用户建立局部会话（创建Session）**</font>。到此，系统A和用户已经是登录状态了。
![在这里插入图片描述](https://img-blog.csdnimg.cn/120d09169c9746eb8d6ab8e93d7d1b92.png?x-oss-process=image,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5LiA5Liq5bCP56CB5Yac55qE6L-b6Zi25LmL5peF,size_20,color_FFFFFF,t_70,g_se,x_16)
> <font size=5 color=blue>**用户A登录系统B流程（此时用户A已经与认证中心建立全局会话）**</font>

此时，用户A 想要访问 系统B（www.order.com）受限的资源(比如说订单功能，订单功能需要登录后才能访问)，系统B（www.order.com）发现用户并没有登录，<font size=4 color=orange>**于是重定向到sso认证中心，并将自己的地址作为参数**</font>。请求的地址如下：
```java
https://www.sso.com?service=www.order.com
```
注意，<font size=4 color=orange>**因为之前用户与认证中心www.sso.com已经建立了全局会话（当时已经把Cookie保存到浏览器上了）**</font>，<font size=5 color=red>**所以这次系统B重定向到认证中心www.sso.com是可以带上Cookie的（重要 重要 重要）**</font>。

<font size=4 color=green>认证中心 **`根据带过来的Cookie发现已经与用户建立了全局会话了`** ，认证中心重定向回系统B，并把Token携带过去给系统B</font>，重定向的地址如下：

```java
https://www.order.com?token=xxxxxxx
```
接着，系统B去sso认证中心验证这个Token是否正确，<font size=4 color=red>**如果正确，则系统B和用户建立局部会话（创建Session）**</font>。到此，系统B和用户已经是登录状态了。
![在这里插入图片描述](https://img-blog.csdnimg.cn/45bdf26502734f0091778b8e3afb1841.png?x-oss-process=image,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5LiA5Liq5bCP56CB5Yac55qE6L-b6Zi25LmL5peF,size_20,color_FFFFFF,t_70,g_se,x_16)






<br>
<br>

<font size=5 color=blue>**其实SSO认证中心就类似一个中转站**</font>。
<br>
<br>

参考文章：[https://blog.csdn.net/cnpinpai/article/details/90669587](https://blog.csdn.net/cnpinpai/article/details/90669587)

