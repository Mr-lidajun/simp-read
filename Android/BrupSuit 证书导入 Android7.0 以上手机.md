> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.chenjia.me](https://blog.chenjia.me/articles/171029-223953.html)

2017-10-29(日) by [chenjia.me](https://blog.chenjia.me/author/chenjiame.html)

Android 安全策略提升
--------------

Android7.0 对 SSL 证书提升了大量限制。 例如应用不信任用户证书，用户无法自己导入证书等。

如何破解
----

我们可以通过`https://github.com/levyitay/AddSecurityExceptionAndroid`项目来对 apk 进行修改。也可以通过导入证书为系统证书来实现。后者需要有 root 权限

作为证书变为系统证书
----------

1.  从 brupsuit 中导出证书文件`certificate.der`
2.  生成 PEM 文件`openssl x509 -in certificate.der -inform DER -out certificate.pem -outform PEM`
3.  提取 hash:`openssl x509 -inform PEM -subject_hash -in certificate.pem | head -1`, 记住输出的 hash，例如`a0b1c2d3`
4.  `cat certificate.pem > a0b1c2d3.0`
5.  `openssl x509 -inform PEM -text -in certificate.pem -out /dev/null >> a0b1c2d3.0`
6.  连接手机，执行`adb root`和`adb remount`
7.  `adb push a0b1c2d3.0 /system/etc/security/cacerts/`
8.  进入手机终端`adb shell`
9.  加上权限`chmod 644 /system/etc/security/cacerts/a0b1c2d3.0`
10.  重启手机，查看证书是否安装完成。

参考
--

1.  [https://nvisium.com/blog/2017/07/12/advantages-and-disadvantages-of-android-n-network-security-configuration/](https://nvisium.com/blog/2017/07/12/advantages-and-disadvantages-of-android-n-network-security-configuration/)
2.  [https://github.com/levyitay/AddSecurityExceptionAndroid](https://github.com/levyitay/AddSecurityExceptionAndroid)

* * *

Comments
--------