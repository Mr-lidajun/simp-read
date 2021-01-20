> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [se8s0n.github.io](https://se8s0n.github.io/2018/09/11/HTTP%E7%B3%BB%E5%88%97(%E4%BA%94)/)

如果, 你伴随 HTTP 系列一路成长, 那么现在的你, 已经可以乘船开启 (学习)HTTP 安全之旅。我保证, 你会很享受这个学习过程。🙂

许多公司都是安全漏洞的受害者, 一些知名的公司, 比如 Dropbox(支持文件同步、备份、共享云存储的软件)、Linkedin(领英, 全球最大职业社交网站)、MySpace(社交网站)、Adobe、索尼和福布斯等许多公司遭受过恶意攻击。除此之外, 许多用户也被 (恶意攻击公司的事件) 危及过, 你, 很可能曾是受害者之一。🙂

你可以在 [Have I Been Pwned](https://haveibeenpwned.com/) 查询你是否是受害者之一。

我的电子邮件地址出现在 4 个不同的存在安全漏洞的网站上。

Web 应用安全涉及面广, 因此, 只有一篇文章难以涵盖所有的知识。不过, 本文可作为 (Web 应用安全的) 启蒙。首先, 让我们来了解一下, HTTP 协议如何安全地通信。

本文是 HTTP 系列的第五部分。

文章概要：

*   你真的需要 HTTPS 吗?
*   HTTPS 的基本概念
    *   SSL vs TLS
    *   TLS 握手
    *   证书 (Certificate) 及证书认证(CAs, Certification Authorities)
    *   证书链 (certificate chain)
*   HTTPS 的不足

文章涵盖了大量的内容, 不管怎样, 我们从头学起吧！

[](#你真的需要HTTPS吗 "你真的需要HTTPS吗?")你真的需要 HTTPS 吗?
---------------------------------------------

你可能认为, 不是所有的网站都需要安全防护的。如果一个网站不提供敏感信息, 也不能以任何形式上传数据, 仅仅是为了在 URL 栏上可以显示绿色的标志 (网站安全的标志), 就去购买证书和降低访问网站的速度, 这看起来像是想要杀猪却用了宰牛刀。

如果有一个属于你的网站, 你就会知道加载速度——越快越好——对网站来说是至关重要的, 因此, 你不会想给你的网站增加负担。

从 HTTP 迁移到 HTTPS 是一个痛苦的过程, 何必为了维护原本就不用被保护的网站遭受这样的痛苦呢? 你要考虑的第一件事, 就是出钱买网站的安全——因为购买证书对于一般的小公司或个人来说是一笔很大的费用。

_注: 听说已经有机构开始颁发免费证书, 详情可以从[十大免费 SSL 证书：网站免费添加 HTTPS 加密](https://blog.csdn.net/ithomer/article/details/78075006) 中了解_

现在就让我们一起来看看, (为安全) 惹上这个麻烦到底值不值得。

### [](#HTTPS给报文加密并防止中间人攻击 "HTTPS给报文加密并防止中间人攻击")HTTPS 给报文加密并防止中间人攻击

在之前的 HTTP 系列文章中, 我们谈论了不同的 HTTP 认证机制及其安全缺陷。然而, 不管是 basic 认证还是 digest 认证, **都不能阻止中间人攻击**。中间人攻击——中间人, 就是恶意介入用户与网站通信的一批人——的目的是拦截双方通信的原始报文 (如果有需要, 还可以修改报文), 通过转发拦截的报文, (这个攻击行为) 就能不被发现。  
[![](https://code-maze.com/wp-content/uploads/2017/07/MITM.png)](https://code-maze.com/wp-content/uploads/2017/07/MITM.png)  
原本参与通信的人可能不能察觉到他们的通信正在被监听。

HTTPS 通过加密通信 (报文) 防止中间人攻击, 但是这并不意味着 (双方的) 通信不会再被监听, 它仅仅阻止了那些查看和拦截 (双方通信) 报文的人看报文内容罢了。如果中间人想要将报文解密, 那么他必须要有解密的密钥。之后, 你就会知道, 这个密钥是如何起作用的。

让我们继续往下走。

### [](#HTTPS——影响搜索排名的因素 "HTTPS——影响搜索排名的因素")HTTPS——影响搜索排名的因素

不久之前 (2014 年), [Google 让 HTTPS 成为一个影响 (搜索算法) 排名的因素](https://www.wosign.com/news/Google_https.htm)。

这意味着什么呢？

这意味着, 如果你是一个关心 Google 搜索排名的网络管理员, 你的网站一定要用 HTTPS 协议。尽管 HTTPS 没有内容质量、反向链接这些因素影响力大, 但它还是很重要的。

Google 通过这项举措鼓励网站管理员尽快 (从 HTTP) 过渡到 HTTPS, 从而提高互联网整体的安全性。

### [](#证书-完全免费 "(证书)完全免费")(证书) 完全免费

要架设 HTTPS 网站, 就需要一个证书来解决证书认证的问题。从前, 证书费用高昂, 并且年年需要更新。

如今, 感谢 [Let’s encrypt](https://letsencrypt.org/) 的小伙伴们, 让我们付得起买证书的钱 (只要 0 元!)。那些证书是真的完全免费的！

Let’s encrypt 的证书安装起来很简单, 这离不开其背后大公司的支持和超棒的社区人员（的努力）。看看 Let’s encrypt 的赞助商和支持 (Let’s encrypt) 的公司的名单, 其中可能有你认识的公司。

Let’s encrypt 只提供域名认证 (DV) 证书, 不提供组织认证 (OV) 证书和扩展验证 (EV) 证书。DV 证书只能用 90 天, 不过更新起来很容易。

和其他 (好的) 技术相同, Let’s encrypt 证书也有不好的一面。正因为现在证书获取起来很容易, [钓鱼网站也开始利用起这些证书了](https://www.bleepingcomputer.com/news/security/14-766-lets-encrypt-ssl-certificates-issued-to-paypal-phishing-sites/)。

### [](#关于-传输-速度 "关于(传输)速度")关于 (传输) 速度

一提到 HTTPS, 许多人就会认为它给速度带来了了额外的开销。简单地用 [httpvshttps.com](http://httpvshttps.com/) 来测试一下。

下面是测试结果:  
！[](https://code-maze.com/wp-content/uploads/2017/07/httpvshttpsresults-1.png)  
所以这里发生了什么? 为什么 HTTPS 比 HTTP 快那么多? 这怎么可能!

HTTPS 是使用 HTTP 2.0 协议的必要条件。

如果我们查看网络选项卡, 我们可以看到在使用 HTTPS 时，图像是通过 HTTP 2.0 协议加载的, 除此之外, 流量看起来也很不一样。

HTTP 2.0 是继 HTTP/1.1 之后成功的 HTTP 协议。

在 HTTP/1.1 的基础之上, HTTP 2.0 增加了许多优点:

*   用二进制 (传输) 取代文本(传输)
*   可通过单个 TCP 连接并行发送多个请求
*   采用 [HPACK](http://www.cnblogs.com/ghj1976/p/4586529.html) 压缩报文首部减少 (流量) 开销
*   使用新的应用层协议协商 (Application-Layer Protocol Negotiation，简称 ALPN) 扩展更快地加密通信。
*   可降低 RTT(round trip times, 往返时延), 让网站加载的速度更快。
*   还有许多其他的优点

### [](#不使用HTTPS-浏览器表示拒绝。 "不使用HTTPS?浏览器表示拒绝。")不使用 HTTPS? 浏览器表示拒绝。

如果你现在还不相信——使用 HTTPS 是必要的——那么你应该知道, 现在一些浏览器已经开始举起的大旗, 反对 (传输) 那些未经加密内容。在 16 年 9 月份的时候, Google 发布博文明确地说明了 Chrome 浏览器会怎么对待那些不安全的网站。

现在, 让我们来看一下 Chrome 56 的前后变化吧!

[![](https://code-maze.com/wp-content/uploads/2017/07/https-shaming-1.png)](https://code-maze.com/wp-content/uploads/2017/07/https-shaming-1.png)

下面是最终的效果。

[![](https://code-maze.com/wp-content/uploads/2017/07/https-shaming-2.png)](https://code-maze.com/wp-content/uploads/2017/07/https-shaming-2.png)

你现在相信了吧！😀

### [](#迁移到HTTPS的过程很复杂 "迁移到HTTPS的过程很复杂")迁移到 HTTPS 的过程很复杂

(迁移过程复杂) 这是过去遗留下来的问题。因为历史悠久的网站之前用 HTTP 协议上传了大量的资源, 这让迁移至 HTTPS 对这些网站来说就更加困难了, 不过托管的平台通常都会让这个过程变得简单一点。

大部分托管平台都提供了自动往 HTTPS 迁移的方法, 点击选项栏的一个按钮就可以轻松迁移。

如果你打算搭建一个基于 HTTPS 的网站, 你需要先去确认一下, 托管平台是否有提供往 HTTPS 迁移的方法。或者, 有一个脚本, 可以通过 Let’s encrypt 和设置服务器配置让你自己解决迁移的问题。

以上, 都是要迁移到 HTTPS 的原因。有什么理由不去做呢?

文章写到这里, 希望我已经说服你, 用 HTTPS 是有必要的, 并且这可以让你有动力将网站往迁移 HTTPS 迁移, 知道 HTTPS 是如何工作的。

[](#HTTPS的基本概念 "HTTPS的基本概念")HTTPS 的基本概念
---------------------------------------

HTTPS(Hypertext Transfer Protocol Secure, 超文本传输安全协议) 准确地说, 是客户端和服务器在用 HTTP 进行通信的基础上, 增加使用安全的 SSL/TLS 的协议。

在之前的 HTTP 系列文章中, 我们已经了解过 HTTP 的工作原理了, 不过 SSL/TLS 却没有提到。SSL 和 TLS 到底是什么? 为什么要使用他们?

下面, 就让我们来了解一下。

### [](#SSL-vc-TLS "SSL vc TLS")SSL vc TLS

SSL(Secure Socket Layer, 安全套接层) 和 TLS(Transport Layer Security, 传输安全协议) 可互换使用, 但实际上, 现在提到 SSL 可能就是在说 TLS。

SSL 一开始是由网景 (Netscape) 公司提出的, 不过它的 1.0 版本早已停用。SSL 的 2.0 版本和 3.0 版本在 1995、1996 年相继现世, 这两个版本在出现之后, 即便是之后 TLS 已经开始推行了, 也沿用了很长一段时间(至少对 IT 行业来说, 时间挺长的)。直到 2014 年 [Poodle 攻击](https://www.puritys.me/zh_cn/docs-blog/article-376-HTTP-:-SSLv3-%E6%BC%8F%E6%B4%9E---Poodle-%E6%94%BB%E5%87%BB.html)发生之后才有所改变。当时服务器支持从 TLS 降级到 SSL, 这是 Poodle 攻击如此成功的主要原因。

SSL 一开始是由网景 (Netscape) 公司提出的, 不过它的 1.0 版本早已停用。SSL 的 2.0 版本和 3.0 版本在 1995、1996 年相继现世, 即便是之后 TLS 已经开始推行了, 也沿用了很长一段时间(至少对 IT 行业来说, 时间挺长的)。2014 年, 当时服务器支持从 TLS 降级到 SSL, 这是 [Poodle 攻击](https://www.puritys.me/zh_cn/docs-blog/article-376-HTTP-:-SSLv3-%E6%BC%8F%E6%B4%9E---Poodle-%E6%94%BB%E5%87%BB.html)发生之后如此成功的主要原因。

从此之后, 服务器**降级到 SSL 的功能已经完全消失了**。

如果你用 [Qualys SSL Labs tool](https://www.ssllabs.com/ssltest/) 检查你或其他人的网站, 你可能会看到这个:  
[![](https://code-maze.com/wp-content/uploads/2017/07/www.code-maze.com-Powered-by-Qualys-SSL-Labs.png)](https://code-maze.com/wp-content/uploads/2017/07/www.code-maze.com-Powered-by-Qualys-SSL-Labs.png)  
你可以看到, 根本不支持 SSL 2 和 3, 而 TLS 1.3 还未启用。

不过, 因为 SSL 流行了很长一段时间, 它已经成为了大部分人熟悉的术语, 现在被用于许多方面。 因此, 当你听到有人用 SSL 代替 TLS, 这是个历史遗留问题, 别想少了, 他们真正想说的不是 SSL。

因为我们要摆脱那种表达方式, 所以从现在开始, 我会用 TLS, 毕竟这个会更加准确。

那么, 客户端和服务器到底是怎样建立安全连接的？

### [](#TLS握手 "TLS握手")TLS 握手

客户端和服务器在进行真正的、加密过的通信之前会有一个”TLS 握手” 的过程。

下面是 TLS 的工作原理 (非常简单, 后面有附加连接)。  
[![](https://code-maze.com/wp-content/uploads/2017/07/TLS-handshake.png)](https://code-maze.com/wp-content/uploads/2017/07/TLS-handshake.png)

在建立完连接之后, 会开始加密通信。

真的 TLS 握手机制会比这个图更加复杂, 不过, 只是为了实现 HTTPS 协议, 你不必了解所有的真实的细节。

如果你想确切了解 TLS 握手的工作原理, 可以在 [RFC 2246](https://www.ietf.org/rfc/rfc2246.txt) 中找找看。

TLS 握手的过程中, 发送的证书扮演着什么样的角色? 又是怎样颁发的呢? 让我们来一起看看吧!

证书是通过 HTTPS 进行安全通信的关键部分, 它由受信任的证书颁发机构颁发。

数字证书让网站的用户在使用网站时可以以安全的方式进行通信。

比如, 在浏览我的文章时, 证书是这个样子的:  
[![](https://code-maze.com/wp-content/uploads/2017/07/Code-Maze-Certificate-modified.png)](https://code-maze.com/wp-content/uploads/2017/07/Code-Maze-Certificate-modified.png)

如果你用的是 Chrome 浏览器, 你可以按 F12 打开开发者工具安全栏查看证书。

我想要点明两件事。在上图的第一个红色框框里, 你可以看到证书的真实目的。它是用来确保你访问的是正确的网站的如果有些网站伪造成你真正想进行通信的网站, 你的浏览器一定会提醒你的。

第二, 浏览器**左边最上面的那个绿色的” 安全” 标志, 只能说明你是在和有证书的网站在进行通信**, 并不能证明你是在与原本想通信的网站进行通信。因此, 要小心了。如果恶意攻击者有一个合法的域名及证书的话, 那么他们就能窃取你的身份凭证了。🙂

_注: 有些钓鱼网站, 通过用与大型网站相似的域名申请域名证书 (比如用 Web1.com 替代原本的 Webone.com), 瞒过了浏览器的法眼, 骗取用户的身份凭证或其他重要信息。_

扩展验证证书 (EV, Extended validation certificates), 换句话说, 就是用来证明网站是由合法实体(域名持有者或公司) 持有的证书。不过, 现在 EV 证书的可用性还存在着争议, 因为可能不适用于互联网上的特殊用户(比如为经过浏览器安全功能训练的用户)。你可以在 URL 栏左方看到 EV 证书持有者的名字。例如, 当你访问 twitter.com 的时候, 你可以看到:  
[![](https://code-maze.com/wp-content/uploads/2017/07/Twitter-EV-certificate.png)](https://code-maze.com/wp-content/uploads/2017/07/Twitter-EV-certificate.png)

这意味着, 网站用 EV 证书来证明其背后的持有者 (或公司)。

### [](#证书链-certificate-chain "证书链(certificate chain)")证书链 (certificate chain)

任何一个服务器都可以 (向客户端) 发送数字证书, 并以此声称它们就是你想要访问的那个服务器。所以, 到底是什么让你使用的浏览器信任服务器返回的证书呢?

这里, 就要引入根证书了。通常, 证书都是链接起来的,  
(链接最顶层的) 根证书是你的电脑全心全意地信任的一个证书。

我博客的证书链是这个样子的:  
[![](http://res.cloudinary.com/dq4c0zqqy/image/upload/v1537102525/certificate-chain-1_xpgrij.png)](http://res.cloudinary.com/dq4c0zqqy/image/upload/v1537102525/certificate-chain-1_xpgrij.png)

最底层的是我的域名证书, 这是由它的上一级的证书签署的, 它的上一级证书是上上级证书签署的… 这样循环往复直到最顶层的根证书。

不过, 到底是谁签署的根证书呢？

好吧, 告诉你, 是它自己给自己签署的!😀

[![](https://code-maze.com/wp-content/uploads/2017/07/root-certificate.png)](https://code-maze.com/wp-content/uploads/2017/07/root-certificate.png)  
颁发给: 域根证书; 颁发者: 域根证书。

你的电脑和浏览器会有一个信任的根证书名单, 这些根证书依靠确认你浏览的域名是通过验证的 (来保证服务器端证书是值得信任的)。若证书链由于某些原因断裂了——当时我在我博客上开启了 CDN[内容分发网络], 就出现了这样的问题——你的电脑会显示你的网站是” 不安全”的。

在键盘上按 win+R 输入” certmgr.msc”后运行证书管理器, 就可以在 windows 系统上查询信任的根证书的名单。然后, 你可以在受信任的根证书颁发机构文件夹中找到电脑信任的证书。这个名单适用于 Windows 系统、IE 浏览器、Chrome 浏览器和 Firefox 浏览器, 换言之, 他们自己在管理自己 (信任的证书) 的名单。

通过交换证书, 客户端和服务器能确认与对方进行通信的人是正确的, 并且可以开始加密传输报文了。

[](#HTTPS的不足 "HTTPS的不足")HTTPS 的不足
---------------------------------

HTTPS 有时会让人误以为当前站点是绝对安全的，实际上启用 HTTPS 机制只是提高了网站对某些认证攻击的防御能力，比如中间人攻击。但实际上，如果网站后端代码有问题，还是会存在一些其它漏洞，除了中间人攻击之外，还有很多种窃取用户信息的方式。

而且，如果 HTTPS 本身的设置有问题的话，依然可能发生中间人攻击。网站管理员可能会误以为，只要使用了 HTTPS，就不存在中间人攻击了。

如果网站登录页面的后端代码存在漏洞，即使使用了 HTTPS 机制，也还是会导致用户账号密码泄露。所以，根本方法，要解决整个网站的安全问题，而不仅仅是企图依赖于 HTTPS。

[](#总结 "总结")总结
--------------

以上就是 HTTP 系列的全部了希望你可以从中受益, 或者 (通过文章) 你理解了你曾经不能理解的概念。

在写文章的过程中, 我获得了喜悦和大量的新知识。希望我的文章也能给你带来快乐 (或者至少于都过程是享受的)。🙂