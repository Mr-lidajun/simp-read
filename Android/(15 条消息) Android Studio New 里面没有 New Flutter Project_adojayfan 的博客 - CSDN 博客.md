> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/adojayfan/article/details/89031162)

跟着 Flutter 中文网的配置教程，安装好了 flutter, 在 Android studio 里面也安装了 dart 和 flutter 的插件。重启后还是在 FIle->New 里面没有显示 New Flutter Project。

反复卸载重装 dart 和 flutter 插件好几次，依然没有效果。  
最后在教程下面的评论里面找到解决方案。  
[https://flutterchina.club/get-started/editor/](https://flutterchina.club/get-started/editor/)

原来是没有把 Android APK Support 插件启用。  
由于某些原因以前把 Android APK Support 关闭过。。。  
![](https://img-blog.csdnimg.cn/20190404174656663.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Fkb2pheWZhbg==,size_16,color_FFFFFF,t_70)  
勾选上后，重启 Android Studio，就在 New 中显了 New Flutter Project.  
![](https://img-blog.csdnimg.cn/20190404175106423.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Fkb2pheWZhbg==,size_16,color_FFFFFF,t_70)