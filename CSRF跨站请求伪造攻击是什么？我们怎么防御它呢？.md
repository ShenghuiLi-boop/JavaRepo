## 复习一下 cookie 和 session以及服务器是怎么用它们验证用户登录的？
cookie数据存放在客户的浏览器（客户端）上，session数据放在服务器上，但是服务端的session的实现对客户端的cookie有依赖关系的，这个关系通过sessionID来维护：
- 假设我们的浏览器中已经保存对应某个网站的cookie了；
- 我们打开浏览器，在地址栏中输入该网站对应的网址，回车；
- 发起get请求，这个请求中包含该网站的对应的cookie；该cookie中包含验证信息，所以不用输入账号和密码便可以直接登录。
- 服务器内存创建一个session，每一个session有对应的sessionID。这个 sessionID 在返回响应的时候，被发送到客户端（通过什么变量传递回来的，我不知道），**`被客户端保存在cookie中`**。
- 此时，当我们在该网站的不同的网页中左右横跳的时候，通过跟随request中的cookie中的 sessionID ，使用相同的session。
- 当过了一段时间后，，服务器中对应的session的生命周期过去，session中的内容写入数据库，该session被清除。
- 此时再访问页面，**`会使用cookie重新登录，服务器重新创建session，重复上述过程`**。

## CSRF概念 & 攻击原理及流程
> <font size=3 color=blue>**CSRF跨站点请求伪造(Cross—Site Request Forgery)，跟XSS攻击一样，存在巨大的危害性，我们可以这样来理解：**</font>
       
攻击者盗用了用户的身份，以用户的名义发送恶意请求，对服务器来说这个请求是完全合法的，但是却完成了攻击者所期望的一个操作，比如以用户的名义发送邮件、发消息，盗取你的账号，添加系统管理员，甚至于购买商品、虚拟货币转账等。 

> <font size=3 color=blue>**如下：其中Web A为存在CSRF漏洞的网站，Web B为攻击者构建的恶意网站，User C为Web A网站的合法用户。**</font>

- **第一步**：用户C打开浏览器，访问受信任网站A，输入用户名和密码请求登录网站A；
- **第二步**：在用户信息通过验证后，网站A产生Cookie信息并返回给浏览器，此时用户登录网站A成功，可以正常发送请求到网站A；
- **第三步**：<font size=3 color=orange>**用户未退出网站A之前，在同一浏览器中，打开一个TAB页 访问 网站B**</font>；
- **第四步**：网站B接收到用户请求后，**`返回一些攻击性代码，并发出一个请求要求访问第三方站点A`**；
- **第五步**：浏览器在接收到这些攻击性代码后：
	- **用户相当于无意中在自己的 `浏览器` 上执行了网站B返回的攻击代码中的http请求**，在用户不知情的情况下携带Cookie信息，向网站A发出请求；
	- 网站A并不知道该请求其实是由B发起的，所以会根据用户C的Cookie信息以C的权限处理该请求，导致来自网站B的恶意代码被执行。 
## CSRF攻击实例分析
> <font size=3 color=blue>**假设这样一个场景：用户A想给用户B转账1000元，但是用户C想盗取用户A的这1000元。**</font>

<font size=3 color=orange>**A登录银行账户后，给B转账100元，银行后台会校验登录态skey, 最后转账成功，http请求如下(本文全部用http来描述)：**</font>

