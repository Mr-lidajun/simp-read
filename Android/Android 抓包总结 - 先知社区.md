> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [xz.aliyun.com](https://xz.aliyun.com/t/6551?spm=5176.12901015.0.i12901015.103a525c9rsqzO)

这篇文章算是总结一下我之前抓包遇到的一些问题, 个人属性里带 bug, 所以遇到的问题会比较多, 算是给大家提供一个抓包抓不到应该如何解决的思路。

Android 中可用的抓包软件有 fiddler、burpsuite、Charls、HttpCanary、Packet Capture、tcpdump、wireshark 等等。tcpdump 和 wireshark 可以解决部分不是使用 HTTP/HTTPS 协议传输数据的 app, 用 tcpdump 抓包, 用 wireshark 分析数据包。

如果想抓取三大运营商传输的数据包并分析, 因其路由规则的限制, 可能还是需要在 android 系统中利用 iptables 设置反向代理, 用 Fiddler 解密数据包之后分析, 不过好像 Fiddler 好像有自己的反向代理设置方法, 这部分了解不多。

Charls 是 Mac 上常见的抓包工具, 我没用过, 不过网上蛮多教程的。HttpCanary 和 Packet Capture 这两个工具与常规的电脑上的代理抓包不同的是, 能保证一定能抓取到数据包, 我一般都用 Packet Capture 来验证应用是否发送请求。HttpCanary 被称为移动端的 Fiddler, 能够改包和劫持双向认证的应用传输的数据包, 感觉还是蛮强大的。

Fiddler 抓取 Android 数据包
----------------------

### 基础设置

1.  下载好 Fiddler 之后, 打开该软件, 生成证书。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20191012152300-1f4f8c84-ecc1-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20191012152300-1f4f8c84-ecc1-1.png)

设置连接

