> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [techblog.mediaservice.net](https://techblog.mediaservice.net/2017/07/universal-android-ssl-pinning-bypass-with-frida/)

Android SSL Re-Pinning
----------------------

Two kinds of SSL Pinning implementations can be found in Android apps: the home-made and the proper one. The former is usually a single method, performing all the certificate checks (possibly using custom libraries), that returns a Boolean value. This means that this approach can be easily bypassed by identifying the interesting method and flipping the return value. The following example is a simplified version of a Frida JavaScript script:

[![](https://techblog.mediaservice.net/wp-content/uploads/2017/07/Schermata-2017-06-26-alle-14.20.14-300x62.png)](https://techblog.mediaservice.net/wp-content/uploads/2017/07/Schermata-2017-06-26-alle-14.20.14.png)

After we identify the offending method (hint: logcat) we basically hijack it and let it always return true.

When SSL Pinning is instead performed according to the official [Android documentation](https://developer.android.com/training/articles/security-ssl.html), well… things get tougher. There are many excellent solutions out there, being custom android images, underlying frameworks, _socket.relaxsslcheck=yes_ , etc. Almost every attempt at bypassing SSL Pinning is based on manipulating the SSLContext. Can we manipulate the SSLContext with Frida? What we wanted was a generic/universal approach and we wanted to do it with a Frida JavaScript script.

The idea here is to do exactly what the official documentation suggests doing so we’ve ported the SSL Pinning Java code to Frida JavaScript.

How it works:

1.  Load our rogue CAs cert from device
2.  Create our own KeyStore containing our trusted CAs
3.  Create a TrustManager that trusts the CAs in our KeyStore

When the application initializes its SSLContext we hijack the SSLContext.init() method and when it gets called, **we swap the 2nd parameter**, which is the application TrustManager, **with our own TrustManager** we previously prepared. (SSLContext.init(KeyManager, TrustManager, SecuRandom)).

This way we basically re-pinn the application to our own CA!

[![](https://techblog.mediaservice.net/wp-content/uploads/2017/07/Schermata-2017-06-29-alle-18.11.14-1-300x210.png)](https://techblog.mediaservice.net/wp-content/uploads/2017/07/Schermata-2017-06-29-alle-18.11.14-1.png)

Example
-------

`$ adb push burpca-cert-der.crt /data/local/tmp/cert-der.crt`

`$ adb shell "/data/local/tmp/frida-server &"`

`$ frida -U -f it.app.mobile -l frida-android-repinning.js --no-pause`

`[…]  
[USB::Samsung GT-31337::['it.app.mobile']]->  
[.] Cert Pinning Bypass/Re-Pinning  
[+] Loading our CA...  
[o] Our CA Info: CN=PortSwigger CA, OU=PortSwigger CA, O=PortSwigger, L=PortSwigger, ST=PortSwigger, C=PortSwigger  
[+] Creating a KeyStore for our CA...  
[+] Creating a TrustManager that trusts the CA in our KeyStore...  
[+] Our TrustManager is ready...  
[+] Hijacking SSLContext methods now...  
[-] Waiting for the app to invoke SSLContext.init()...  
[o] App invoked javax.net.ssl.SSLContext.init...  
[+] SSLContext initialized with our custom TrustManager!  
[o] App invoked javax.net.ssl.SSLContext.init...  
[+] SSLContext initialized with our custom TrustManager!  
[o] App invoked javax.net.ssl.SSLContext.init...  
[+] SSLContext initialized with our custom TrustManager!  
[o] App invoked javax.net.ssl.SSLContext.init...  
[+] SSLContext initialized with our custom TrustManager!`

In this case the application invoked SSLContext.init four times which means it verified four different certs (two of which were used by 3rd party tracking libs).

Download here: [frida-android-repinning_sa.js](https://techblog.mediaservice.net/wp-content/uploads/2017/07/frida-android-repinning_sa-1.js) or from Frida Codeshare here [https://codeshare.frida.re/@pcipolloni/universal-android-ssl-pinning-bypass-with-frida/](https://codeshare.frida.re/@pcipolloni/universal-android-ssl-pinning-bypass-with-frida/)

Frida & Android [https://www.frida.re/docs/android/](https://www.frida.re/docs/android/)

Written by: Piergiovanni Cipolloni on July 25, 2017.