```java
http://www.bank.com/transfer.php?from=A&money=100&to=B
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/5214814f8a26490a9f1f5cead4a61797.png?x-oss-process=image,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5LiA5Liq5bCP56CB5Yac55qE6L-b6Zi25LmL5peF,size_20,color_FFFFFF,t_70,g_se,x_16)
<font size=3 color=orange>**C想通过恶意操作盗取A的钱，于是C在自己的浏览器中执行：**</font>

```java
http://www.bank.com/transfer.php?from=A&money=100&to=C
```
**这个请求会失败，因为C的浏览器中，没有A的登录态skey,  所以无法把A的钱转出来：**
![在这里插入图片描述](https://img-blog.csdnimg.cn/4dc28196a0124c9a95811f805cf4943a.png?x-oss-process=image,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5LiA5Liq5bCP56CB5Yac55qE6L-b6Zi25LmL5peF,size_20,color_FFFFFF,t_70,g_se,x_16)
<font size=3 color=orange>**C通过让A点击自己设计的恶意链接，从而让A在A自己的浏览器中执行：**</font>

```java
http://www.bank.com/transfer.php?from=A&money=100&to=C
```
**此时，如果A点击链接，会携带A的登录态skey， 于是C就盗走了A的100元:**
![在这里插入图片描述](https://img-blog.csdnimg.cn/13188a08ae6e44dea6466781b6a4dc82.png?x-oss-process=image,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5LiA5Liq5bCP56CB5Yac55qE6L-b6Zi25LmL5peF,size_20,color_FFFFFF,t_70,g_se,x_16)
## 如何防御CSRF攻击？
目前防御 CSRF 攻击主要有三种策略：**`验证 HTTP Referer 字段、在请求地址中添加 CSRF Token 并验证、在 HTTP 头中自定义属性并验证`**。
### 方法一：验证 HTTP Referer 字段
> <font size=3 color=blue>**验证 HTTP Referer 字段 原理分析**</font>

根据 HTTP 协议，在 HTTP 头中有一个字段叫 Referer，<font size=3 color=orange>**它记录了该 HTTP 请求的来源地址。**</font>

通常情况下，<font size=3 color=red>**访问一个安全受限页面的请求来自于同一个网站**</font>，比如需要访问：
```java
http://www.bank.com/transfer.php?from=A&money=100&to=B
```
用户必须先登陆 www.bank.com，然后通过点击页面上的按钮来触发转账事件。这时，<font size=3 color=red>**该转帐请求的 Referer 值就会是转账按钮所在的页面的 URL，通常是以 www.bank.com 域名开头的地址。**</font>

而如果黑客要对银行网站实施 CSRF 攻击，他只能在他自己的网站构造请求，当用户通过黑客的网站发送请求到银行时，**`该请求的 Referer 是指向黑客自己的网站`**。

因此，要防御 CSRF 攻击，银行网站 **`只需要对于每一个转账请求验证其 Referer 值`**：
- 如果是 **`以 www.bank.com 开头的域名`**，则 **`说明该请求是来自银行网站自己的请求，是合法的`**；
- 如果 Referer 是其他网站的话，则有可能是黑客的 CSRF 攻击，拒绝该请求。

这种方法的显而易见的好处就是简单易行，网站的普通开发人员不需要操心 CSRF 的漏洞，<font size=3 color=orange>**只需要在最后给所有安全敏感的请求统一增加一个拦截器来检查 Referer 的值就可以**</font>。特别是对于当前现有的系统，不需要改变当前系统的任何已有代码和逻辑，没有风险，非常便捷。
> <font size=3 color=blue>**验证 HTTP Referer 字段 缺点分析**</font>

 然而，这种方法并非万无一失。Referer 的值是由浏览器提供的，虽然 HTTP 协议上有明确的要求，但是每个浏览器对于 Referer 的具体实现可能有差别，**`并不能保证浏览器自身没有安全漏洞`**。**`使用验证 Referer 值的方法，就是把安全性都依赖于第三方（即浏览器）来保障`**，从理论上来讲，这样并不安全。
- 事实上，对于某些浏览器，比如 IE6 或 FF2，目前已经有一些方法可以篡改 Referer 值。
- 如果 www.bank.com  网站支持 IE6 浏览器，黑客完全可以把用户浏览器的 Referer 值设为以 www.bank.com  域名开头的地址，这样就可以通过验证，从而进行 CSRF 攻击。

即便是使用最新的浏览器，黑客无法篡改 Referer 值，这种方法仍然有问题。
- **`因为 Referer 值会记录下用户的访问来源，有些用户认为这样会侵犯到他们自己的隐私权`**，特别是有些组织担心 Referer 值会把组织内网中的某些信息泄露到外网中。
- 因此，<font size=4 color=red>**用户自己可以设置浏览器使其在发送请求时不再提供 Referer**</font>。当他们正常访问银行网站时，**`网站会因为请求没有 Referer 值而认为是 CSRF 攻击，拒绝合法用户的访问`**。

### 方法二：在请求地址中添加 CSRF Token 并验证

> <font size=3 color=blue>**在请求地址中添加 CSRF Token 并验证 原理分析**</font>

CSRF 攻击之所以能够成功，是因为黑客可以完全伪造用户的请求，该请求中所有的用户验证信息都是存在于 cookie 中，因此黑客可以在不知道这些验证信息的情况下直接利用用户自己的 cookie 来通过安全验证。

要抵御 CSRF，<font size=4 color=red>**关键在于在请求中放入黑客所不能伪造的信息，并且该信息不存在于 cookie 之中**</font>。

<font size=4 color=orange>**可以在 HTTP 请求中以参数的形式加入一个随机产生的 token，并在服务器端建立一个拦截器来验证这个 token，如果请求中没有 token 或者 token 内容不正确，则认为可能是 CSRF 攻击而拒绝该请求**</font>。

这种方法要比检查 Referer 要安全一些，token 可以在用户登陆后产生并放于 session 之中，然后在每次请求时把 token 从 session 中拿出，与请求中的 token 进行比对，<font size=4 color=orange>**但这种方法的难点在于如何把 token 以参数的形式加入请求**</font>：
 - 对于 GET 请求，token 将附在请求地址之后，这样 URL 就变成 `http://url?csrftoken=tokenvalue`；
 - 对于 POST 请求来说，要在 form 的最后加上 `<input type=”hidden” name=”csrftoken” value=”tokenvalue”/>`，这样就把 token 以参数的形式加入请求了。
 
