> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.colabug.com](https://www.colabug.com/2020/0409/7232893/)

### 简介

大家平时在开发工程中，解决测试，线上 bug 的时候等很多时候都会用到网络抓包工具来分析服务器端返回的数据，找到十足的证据甩锅给后端的同事。

就 Android 开发来说，http 抓包，很容易，如果是 https 呢，有一点麻烦，用低版本的手机（比如 Android5.0）装上抓包工具生成的证书也就解决了。但是在高版本手机（比如 Android7.0）发现装了证书不管用，直接提示握手失败或者无法访问网络，或者抓包工具什么数据内容都看不到，看到了也是一堆乱码。不慌，能解决。

如果应用进行了代理检测，应用加固，当线上出问题了，自己反编译 apk 又不太好操作（当然可以在本地打一个线上环境的 apk 调试 bug），这时候怎么版，不慌，能解决。

#### 我写的其它抓包相关的文章

*   Wireshark  
    Wireshark 抓包工具使用，一共四篇文章，点击链接，里面有目录  
    [https://www.jianshu.com/p/28740f217642](https://www.colabug.com/goto/aHR0cHM6Ly93d3cuamlhbnNodS5jb20vcC8yODc0MGYyMTc2NDI=)
*   其它  
    [Android 利用 tcpdump 和 wireshark 抓取网络数据包](https://www.colabug.com/goto/aHR0cHM6Ly93d3cuamlhbnNodS5jb20vcC80MDVhZDc2YjQzOGU=)

### 工具

#### PC 端

*   [Fiddler](https://www.colabug.com/goto/aHR0cHM6Ly9saW5rcy5qaWFuc2h1LmNvbS9nbz90bz1odHRwcyUzQSUyRiUyRnd3dy50ZWxlcmlrLmNvbSUyRmRvd25sb2FkJTJGZmlkZGxlciUyRmZpZGRsZXI0)
    
    Window 下载地址，mac 也有，自己可以百度，mac 没有直接支持的安装包，可以通过其它工具配合来支持，我 mac 是安装上了，但是始终用不了。
    
*   [charles](https://www.colabug.com/goto/aHR0cHM6Ly9saW5rcy5qaWFuc2h1LmNvbS9nbz90bz1odHRwcyUzQSUyRiUyRnd3dy5jaGFybGVzcHJveHkuY29tJTJG)
    
    目前我在 mac 上用的抓包工具，Window 可以自己去下载安装。
    
*   [wireshark](https://www.colabug.com/goto/aHR0cHM6Ly9saW5rcy5qaWFuc2h1LmNvbS9nbz90bz1odHRwcyUzQSUyRiUyRnd3dy53aXJlc2hhcmsub3JnJTJGZG93bmxvYWQuaHRtbA==)
    
    专业的抓包工具，使用更加复杂，但功能是否强悍。
    

#### 手机端

这里列的是 Android 手机。

*   [HttpCanary](https://www.colabug.com/goto/aHR0cHM6Ly9saW5rcy5qaWFuc2h1LmNvbS9nbz90bz1odHRwcyUzQSUyRiUyRmdpdGh1Yi5jb20lMkZNZWdhdHJvbktpbmclMkZIdHRwQ2FuYXJ5)
    
    如果可以访问 Google Play 可以直接下载安装，如果没有自己百度，上面有很多下载地址。
    
*   [平行空间](https://www.colabug.com/goto/aHR0cHM6Ly9saW5rcy5qaWFuc2h1LmNvbS9nbz90bz1odHRwJTNBJTJGJTJGcGFyYWxsZWwtYXBwLmNvbSUyRmluZGV4X2NuLnBocA==)
    
    实现应用双开，和抓包工具配合，抓取 https。
    
*   [Packet Capture](https://www.colabug.com/goto/aHR0cHM6Ly9saW5rcy5qaWFuc2h1LmNvbS9nbz90bz1odHRwcyUzQSUyRiUyRnd3dy5uZXRzY2FudG9vbHMuY29tJTJGbnN0cHJvLWRlbW8tZG93bmxvYWQuaHRtbA==)

### 其它工具

*   [VirtualXposed](https://www.colabug.com/goto/aHR0cHM6Ly9saW5rcy5qaWFuc2h1LmNvbS9nbz90bz1odHRwcyUzQSUyRiUyRnZ4cG9zZWQuY29tJTJG)
    
    在非 ROOT 设备上实现 Xposed 部分功能。
    
*   [Lucky Patcher](https://www.colabug.com/goto/aHR0cHM6Ly9saW5rcy5qaWFuc2h1LmNvbS9nbz90bz1odHRwcyUzQSUyRiUyRnd3dy5sdWNreXBhdGNoZXJzLmNvbSUyRmRvd25sb2FkJTJG)
    
    Xposed module，具体各种破解功能，具网上资料，可以破解 HttpCanary，但是我没有成功。
    
*   [JustTrustMe](https://www.colabug.com/goto/aHR0cHM6Ly9saW5rcy5qaWFuc2h1LmNvbS9nbz90bz1odHRwcyUzQSUyRiUyRmdpdGh1Yi5jb20lMkZGdXppb24yNCUyRkp1c3RUcnVzdE1lJTJGcmVsZWFzZXM=)
    
    Xposed module，关闭证书检测，这样 https 可以直接抓包。
    

### 说明

本篇文章，以自己亲自事件总结的抓包相关的知识点。本着以简单的方式解决问题。下面将从简到难，分场景讲解。

### http

http 抓包就很简单了，用上面的任何一个工具都能够轻易抓包到，至于抓到包后的一些骚操作就要自己研究抓包软件了。

### https

https 通信过程，会进行认证，数据加密，如果设置代理抓 https 网络数据，会出现握手失败，无法连接网络，看不到抓包内容或者乱码等情况。

#### Android 版本对证书的限制

Android 从 7.0 开始系统不再信任用户 CA 证书（应用 targetSdkVersion >= 24 时生效，如果 targetSdkVersion <24 即使系统是 7.0 + 依然会信任）。也就是说即使安装了用户 CA 证书，在 Android 7.0 + 的机器上，targetSdkVersion>= 24 的应用的 HTTPS 包就抓不到了。

#### 手机系统版本低于 24 或者 targetSdkVersion < 24

安装抓包工具提供的证书，安装步骤自己百度，正常抓包。

#### 手机系统版本高于 2（包含 24）并且 targetSdkVersion >=24

Android 系统不在信任用户证书，即使你赚了抓包工具提供的证书依然不行。解决办法如下：

非 root 设备

*   HttpCanary + 平行空间
    
    HttpCanary 设置里面可以直接安装平行空间，然后在 HttpCanary 设置里面安装证书，使用平行空间空间打开应用，然后用 HttpCanary 抓包。
    
    异常情况，如果出现握手失败或者无法连接网络，可以先关闭 HttpCanary，先进入应用，再打开 HttpCanary，重新抓，这是我最开始使用 HttpCanary + 平行空间出现的问题，后面好像没有出现了。
    
*   pc 抓包工具 + 平行空间
    
    手机安装 pc 抓包工具提供的证书或者 HttpCanary 设置中安装证书，平行空间中打开应用。pc 或者手机正常抓包。
    
    在锤子 7.1.1 系统，targetSdkVersion = 26 亲测，可以正常抓包。
    
*   VirtualXposed + 上面两种方案
    
    VirtualXposed 用在非 root 设备上实现 Xposed 功能，对抗应用代理检测，应用加固等阻碍抓包机制。
    

root 设备

root 设备除了拥有非 root 设备解决方案以外，其它解决办法。

*   HttpCanary
    
    Android 系统证书都是以 “.0” 结尾的，系统证书位置： /system/etc/security/cacerts。HttpCanary 可以将自己的 “.0” 证书导入系统证书目录，这样可以实现抓包。
    
    刚开始用这种方式的时候，也出现了握手失败或者网络连接失败，通过先关闭抓包，走应用正常流程后再打开抓包就可以了（有点奇怪）或者 关闭抓包后，等操作应用后，立即打开抓包，可以抓包 https。
    
*   JustTrustMe
    
    安装抓包工具提供的证书，pc，手机正常抓包，但是启动 JustTrustMe，很多浏览器里面的网站打不开，这些页面服务器对证书应该有验证。
    
*   Xposed
    
    对抗代理检测等机制，Xposed 有强大，开启应用调试，hook 等等。
    

### 证书固定

客户端内置 Server 端真正的公钥证书。

在这种情况下，由于 MITM Server 创建的公钥证书和 Client 端内置的公钥证书不一致，MITM Server 就无法伪装成真正的 Server 了。这时，抓包就表现为 App 网络错误。

有些服务器采用的自签证书（证书不是由真正 CA 发行商签发的），这种情况 App 请求时必须使用证书固定。

很多应用还会将内置的公钥证书伪装起来或者加密，防止逆向提取。

*   持有证书  
    好办，导入抓包工具即可。
*   没有  
    反编译 APK，Xposed hook 应用关键代码，关闭证书设置或者替换成自己的。

### 非 HTTP 协议抓包

如果上面的方式都不能够抓到网络包，那么考虑请求可能不是 http，https 请求，可以选择支持其它协议的抓包工具。

### 其它方式参考

注意：本文来源网络 / 媒体，本站无法对本文内容的真实性、完整性、及时性、原创性提供任何保证，  
请您自行验证核实并承担相关的风险与后果！  
CoLaBug.com 遵循 [[CC BY-SA 4.0](https://www.colabug.com/goto/cclicensing)] 分享并保持客观立场，本站不承担此类作品侵权行为的直接责任及连带责任。  
如您有版权、意见、投诉等问题，请通过 [[eMail]](https://www.colabug.com/goto/tousu) 联系我们处理，如需商业授权请联系原作者 / 原网站。