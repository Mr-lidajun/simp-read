> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [bbs.pediy.com](https://bbs.pediy.com/thread-251063.htm)

一、概述
====

        拖延症晚期，一枚小菜鸟终于完成对炫舞梦工厂 APP 的分析。该直播 APP 采用 TCP 协议，TCP 连接建立之后，首先进行基础连接认证，认证通过之后，进行帐号认证，完成即可进行获取角色信息、进入房间等各类操作。发送数据先进行 ProtoBuf 序列化，接着采用 CRC32 循环加密，添加包头（包括命令号以及长度、校验位等）之后，发送，接受到的数据反之。本文主要阐述逆向中遇到的难点及思路、基础连接认证过程、数据包的序列化过程、CRC32 循环加密过程以及重要数据包的分析过程等，旨在提供 APP 逆向中，TCP 通信协议的一般性分析研究方法，如果有更好的思路或者经验交流，欢迎联系。

        本文使用到的工具有：JEB2.3.13、GDA3.53、IDA7.0、雷电模拟器、炫舞梦工厂 APP。

二、 逆向难点及思路
==========

###         1. 寻找切入点

        一般来说，一个好的切入点可以让我们事半功倍。接触到 APP 之后，首先进行了流程分析。开启 android monitor，进行登录操作。通过分析日志以及调用方法堆栈，发现一条日志信息很可疑：

```
I/SystemLog(20023): [CallCenter[Net]J](handleCEventVideoInitConnectionResponse) : initConnectionRes ok... is_force_ver_up:true tunnelId:0 roomId:0

```

其中，is_force_ver_up 表示是否强制更新，正好对应了手机上面要求强制更新。通过 JEB 逆向 app 之后，搜索相关信息：handleCEventVideoInitConnectionResponse，发现如下：

```
if(v0 == 0) {
                z.c("CallCenter[Net]", "(handleCEventVideoInitConnectionResponse) : initConnectionRes ok... is_force_ver_up:" + arg5.h + " tunnelId:" + this.s() + " roomId:" + this.u());
                if(arg5.h) {
                    z.d("CallCenter", "(handleCEventVideoInitConnectionResponse) initConnectionRes.is_force_ver_up current_version:" + arg5.i);
                    com.h3d.qqx5.model.video.version_update.c.a().a(arg5.b());
                    com.h3d.qqx5.model.video.version_update.c.a().i();
                    return;
                }

```

有理由相信，z 即为所需的日志类。有这么一个入手点，就好办很多。一般 app 都会对日志类进行重写，并且在发布版本时，关闭部分信息的输出。采用 xposed 对 z 类所有方法进行 hook，打印出日志信息，可以发现，日志输出了很多有用的信息。通过对日志信息的分析，搜索日志标签：NetConnection，分析发现 com.h3d.qqx5.framework.d.g 即为 app 的网络操作类，在该类中，进行了连接之后的基础连接认证，以及后续的数据包的发送接收等操作。也发现了具体的发送数据包的函数

```
    private void b(h arg5) {
        arg5.a();
        OutputStream v0 = this.c.getOutputStream();
        v0.write(arg5.d().k().array());
        v0.write(arg5.c().k().array(), 0, arg5.a);
    }

```

与此同时，日志也明确告诉 TCP 连接的 IP 和端口，方便抓包分析。

```
I/SystemLog(20023): [CallCenter[Net]J](connectImpl) : try to connect ip:rp.mgc.qq.com port:31000

```

找到发包函数，即有了突破点。一层一层向上追溯即可找到调用层次。

###         2. 代码混淆

        该 APP 代码做了混淆，对于分析协议的过程中十分不利。我的思路为跟踪方法过程，并及时进行重命名，重命名的过程中，我的命名规则是：原方法名_现方法名，之所以保留原方法名，是方便后续的 hook 操作。比如上述发包函数，在我对此方法详细分析之后，其效果如下：

```
    private void b_sendMessage(h arg5) {
        arg5.a_addHeader();
        OutputStream m_outputStream = this.c_socket.getOutputStream();
        m_outputStream.write(arg5.d().k_buffer().array());
        m_outputStream.write(arg5.c().k_buffer().array(), 0, arg5.a);
    }

```

可读性增强很多，慢慢由点及面，从日志入手，各处分析，最后零零散散，即可大致分析出整体 app 通信流程。

三、基础连接认证
========

通过抓包分析以及对反编译之后的代码分析，关键认证代码在 com.h3d.qqx5.framework.d.g.b.a：

```
        void a() {
            int v3 = 0x20;
            h v0 = new h();
            f_NetBuffer netBuffer = v0.c();
            netBuffer.b_putInt(9);
            netBuffer.a_putLong(0);
            netBuffer.a_putLong(0);//以上完成组包
            g.a_send(this.a, v0);//发送数据包
            int v0_1 = this.a.c_recv().c().e_getInt();
            if(v0_1 != 10) {//看第一个int是否是10
                throw new IOException("protocol error:" + v0_1);
            }
 
            if(this.f) {
                v0 = this.a.c_recv();//接收数据包
                int v1_1 = v0.c().e_getInt();//看第一个int是否是1
                if(v1_1 != 1) {
                    throw new IOException("protocol error:" + v1_1);
                }
                else {
                    byte[] v6 = new byte[v3];
                    v0.c().k_buffer().get(v6, 0, v3);
                    long v0_2 = this.a_SolvePuzzle(v0.c().f_getLong(), v0.c().f_getLong(), v6);//完成计算
                    h v2 = new h();
                    f_NetBuffer v3_1 = v2.c();
                    v3_1.b_putInt(2);
                    v3_1.a_putLong(v0_2);
                    g.a_send(this.a, v2);
                    v0_1 = this.a.c_recv().c().e_getInt();
                    if(v0_1 != 3) {
                        throw new IOException("puzzel error:" + v0_1);
                    }
                    else {
                        v0 = new h();
                        netBuffer = v0.c();
                        netBuffer.b_putInt(5);
                        netBuffer.a(Long.toString(this.d));
                        g.a_send(this.a, v0);
                        v0 = new h();
                        netBuffer = v0.c();
                        netBuffer.b_putInt(6);
                        netBuffer.b_putInt(200);
                        k v2_1 = new k();
                        v2_1.b = ((int)this.d);
                        System.arraycopy(this.e, 0, v2_1.d, 0, this.e.length);
                        v2_1.c = 0;
                        v2_1.a(netBuffer);
                        g.a_send(this.a, v0);
                        v0_1 = this.a.c_recv().c().e_getInt();
                        if(v0_1 != 7) {
                            throw new IOException("verify error :" + v0_1);
                        }
                    }
                }
            }
        }

```

  
得知建立连接之后基础连接认证流程，对比数据包进行分析：  

protocal：

```
send：09 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00

```

```
recv：1C 00 00 00 38 F3 01 64 0A 00 00 00 A8 86 0A 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 34 00 00 00 79 F3 56 71 01 00 00 00 57 F4 7C 1A 35 90 CD DA C7 F3 2A BF 06 6F EF 91 5C BC 6D A8 75 F8 02 81 01 C2 8B E6 2C 45 FE 12 D8 F4 F7 15 1A 7C 35 06 DA F4 F7 15 1A 7C 35 06

```

接收到的数据包第一个四字节为长度，所有整数均为大端存储，第二个四字节无意义，下文的 send 以及 recv 同理。之后列出来的数据包都是从第九个字节开始。

```
1C 00 00 00 //非此数据则出错，protocol error
38 F3 01 64 0A 00 00 00 A8 86 0A 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 34 00 00 00 79 F3 56 71 //无意义
01 00 00 00 //非此数据则出错，protocol error
57 F4 7C 1A 35 90 CD DA C7 F3 2A BF 06 6F EF 91 5C BC 6D A8 75 F8 02 81 01 C2 8B E6 2C 45 FE 12 //long_key_sha-256
D8 F4 F7 15 1A 7C 35 06 //long_1
DA F4 F7 15 1A 7C 35 06 //long_1

```

从 long_1 到 long_2 依次计算其 sha-256 值，直至结果与数据包返回的 long_key_sha-256 值相同，此处将符合条件的值命名为 long_keypuzzle：

```
send:02 00 00 00 D8 F4 F7 15 1A 7C 35 06
其中，02 00 00 00 //固定
D8 F4 F7 15 1A 7C 35 06 //上文计算得到的long_key

```

```
recv: 03 00 00 00 EC AA 33 74 42 FA CD D5
其中，03 00 00 00 //表示成功
EC AA 33 74 42 FA CD D5 //无意义

```

verify：

```
send：05 00 00 00 09 00 00 00 39 30 39 37 38 32 37 32 32 00 2C 03 00
其中，05 00 00 00 //固定
09 00 00 00 //帐号长度
39 30 39 37 38 32 37 32 32 //string_帐号
00 //固定

```

```
两个数据包其实在一起，这儿讲解方便，因此分开
send：06 00 00 00 C8 00 00 00 00 00 00 00 C2 2E 3A 36 ...省略796个0字节
其中，06 00 00 00 //固定
C8 00 00 00 00 00 00 00 //固定
C2 2E 3A 36 //int_帐号
其实后面的数据，从代码上看，是有含义的，但是实际抓包发现，始终全部为0

```

```
recv:07 00 00 00 //返回该值，说明成功

```

  
至此，tcp 基础连接认证成功！可以进行后续操作。

三、数据序列化过程
=========

这部分是最让人难受的一部分，因为一直以为是谷歌的 protobuf，尝试套上去解数据包，结果死活不行。app 解数据包这儿流程又比较复杂，加上混淆，理解了好久才理解明白流程。  
以 CEventQueryVideoAccountInfo 该事件的发送以及接受处理为例说明首先，java 有一个类，即为 CEventQueryVideoAccountInfo 类，

```
public class d extends d_INetMessage {
    @n(a=1) public int a;
    @n(a=2) public int b;
    @n(a=3) public int c;
    @n(a=4) public String d;
    @n(a=5) public String e;
 
    public d() {
        super();
    }
 
    public int h() {
        return 60001; //即为61 EA 00 00
    }
 
    public String toString() {
        return "CEventQueryVideoAccountInfo [m_time_stamp=" + this.a + ", m_device_type=" + this.b + ", m_appid=" + this.c + ", m_skey=" + this.d + ", m_open_id=" + this.e + "]";
    }
}

```

通过代码，可以得知该类的功能为 CEventQueryVideoAccountInfo，该类的属性虽然被混淆，但是根据日志，我们可以还原各个属性的真实含义。首先，先设置各个属性的值，接着进行序列化。序列化之后的数据如下：

```
63 00 00 00 09 00 00 11 00 02 19 00 D6 F6 AE 9E 08 20 00 25 00 00 00 20 00 00 00 34 44 34 30 43 33 43 45 35 36 45 36 42 32 35 36 37 42 45 32 43 42 31 43 45 32 33 45 37 31 37 37 00 28 00 25 00 00 00 20 00 00 00 36 30 35 31 30 33 46 33 46 33 33 34 36 42 46 39 31 35 43 45 43 33 31 45 33 34 43 44 42 35 30 44 00 

```

  

```
解析如下：
63 00 00 00 //数据包长度
09 00 //代表了index和type，即索引号和类型，下面同理。这儿对比谷歌的protobuf即可理解
    00 //timestamp:0 //凡是值为整数形式的，均采用了压缩整数，但是和谷歌的protobuf的压缩算法不太一样。
11 00 
    02 //device_type:1
19 00 
    D6 F6 AE 9E 08 //appid：1105583531
20 00 
    25 00 00 00 //长度
        20 00 00 00 //长度
            34 44 34 30 43 33 43 45 35 36 45 36 42 32 35 36 37 42 45 32 43 42 31 43 45 32 33 45 37 31 37 36 
            //skey：4D40C3CE56E6B2567BE2CB1CE23E7176
        00 
28 00 
    25 00 00 00 
        20 00 00 00 
            35 30 35 31 30 33 46 33 46 33 33 34 36 42 46 39 31 35 43 45 43 33 31 45 33 34 43 44 42 35 30 44 
            //openid：505103F3F3346BF915CEC31E34CDB50D
    00 

```

序列化完成之后，进行加密，加密之后添加包头

```
61 EA 00 00 //clsid，命令号
AF DE AB AC //security_flag，固定
00 00 00 00 //checksum，固定
67 00 00 00 //密文长度
2D 61 BC 00 55 E8 79 63 C9 51 39 50 8F 10 3E BC D3 1E 4A 2E 51 ED 5D F5 A2 D1 0D 05 13 EF 99 CC 0B 4C 42 C6 8D 23 BB 7D 83 06 CF 1A C2 CD 17 28 AC E1 02 11 74 B2 1A E1 AD DD D4 64 D5 1D BA B4 33 97 36 F6 7A E0 72 D6 E4 12 AD CD 82 31 C1 AA C1 B6 69 E5 82 2B 08 7F 4B AB 2C 69 C2 45 10 81 1E 53 6E 6E 1F 21 0C //密文

```

  
整数型压缩与解压算法和加密与解密算法随后给出  
  
接下来先讨论返回的数据  
java 同样定义了返回的数据类

```
public class e extends d_INetMessage {
    @n(a=1) public int a;
    @n(a=2) public boolean b;
    @n(a=3) public ArrayList c;
    @n(a=4) public ArrayList d;
    @n(a=5) public int e;
    @n(a=6) public ag f;
    @n(a=7) public long g;
    @n(a=8) public boolean h;
 
    public e() {
        super();
    }
 
    public int h() {
        return 60002; //即为62 EA 00 00
    }
 
    public String toString() {
        return "CEventQueryVideoAccountInfoRes [m_time_stamp=" + this.a + ", m_succ=" + this.b + ", m_room_proxy_infos=" + this.c + ", m_account_infos=" + this.d + ", m_err_code=" + this.e + ", m_last_login_acc=" + this.f + ", m_qq=" + this.g + ", m_is_also_anchor=" + this.h + "]";
    }
}

```

收到数据：

```
62 EA 00 00 AF DE AB AC 00 00 00 00 E4 0C 00 00 AE 6D BC 00 E4 08 8F 95 84 B9 2C 4A 23 F1 C8 26 B7 1E EB F6 4B 67 0D 5E 3B D7 B4 14 C8 3C DB 1C 9A 45 01 72 0C 74 37 ......数据太长，后面不放出来了。

```

与发送数据正好相反

```
62 EA 00 00 //clsid
AF DE AB AC //security_flag
00 00 00 00 //checksum
E4 0C 00 00 //密文长度
AE 6D BC 00 E4 08 8F 95 84 B9 2C 4A 23 F1 C8 26 B7 1E EB F6 4B 67 0D 5E 3B D7 B4 14 C8 3C DB 1C 9A 45 01 72 0C 74 37 //密文

```

  
客户端拿到数据之后，首先获取 clsid 号，接着在一个 hashmap 中寻找对应的类，找到之后，通过反射获取类的所有属性，从而完成解析。解析之后数据如下：

```
CEventQueryVideoAccountInfoRes [m_time_stamp=0, m_succ=true, m_room_proxy_infos=[RoomProxyInfo [name=华南一区, provider=, group=, recommend=false, zoneid=110, addresses=[], channel=0], RoomProxyInfo [name=华南二区......数据太长，截取部分

```

  
需要注意的是，并不是所有的数据结构类都会重写 toString() 方法帮助我们理解各个属性的真实含义，同样有很多类是没有重写该方法的。因此，我们需要尽可能重命名我们知道真实含义的每个值，方便以后遇到没有重写 toString() 方法时，对其属性的含义完成猜测和验证。  

四、压缩和解压算法，加密与解密算法
=================

  
1. 压缩解压算法如下：

```
    //long转为var_int
    private static void a_to_var_int(long arg6, f_NetBuffer arg8) {
        byte v2;
        long v0 = arg6 << 1 ^ arg6 >> 0x3F;
        while(true) {
            v2 = ((byte)(((int)(0x7F & v0))));
            v0 >>>= 7;
            if(v0 == 0) {
                break;
            }
 
            arg8.a_putByte(((byte)(v2 | 0x80)));
        }
 
        arg8.a_putByte(v2);
    }

```

```
    //var_int转为long
    private static long a_from_var_int(f_NetBuffer arg8) {
        long v2 = 0;
        int v0 = 0;
        do {
            int v1 = arg8.g_getByte();
            v2 += ((((long)v1)) & 0x7F) << v0;
            v0 += 7;
        }
        while((v1 & 0x80) != 0);
 
        return v2 >>> 1 ^ -(v2 & 1);
    }

```

2. 加密解密算法：  

```
    //加密代码片段，arg4为初始密钥，12345678，arg5为明文，arg6为长度
    static void a(int arg4, byte[] arg5, int arg6) {
        int v0;
        for(v0 = 0; v0 < arg6; v0 += 4) {
            int v1 = v0;
            arg5[v1] = ((byte)(arg5[v1] ^ (((byte)arg4))));
            v1 = v0 + 1;
            arg5[v1] = ((byte)(arg5[v1] ^ (((byte)(arg4 >> 8)))));
            v1 = v0 + 2;
            arg5[v1] = ((byte)(arg5[v1] ^ (((byte)(arg4 >> 16)))));
            v1 = v0 + 3;
            arg5[v1] = ((byte)(arg5[v1] ^ (((byte)(arg4 >> 24)))));
            arg4 = l.a_GetCRC32(arg5, v0, 4);
        }
    }

```

  

```
    //解密代码片段，arg5为初始密钥，12345678，arg6为密文，arg7为长度
    static void b(int arg5, byte[] arg6, int arg7) {
        int v0 = 0;
        while(v0 < arg7) {
            int v1 = l.a_GetCRC32(arg6, v0, 4);
            int v2 = v0;
            arg6[v2] = ((byte)(arg6[v2] ^ (((byte)arg5))));
            v2 = v0 + 1;
            arg6[v2] = ((byte)(arg6[v2] ^ (((byte)(arg5 >> 8)))));
            v2 = v0 + 2;
            arg6[v2] = ((byte)(arg6[v2] ^ (((byte)(arg5 >> 16)))));
            v2 = v0 + 3;
            arg6[v2] = ((byte)(arg6[v2] ^ (((byte)(arg5 >> 24)))));
            v0 += 4;
            arg5 = v1;
        }
    }

```

  

五、重要数据包分析
=========

        在数据序列化一节已经以 QueryVideoAccountInfo 为例进行了阐述。在抓包之后，想要完成对数据包的详细分析，我采用 xposed 和 jeb 分析相结合的方法。我们已经知道，数据包开头即为 clsid 号，对应了一个 java 类，我们可以在 JEB 的 disassembly 页面直接搜索相关的十六进制或者十进制的值来定位关键类，同时，我们也可以通过 xposed，在其序列化和反序列化的入口，拦截这个类，打印其方法名，从而得到其具体位置。  
假设我们现在有一个数据包，其 clsid 为 D1 A0 00 00。我们想要找到其实现类。  
通过 xposed 日志，我们搜索数据包发送的 clsid: D1 A0 00 00接着找到 SaveStream_param_0: com.h3d.qqx5.model.s.a.c进入 com.h3d.qqx5.model.s.a.c 类，我们发现并没有重写 toString() 方法，无法准确得知各个属性的含义：

```
public class c extends d_INetMessage {
    @n(a=1) public String a;
    @n(a=2) public String b;
    @n(a=3) public String c;
    @n(a=4) public String d;
    @n(a=5) public String e;
    @n(a=6) public int f;
    @n(a=7) public int g;
    @n(a=8) public boolean h;
 
    public c() {
        super();
        this.h = false;
    }
 
    public int h() {
        return 0xA0D1;
    }
}

```

利用 jeb 交叉引用功能，以及之前对其他类的各种注释以及猜测，我们尽可能的完成了对属性的含义的解析

```
public class c extends d_INetMessage {
    @n(a=1) public String a_open_id;
    @n(a=2) public String b_open_key;
    @n(a=3) public String c_pay_token;
    @n(a=4) public String d_pf;
    @n(a=5) public String e_pf_key;
    @n(a=6) public int f_save_num;
    @n(a=7) public int g_device_type;
    @n(a=8) public boolean h_0; //其含义并没有分析出来，但是分析发现其固定为false，也就是0
 
    public c() {
        super();
        this.h_0 = false;
    }
 
    public int h() {
        return 0xA0D1;
    }
}

```

分析出其含义，我们对于数据包的组包和解包也就有了很大的信息。  
下面给出 xposed 代码：

```
public class qqx5 {
    public static void qqx5Hook(final XC_LoadPackage.LoadPackageParam loadPackageParam) throws ClassNotFoundException{
        if (!loadPackageParam.packageName.contains("com.tencent.tmgp.MGCForAndroid")) return;
        XposedBridge.log("Loaded app: " + loadPackageParam.packageName);
         
        //该函数应该是一个日志类，此处hook该类可以查看关键信息
        XposedHelpers.findAndHookMethod("com.tencent.open.a.f", loadPackageParam.classLoader, "c", String.class, String.class, new XC_MethodHook(){
            @Override
            protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
                //XposedBridge.log("日志类*****");
                String str_1 = param.args[0].toString();
                if((!str_1.contains("HotUpdate")) && (!str_1.contains("GiftConfig"))){
                    XposedBridge.log("日志类:" + str_1 + ":" + param.args[1].toString());
                }
            }
        });
         
        //另一个日志类
        XposedHelpers.findAndHookMethod("com.h3d.qqx5.utils.z", loadPackageParam.classLoader, "b", String.class, String.class, new XC_MethodHook(){
            @Override
            protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
                XposedBridge.log("日志类*****");
                String str_1 = param.args[0].toString();
                if((!str_1.contains("HotUpdate")) && (!str_1.contains("GiftConfig"))){
                    XposedBridge.log("日志类:" + str_1 + ":" + param.args[1].toString());
                }
            }
        });
        XposedHelpers.findAndHookMethod("com.h3d.qqx5.utils.RSAUtil", loadPackageParam.classLoader, "encrypt", int.class, String.class, int.class, int.class, new XC_MethodHook(){
            @Override
            protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
                XposedBridge.log("RSAUtilencrypt*****");
                XposedBridge.log("RSAUtilencrypt_param_0:" + param.args[0].toString());
                XposedBridge.log("RSAUtilencrypt_param_1:" + param.args[1].toString());
                XposedBridge.log("RSAUtilencrypt_param_2:" + param.args[2].toString());
                XposedBridge.log("RSAUtilencrypt_param_3:" + param.args[3].toString());
            }
            @Override
            protected void afterHookedMethod(MethodHookParam param) throws Throwable {
                XposedBridge.log("RSAUtilencrypt_return:" + param.getResult().toString());
            }
        });
      //发送数据包加密与解密
        XposedHelpers.findAndHookMethod("com.h3d.qqx5.framework.d.j", loadPackageParam.classLoader, "b", int.class, byte[].class,int.class, new XC_MethodHook(){
            @Override
            protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
                XposedBridge.log("decrypt*****");
                XposedBridge.log("decrypt_param_0:" + param.args[0].toString());
                XposedBridge.log("decrypt_param_1:" + utils.bytesToHexFun1((byte[])param.args[1]));
                XposedBridge.log("decrypt_param_2:" + param.args[2].toString());
            }
            @Override
            protected void afterHookedMethod(MethodHookParam param) throws Throwable {
                XposedBridge.log("decrypt_return:" + utils.bytesToHexFun1((byte[])param.args[1]));
            }
        });
        XposedHelpers.findAndHookMethod("com.h3d.qqx5.framework.d.j", loadPackageParam.classLoader, "a", int.class, byte[].class,int.class, new XC_MethodHook(){
            @Override
            protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
                XposedBridge.log("encrypt*****");
                XposedBridge.log("encrypt_param_0:" + param.args[0].toString());
                XposedBridge.log("encrypt_param_1:" + utils.bytesToHexFun1((byte[])param.args[1]));
                XposedBridge.log("encrypt_param_2:" + param.args[2].toString());
            }
            @Override
            protected void afterHookedMethod(MethodHookParam param) throws Throwable {
                XposedBridge.log("encrypt_return:" + utils.bytesToHexFun1((byte[])param.args[1]));
            }
        });
         
        Class NetBufferClass = loadPackageParam.classLoader.loadClass("com.h3d.qqx5.framework.d.f");
        
        XposedHelpers.findAndHookMethod("com.h3d.qqx5.framework.d.j", loadPackageParam.classLoader, "a", Object.class, NetBufferClass, new XC_MethodHook(){
            @Override
            protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
                String str = param.args[0].getClass().getName();
                XposedBridge.log("SaveStream_param_0:" + str);
            }
        });
        XposedHelpers.findAndHookMethod("com.h3d.qqx5.framework.d.j", loadPackageParam.classLoader, "b", Object.class, NetBufferClass, new XC_MethodHook(){
            @Override
            protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
                String str = param.args[0].getClass().getName();
                XposedBridge.log("LoadStream_param_0:" + str);
            }
        });
         
        XposedHelpers.findAndHookMethod("com.h3d.qqx5.framework.d.h", loadPackageParam.classLoader, "c", new XC_MethodHook(){
            @Override
            protected void afterHookedMethod(MethodHookParam param) throws Throwable {
                Method rpcMethod = param.getResult().getClass().getMethod("k");
                ByteBuffer response = (ByteBuffer) rpcMethod.invoke(param.getResult());
                byte[] bytes = response.array();
                XposedBridge.log("m_body_buffer_return:" + utils.bytesToHexFun1(bytes));
            }
        });
         
        XposedHelpers.findAndHookMethod("com.h3d.qqx5.framework.d.h", loadPackageParam.classLoader, "d", new XC_MethodHook(){
            @Override
            protected void afterHookedMethod(MethodHookParam param) throws Throwable {
                Method rpcMethod = param.getResult().getClass().getMethod("k");
                ByteBuffer response = (ByteBuffer) rpcMethod.invoke(param.getResult());
                byte[] bytes = response.array();
                XposedBridge.log("m_header_buffer_return:" + utils.bytesToHexFun1(bytes));
            }
        });
    }
 
}

```

  
全部完毕。  
这次 app 逆向中，学习到了很多知识，该 app 采用了 java 完成了序列化和反序列化，让我知道反射还可以这样玩，深入理解了基础连接认证，实际上安全隐患很大，因为其数据包并没有采用安全的密钥体系，中间人随意窃听。深入理解了其加密解密流程，整数的压缩与解压流程，感觉自己的功力又深了一层等等等等。。收货蛮大。  
彩蛋环节：在分析 app 的过程中，发现在帐号的认证过程中，也就是 CEventQueryVideoAccountInfo 数据包，account，skey，openid 三者应该对应才可以成功登陆帐号，进而对帐号进行后续操作。实际发现，服务器对于 skey 与 openid 并没有校验，也就是说任意帐号即可登录炫舞梦工厂 app。已经和腾讯安全应急中心报告了该漏洞，并且已经完成了修复。顺便说一句，腾讯是真的抠门。。  
  
  

[[招聘] 欢迎你加入看雪团队！](https://job.kanxue.com/company-read-31.htm)

最后于 2019-4-27 18:25 被小堆编辑 ，原因：