> <font size=3 color=blue>**在请求地址中添加 CSRF token 并验证 缺点分析**</font>

 在一个网站中，可以接受请求的地方非常多，要对于每一个请求都加上 token 是很麻烦的，并且很容易漏掉，通常使用的方法就是在每次页面加载时，使用 javascript 遍历整个 dom 树，对于 dom 中所有的 a 和 form 标签后加入 token。**`这样可以解决大部分的请求，但是对于在页面加载之后动态生成的 html 代码，这种方法就没有作用，还需要程序员在编码时手动添加 token`**。
 
该方法还有一个缺点是 **`难以保证 token 本身的安全`**。特别是在一些论坛之类支持用户自己发表内容的网站，黑客可以在上面发布自己个人网站的地址。由于系统也会在这个地址后面加上 token，黑客可以在自己的网站上得到这个 token，并马上就可以发动 CSRF 攻击。为了避免这一点，系统可以在添加 token 的时候增加一个判断
- 如果这个链接是链到自己本站的，就在后面添加 token，如果是通向外网则不加；
- 不过，即使这个 csrftoken 不以参数的形式附加在请求之中，**`黑客的网站也同样可以通过 Referer 来得到这个 token 值`** 以发动 CSRF 攻击。**`这也是一些用户喜欢手动关闭浏览器 Referer 功能的原因`**。
### 方法三：在 HTTP 头中自定义属性并验证

> <font size=3 color=blue>**在 HTTP 头中自定义属性并验证 原理分析**</font>

这种方法也是使用 token 并进行验证，和上一种方法不同的是，这 **`里并不是把 token 以参数的形式置于 HTTP 请求之中`**，<font size=3 color=orange>**而是把它放到 HTTP 头中自定义的属性里**</font>。
- 通过 **`XMLHttpRequest`** 这个类，可以一次性给所有该类请求加上 csrftoken 这个 HTTP 头属性，并把 token 值放入其中。这样 **`解决了上种方法在请求中加入 token 的不便`**；
- 同时，通过 **`XMLHttpRequest`** <font size=3 color=orange>**请求的地址不会被记录到浏览器的地址栏，也不用担心 token 会透过 Referer 泄露到其他网站中去**</font>，。

> <font size=3 color=blue>**在 HTTP 头中自定义属性并验证 缺点分析**</font>

这种方法的局限性非常大。
- XMLHttpRequest 请求 **`通常用于 Ajax 方法中对于页面局部的异步刷新`**，**`并非所有的请求都适合用这个类来发起`**；-
- 而且 **`通过该类请求得到的页面不能被浏览器所记录下，从而进行前进，后退，刷新，收藏等操作`**，给用户带来不便；
- 另外，对于没有进行 CSRF 防护的遗留系统来说，要采用这种方法来进行防护，要 **`把所有请求都改为 XMLHttpRequest 请求，这样几乎是要重写整个网站`**，这代价无疑是不能接受的。
## CSRF漏洞检测
检测CSRF漏洞是一项比较繁琐的工作，最简单的方法就是抓取一个正常请求的数据包，去 **`掉Referer字段后再重新提交，如果该提交还有效`**，那么基本上可以确定存在CSRF漏洞。

随着对CSRF漏洞研究的不断深入，不断涌现出一些专门针对CSRF漏洞进行检测的工具，如CSRFTester，CSRF Request Builder等。

以CSRFTester工具为例，CSRF漏洞检测工具的测试原理如下：使用CSRFTester进行测试时，首先需要抓取我们在浏览器中访问过的所有链接以及所有的表单等信息，然后通过在CSRFTester中修改相应的表单等信息，重新提交，这相当于一次伪造客户端请求。如果修改后的测试请求成功被网站服务器接受，则说明存在CSRF漏洞，当然此款工具也可以被用来进行CSRF攻击。
## 最后补充一下 登录CSRF
通过前面的讲解我们知道防守CSRF也不难，页面显示表单时，给每台设备分配不同的随机码，而处理表单提交的时候，需要验证随机码是否一致。而随机码跟设备是绑定的，攻击者不可能猜的出来受害者的设备的随机码是什么，从而防止了 CSRF 攻击。而这个随机码，就叫 CSRF Token。

下面说说 <font size=4 color=blue>**登录CSRF**</font>，登录CSRF指的是用户在不知情的情况下登录攻击者的账号，而CSRF 存在的意义是为了让受害者在不知情的情况下做一些对攻击者有意义的『操作』，那么登录攻击者的账号有什么意义哪？我们下面以 社交网站 为例：
- 攻击者在社交网站创建一个帐号；
- 通过登录 CSRF 让被攻击的用户在不知情的情况下用攻击者的社交账号登录社交网站；
- 不明真相的受害者如果不注意，还以为是自己的社交网站账户，创建了相册，并且上传了一些非公开的照片；
- 攻击者再登录此账户，就可以随意浏览受害者刚上传的非公开照片了。

**虽然不如普通 CSRF 造成的危害直接，不过对于注重用户隐私的网站，是没有理由不防此种攻击的。**