[![](https://xzfile.aliyuncs.com/media/upload/picture/20191012152301-1f8ebfc6-ecc1-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20191012152301-1f8ebfc6-ecc1-1.png)

设置 HTTPS

[![](https://xzfile.aliyuncs.com/media/upload/picture/20191012152301-1fc8ff10-ecc1-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20191012152301-1fc8ff10-ecc1-1.png)

用 ipconfig 查看当前主机的 ip

[![](https://xzfile.aliyuncs.com/media/upload/picture/20191012152302-205e0ede-ecc1-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20191012152302-205e0ede-ecc1-1.png)

手机和电脑在同一局域网中即可, 手机端设置 WLAN 种给网络设置代理, 选择对应的 WLAN, 选中修改网络, 手动设置代理, 主机名填上面电脑 ip 地址, 端口写 fiddler 默认端口 8888。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20191020193957-57e046a6-f32e-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20191020193957-57e046a6-f32e-1.png)

手机端用浏览器访问 [http:// 电脑 IP:8888(http:// 电脑 IP: 端口](http://电脑IP:8888(http://电脑IP:端口)), 观察网络是否访问成功, 成功之后, 点击 "FiddlerRoot.certificate" 下载 Fiddler 的证书并安装。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20191012152303-20c6b934-ecc1-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20191012152303-20c6b934-ecc1-1.png)

如果上述步骤都原原本本做完了, 还是不能出现上图的效果, 可以换个路由或者直接手机开热点。我当时遇到不能访问的问题, ping 了一下, 一直显示 destination unreachable, 应该是路由器安全规则的限制, 换成了手机开热点就 ok 了。

继续进行测试的时候, 发现不管是修改密码还是用验证码进行登录, 我都抓不到那些包。想不出是哪里出了问题..... 大概找了一下, 发现是 SSL Pinning 的机制阻止了我抓包。使用了 Xposed+JustTrustMe, 就抓取到数据包了, 数据包如下：

[![](https://xzfile.aliyuncs.com/media/upload/picture/20191012152303-20fd3374-ecc1-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20191012152303-20fd3374-ecc1-1.png)

如果知道 Fiddler 怎么抓包了, 不知道怎么改包, 可以用 Fiddler 左下角的黑框框中断请求, 修改之后再发出, 比如输入`bpu baidu.com`就可以中断所有发向 baidu.com 的请求。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20191012152304-2128b4cc-ecc1-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20191012152304-2128b4cc-ecc1-1.png)

之后查看中断的数据包会出现如下效果, 修改完点击 Run to Completion 就可以把请求发出去了。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20191012152304-2152d3c4-ecc1-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20191012152304-2152d3c4-ecc1-1.png)

### Fiddler 设置之后手机无法连接上代理

1.  关闭电脑防火墙
    
2.  打开注册表（cmd-regedit）, 在 HKEY_CURRENT_USER\Software\Microsoft\Fiddler2 下创建一个 DWORD, 值置为 80（十进制）[在空白处右键即可创建]。
    

[![](https://xzfile.aliyuncs.com/media/upload/picture/20191012152304-218544b2-ecc1-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20191012152304-218544b2-ecc1-1.png)

1.  编写 fiddlerScript rule：在 Fiddler 上点击 Rules->Customize Rules, 用 Ctrl+F 查找 OnBeforeRequest 方法添加一行代码。

```
if (oSession.host.toLowerCase() == "webserver:8888") 
{
        oSession.host = "webserver:80"; 
}

```

Burpsuite 抓取 Android 数据包
------------------------

### 基础设置

Burpsuite 改包的步骤就不在这里赘述了, 网上有很多教程, 接下来我们要设置 burpsuite, 以求抓取到数据包, 设置如下：

[![](https://xzfile.aliyuncs.com/media/upload/picture/20191012152305-21bf1746-ecc1-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20191012152305-21bf1746-ecc1-1.png)

提示, 监听的端口号、电脑内网 ip 要和手机上的代理设置一致, 电脑内网 ip 可以用 ipconfig 查看。用 burpsuite 一直抓取不到 https 的证书, 怀疑是我 burpsuite 证书没有安装到手机上, 所以我现在先将它装到系统证书中, 再看看能不能先抓取到 https 的证书。

### 安装证书至系统中

1、下载. der 格式的证书, 将下载的 cacert.der 转换格式, 并获取证书 hash 值, 生成 <证书 hash>.0 文件, 例如：7bf17d07.0

2、把 <证书 hash>.0 证书 push 到 / data/local/tmp 目录下后移动至 / system/etc/security/cacerts/  
(mv 操作出错之后, 先试一下 “mount -o rw,remount /system” 如果出现了报错“mount: '/system' not in /proc/mounts”, 再尝试“mount -o rw,remount /”, 就可以操作 system 目录了)

3、重启手机

只有 root 环境才能将 proxy 证书安装至 android 系统证书中, 这种方法好像能绕过应用本地证书校验, 其实 burp 和 Fiddler 还有其他的代理证书的安装方法都差不多, 最后将 <hash 证书>.0 的文件 mv 至 / system/etc/security/cacerts / 目录下即可, 不建议直接将用户证书直接 mv, 可能会导致环境出错也不好排查证书错误, 甚至可能导致 android 网络环境出错。</hash 证书 >

下面是具体步骤, 先在设置本地代理, 将 burpsuite 证书下载下来

[![](https://xzfile.aliyuncs.com/media/upload/picture/20191012152305-21efb61c-ecc1-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20191012152305-21efb61c-ecc1-1.png)

打开浏览器输入本地地址, 下载. der 格式的证书

[![](https://xzfile.aliyuncs.com/media/upload/picture/20191012152305-222207e8-ecc1-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20191012152305-222207e8-ecc1-1.png)

此处参照文章 [BrupSuit 证书导入 Android7.0 以上手机](https://blog.chenjia.me/articles/171029-223953.html), 因为我 windows 本地安装了 ubuntu 的子系统, 所以直接用 ubuntu1604 子系统对证书进行操作。

```
// 转换证书的格式
$ openssl x509 -in cacert.der -inform DER -out cacert.pem -outform PEM
// 提取证书的hash
$ openssl x509 -inform PEM -subject_hash -in cacert.pem

```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20191012152306-226b1adc-ecc1-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20191012152306-226b1adc-ecc1-1.png)

上图中的`7bf17d07`就为证书的 hash 值, 将该目录下生成的`7bf17d07.0`文件 push 到手机中, 最后移动到 / system/etc/security/cacerts / 目录下

```
$ adb push 7bf17d07.0 /data/local/tmp
$ adb shell 
sailfish:/ $ su
sailfish:/ # mount -o rw,remount / # 拥有操作/目录的权限, 本意是要操作/system目录
sailfish:/ # mv /data/local/tmp/7bf17d07.0 /system/etc/security/cacerts/7bf17d07.0

```

按照原本的文章应该给`7bf17d07.0`文件添加 644 权限, 但是我具体操作的时候没有添加权限也成功了, 如果按照我上面的步骤出错了, 可以尝试给文件添加权限。重启之后可以看到证书安装成功。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20191012152306-229ad84e-ecc1-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20191012152306-229ad84e-ecc1-1.png)

第一次安装证书的时候出现了不能访问使用 https 协议的网站, 应该是我测试的手机环境出现了问题, 我重新刷机再按照上面的步骤走一遍就成功了, 如果你们也遇到访问 https 网站失败的问题, 可以尝试一下使用这个方法。

抓包最重要的是看能不能抓取到数据包, 想要抓到包就要看 app 使用什么传输协议了, 一般情况下使用 HTTP 都是能抓到包的, 这也就不难理解, 为什么 google 坚持推广 HTTPS 了。为什么说使用 HTTPS 会抓不到包？现在的 HTTPS 都是基于 TLS 协议的, 它的特点就是需要确认传输双方的身份。确认了身份之后再传输数据, 这样就能避免中间人攻击了。下面来看看 HTTPS, 是怎么进行数据传输的, 发现 HTTPS 需要先建立连接才能传输数据。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20191012152306-22c991a2-ecc1-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20191012152306-22c991a2-ecc1-1.png)

讲到要认证对方的身份, 我就想起了之前翻译的一篇 [HTTP 安全](https://se8s0n.github.io/2018/09/11/HTTP%E7%B3%BB%E5%88%97(%E4%BA%94)), 里面就有提及到在使用 HTTPS 协议的过程中, 客户端和服务器通过证书来判断对方的身份。之前没有怎么理解, 现在才对证书的作用有比较深刻的理解。

文章中举了个例子, Chrome 浏览器通过判断是否有证书来判断你访问的网站是否安全的, 并不是你访问的网站真的是安全的。提及这个是因为 app 使用 HTTPS 传输也是看证书的, 只不过有的 app 限制的比较严格只信任自带的证书, 有的 app 安全要求没那么高, 直接信任系统证书。

抓包出错排查思路
--------

[![](https://xzfile.aliyuncs.com/media/upload/picture/20191012152307-231e3ca2-ecc1-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20191012152307-231e3ca2-ecc1-1.png)

上面是大概的排查思路, 具体的细节可能有些差异。如果 proxy 带有证书校验, 且 JustTrustMe 绕不过去, 可能要自己重新根据该应用定制 hook 模块, 去绕过其本地证书校验, 但是大部分应用都能通过将证书安装为系统证书绕过, 如果无法在 root 环境下运行, 文章[《Intercepting traffic from Android Flutter applications》](https://blog.nviso.be/2019/08/13/intercepting-traffic-from-android-flutter-applications/)和 JustTrustMe 的源码应该能给你提供一点 hook 模块绕过证书校验的思路, 《Intercepting traffic from Android Flutter applications》讲的是如何绕过 google 开源框架 Flutter 中的证书校验进行抓包。

最后说抓不到包还有一种可能性, 就是要求一定要用 SIM 卡发出传输请求的数据包.... 不过这个应该应该只有使用了三大运营商的 SDK 或他们的应用才会出现这种情况, 这部分应该只能用反向代理才有可能抓取到传输的数据包了, 具体情况就要具体分析了。

当时尝试 tcpdump+wireshark 效果不怎么样, 因为所有的数据都经过了加密, 而 wireshark 不能解密, 所以对于加密传输的数据包这种方法可能有点鸡肋, 听说有 [mitmdump](https://docs.mitmproxy.org/stable/) 抓包工具专门处理 linux 环境下 http/https 的数据包, 不过我自己没用过, 之后要是接触了会进一步补充。

SSL pinning 和双向认证的区别
--------------------

SSL pinning 实际上是客户端锁定服务器端的证书, 在要与服务器进行交互的时候, 服务器端会将 CA 证书发送给客户端, 客户端会调用函数对服务器端的证书进行校验, 与本地的服务器端证书 (存放在`\<app>\asset`目录或`\res\raw`下) 进行比对。

而双向认证是添加了客户端向服务器发送 CA 证书, 服务器端对客户端的证书进行校验的部分, 具体详情可看文章[扯一扯 HTTPS 单向认证、双向认证、抓包原理、反抓包策略](https://juejin.im/post/5c9cbf1df265da60f6731f0a)的单向认证、双向认证部分的内容。

抓取 HTTPS 的数据包
-------------

### Frida 绕过 SSL 单向校验

昨天刚好遇到 JustTrustMe 无法绕过 SSL 单向校验的情况, 这几天接触了 Frida, 就尝试用 DBI 的方法绕过 SSL 的单向校验, 参考文章 [Universal Android SSL Pinning bypass with Frida](https://techblog.mediaservice.net/2017/07/universal-android-ssl-pinning-bypass-with-frida/) 这里就不详细地说明 Frida 的安装方法及使用方法了。

设置 Fiddler 代理, 在本地下载 Fiddler 的证书, 将证书直接重命名为`cert-der.crt`。之后将证书 push 到`/data/local/tmp`目录下, 在 adb shell 里输入`./frida-server &`再在 PC 端进行操作。

新建一个 frida-android-repinning.js 文件, 详细代码如下：

```
setTimeout(function(){
    Java.perform(function (){
        console.log("");
        console.log("[.] Cert Pinning Bypass/Re-Pinning");

        var CertificateFactory = Java.use("java.security.cert.CertificateFactory");
        var FileInputStream = Java.use("java.io.FileInputStream");
        var BufferedInputStream = Java.use("java.io.BufferedInputStream");
        var X509Certificate = Java.use("java.security.cert.X509Certificate");
        var KeyStore = Java.use("java.security.KeyStore");
        var TrustManagerFactory = Java.use("javax.net.ssl.TrustManagerFactory");
        var SSLContext = Java.use("javax.net.ssl.SSLContext");

        // Load CAs from an InputStream
        console.log("[+] Loading our CA...")
        var cf = CertificateFactory.getInstance("X.509");

        try {
            var fileInputStream = FileInputStream.$new("/data/local/tmp/cert-der.crt");
        }
        catch(err) {
            console.log("[o] " + err);
        }

        var bufferedInputStream = BufferedInputStream.$new(fileInputStream);
        var ca = cf.generateCertificate(bufferedInputStream);
        bufferedInputStream.close();

        var certInfo = Java.cast(ca, X509Certificate);
        console.log("[o] Our CA Info: " + certInfo.getSubjectDN());

        // Create a KeyStore containing our trusted CAs
        console.log("[+] Creating a KeyStore for our CA...");
        var keyStoreType = KeyStore.getDefaultType();
        var keyStore = KeyStore.getInstance(keyStoreType);
        keyStore.load(null, null);
        keyStore.setCertificateEntry("ca", ca);

        // Create a TrustManager that trusts the CAs in our KeyStore
        console.log("[+] Creating a TrustManager that trusts the CA in our KeyStore...");
        var tmfAlgorithm = TrustManagerFactory.getDefaultAlgorithm();
        var tmf = TrustManagerFactory.getInstance(tmfAlgorithm);
        tmf.init(keyStore);
        console.log("[+] Our TrustManager is ready...");

        console.log("[+] Hijacking SSLContext methods now...")
        console.log("[-] Waiting for the app to invoke SSLContext.init()...")

        SSLContext.init.overload("[Ljavax.net.ssl.KeyManager;", "[Ljavax.net.ssl.TrustManager;", "java.security.SecureRandom").implementation = function(a,b,c) {
            console.log("[o] App invoked javax.net.ssl.SSLContext.init...");
            SSLContext.init.overload("[Ljavax.net.ssl.KeyManager;", "[Ljavax.net.ssl.TrustManager;", "java.security.SecureRandom").call(this, a, tmf.getTrustManagers(), c);
            console.log("[+] SSLContext initialized with our custom TrustManager!");
        }
    });
},0);

```

在 cmd, 输入如下命令:

```
$ adb push burpca-cert-der.crt /data/local/tmp/cert-der.crt
$ frida -U -f it.app.mobile -l frida-android-repinning.js --no-pause

```

在关闭应用的情况下 (避免 Magisk Hide 处于开启状态), 可得到回显并绕过 SSL pinning。  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20191031094940-b409f166-fb80-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20191031094940-b409f166-fb80-1.png)

### 绕过 SSL 双向校验

其实 SSL 双向校验是在 SSL 单向校验的基础上, 在说明这部分内容的时候同时也会有绕过 SSL 单向校验详细的步骤。参考文章 [Android 平台 HTTPS 抓包解决方案及问题分析](https://juejin.im/post/5cc313755188252d6f11b463#heading-14), 我们可以先用 sxxl.app 来练练手。

在手机上设置完代理之后, 点击完确认, 发现 app 出现如下弹窗：

[![](https://xzfile.aliyuncs.com/media/upload/picture/20191012152307-23518c56-ecc1-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20191012152307-23518c56-ecc1-1.png)

在这个时候查看 Fiddler 会发现应用没有发出任何请求, 这是因为 app 会对服务器端的证书进行校验, 这时候我们前面安装的 Fiddler 证书就不起作用了, 应用在发现证书是伪造的情况下拒绝发送请求。根据这个报错 + 抓不到包, 我们可以确定应用是存在单向校验的, 也就是 SSL pinning, 让我们先来解决 SSL pinning 的问题。使用 JustTrustMe 可以绕过客户端的证书校验, 下面勾选上 JustTrustMe, 在 Xposed 框架下使用 JustTrustMe 绕过 SSL pinning。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20191012152308-2392e7d2-ecc1-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20191012152308-2392e7d2-ecc1-1.png)

绕过 SSL pinning 之后, 就能使用 Fiddler 抓取到 HTTPS 的数据包了。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20191012152308-23c82bae-ecc1-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20191012152308-23c82bae-ecc1-1.png)

我随便输入了一个手机号码, 按下确定之后, 服务器回传了 400 的状态码过来, 说需要发送证书以确认客户端的身份。到这一步基本能确定是存在双向校验的了, 接下来的工作就是绕过 SSL 服务器端的校验了。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20191012152308-240295fa-ecc1-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20191012152308-240295fa-ecc1-1.png)

如果服务器端会对客户端证书进行校验, 证书应该就直接存放在 apk 里, 网上与 SSL 双向校验相关的文章都将证书放到`<app>/asset`目录下, 也就是 app 的资源目录下, 也有可能放在`/res/raw`目录下。直接将 app 解压之后, 发现证书的位置如下：

[![](https://xzfile.aliyuncs.com/media/upload/picture/20191012152309-242dbee2-ecc1-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20191012152309-242dbee2-ecc1-1.png)

如果找半天没找到就用关键词`.p12/.pfx`搜索证书文件。

在我们要使用该证书的时候, 需要输入安装证书的密码。这时候就需要从源码中获取安装证书的密码了。可能是因为多个 dex 文件的原因, 直接用 JEB 反编译的时候出错了, 所以我用 GDA 反编译来分析应用的源代码

### 获取安装证书的密码

发现通过关键词 "PKCS12" 能够定位到加载证书的位置。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20191012152309-2468df04-ecc1-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20191012152309-2468df04-ecc1-1.png)

上图第二个红框中的 load 函数的第二个参数其实就是证书的密钥, 追根溯源, 我们可以知道 v1 参数是下图中调用的函数的返回值。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20191012152309-24a5cfae-ecc1-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20191012152309-24a5cfae-ecc1-1.png)

[![](https://xzfile.aliyuncs.com/media/upload/picture/20191012152310-24d8cbfc-ecc1-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20191012152310-24d8cbfc-ecc1-1.png)

上图的函数的功能就是传递 p0 参数, 也就是说 p0 参数就是证书安装密码。想获取这个密码, 关键在于 Auto_getValue 函数。到这一步, 只要跟进 Null_getStorePassword 函数看看就好了。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20191012152310-250aee70-ecc1-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20191012152310-250aee70-ecc1-1.png)

跟进去发现调用了 native 层的函数, 查看 init 函数中具体加载的是哪个 so 文件：

[![](https://xzfile.aliyuncs.com/media/upload/picture/20191012152310-253d37e0-ecc1-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20191012152310-253d37e0-ecc1-1.png)

用 IDA 反编译 soul-netsdk 之后, 搜索字符串 "getStorePassword", 就定位到函数 getStorePassword 上了, F5 之后, 获得伪代码和密钥：

[![](https://xzfile.aliyuncs.com/media/upload/picture/20191012152311-25c05012-ecc1-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20191012152311-25c05012-ecc1-1.png)

### 代理添加客户端证书

HttpCanary 添加客户端证书进行抓包的过程可以参照文章 [Android 平台 HTTPS 抓包解决方案及问题分析](https://juejin.im/post/5cc313755188252d6f11b463), 在自己头昏的时候也感谢这篇文章的作者 MegatronKing 点醒我。下面主要讲解 Fiddler 和 burpsuite 添加客户端证书的方法。

#### fiddler 操作过程

尝试一下用 Fiddler 处理这部分的内容来安装客户端的证书, 用来绕过双向认证。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20191012152312-25f73f78-ecc1-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20191012152312-25f73f78-ecc1-1.png)

用 Fiddler 抓取该应用的数据包的时候, 发现 Fiddler 出现了上面的弹窗, 提示要添加 ClientCertificate.cer, 才能抓取到传输的数据包, 不然只会出现 400 的状态码。而我们文件目录下只能找到`client.p12`和`client.crt`两种格式的证书文件, 所以我们需要将已有的 client 证书转换成`.cer`格式的证书。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20191012152312-262640a2-ecc1-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20191012152312-262640a2-ecc1-1.png)

好像应用中只出现`.p12`格式的证书的情况比较常见, 所以下面只会提及如何使用 openssl 将`.p12`格式的证书转换成`.cer/.der`格式的证书。(.der 和. cer 格式的证书仅有文件头和文件尾不同)

下面的命令实现了证书的格式转换, `.p12`->`.pem`->`.cer`, 在生成`.pem`格式的证书之后, 需要输入证书的密码, 也就是我们上面逆向获取的证书密码。最后将`ClientCertificate.cer`移动到之前 Fiddler 弹窗出现的目录下, 也就是`<Fiddler安装路径>\Fiddler2`下。

```
# 将.p12证书转换成.pem格式
$ openssl pkcs12 -in client.p12 -out ClientCertificate.pem -nodes
Enter Import Password:
# 将.pem证书转换成.cer格式
$ x509 -outform der -in ClientCertificate.pem -out ClientCertificate.cer

```

现在打开 Fiddler 尝试抓包, 发现原本显示 400 的数据包现在能够正常抓取到了, 如果还是不能正常抓取到, 双击`client.p12`将证书安装到本地试试看。  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20191012152312-267723c8-ecc1-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20191012152312-267723c8-ecc1-1.jpg)

#### burp 操作过程

手机的 burpsuite 证书安装成功之后, 我们会发现只能抓取到 400 的状态码。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20191012152313-26ad33c8-ecc1-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20191012152313-26ad33c8-ecc1-1.png)

因为要绕过服务器端对证书的验证, 我们还需要在这里添加上面我们在 asset 目录下找到的证书。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20191012152313-26f098de-ecc1-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20191012152313-26f098de-ecc1-1.png)

[![](https://xzfile.aliyuncs.com/media/upload/picture/20191012152314-271f0a20-ecc1-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20191012152314-271f0a20-ecc1-1.png)

安装完就能正常抓取数据包了。

抓取 TCP 的数据包
-----------

现在还不知道怎么能够获取 TCP 的数据包并对其中的内容进行解密, 不过之前在看雪上看到一篇分析使用 TCP 传输协议的文章[某直播 APP 逆向 TCP 协议分析](https://bbs.pediy.com/thread-251063.htm), 我大概看了一下, 文章是从逆向的角度分析的, 具体怎么从渗透的角度发现是 TCP 协议传输的数据包还没有分析过, 看作者使用了 wireshark 抓取应用的数据包并进行分析, 这个还是要重新分析一下的。

文章最后, 还要感谢华华师傅, 其实实习的时候接触 android 的时间也不长, 但是之后真的让我接触到很多和学到很多, 也谢谢师傅能耐心地帮我解答问题, 感恩。