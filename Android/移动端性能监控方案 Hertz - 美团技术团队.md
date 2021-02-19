> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [tech.meituan.com](https://tech.meituan.com/2016/12/19/hertz.html)

性能问题是造成 App 用户流失的罪魁祸首之一。App 的性能问题包括崩溃、网络请求错误或超时、响应速度慢、列表滚动卡顿、流量大、耗电等等。而导致 App 性能低下的原因有很多，除去设备硬件和软件的外部因素，其中大部分是开发者错误地使用线程、锁、系统函数、编程范式、数据结构等导致的。即便是最有经验的程序员，也很难在开发时就能避免所有导致性能低下的 “坑”，因此解决性能问题的关键是在于能不能尽早地发现和定位这些 “坑”。

美团外卖在实践中通过总结常见性能问题，并在学习了业内微信、360 等性能监控技术原理后，开发了一套移动端性能监控解决方案——**Hertz（赫兹）**。Hertz 的目标是实现这三个功能：

*   开发时期，检查性能异常点并通知给开发者；
*   测试时期，和现有测试工具结合产生性能测试报告；
*   上线时期，通过监控平台上报性能数据，实现线上问题定位和追查。

![](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2016/8b42fbbd.png)

要实现这三个功能，首先要采集到可衡量、有价值的性能数据，因此性能数据的采集是我们关注的最核心的问题之一。

虽然用户可以感知到的性能问题多种多样，我们仍然可以将其抽象成具体的监控指标。在 Hertz 中这些监控指标包括：**FPS、CPU 使用率、内存占用、卡顿、页面加载时间、网络请求流量** 等。这其中有的性能指标比较容易获取，例如 FPS、CPU 使用率、内存占用等，有的性能指标不易获取，例如卡顿、页面加载时间、网络请求流量等。

例如在 iOS 中我们可以这样获取 FPS：

```
- (void)tick:(CADisplayLink *)link
{
    NSTimeInterval deltaTime = link.timestamp - self.lastTime;
    self.currentFPS = 1 / deltaTime;
    self.lastTime = link.timestamp;
}


```

在 Android 中我们可以这样获取内存占用：

```
public long useSize() {
    Runtime runtime = Runtime.getRuntime();
    long totalSize = runtime.maxMemory() >> 10;
    this.memoryUsage = (runtime.totalMemory() - runtime.freeMemory()) >> 10;
    this.memoryUsageRate = this.memoryUsage * 100 / totalSize;
}


```

上面的例子只是为了说明获取 FPS、内存、CPU 这些指标非常简单，但是这些指标必须与其它数据结合才具有意义，这些数据包括**当前页面的信息、当前 App 运行时间，或者卡顿发生时程序执行的堆栈和运行日志**等等。例如：CPU 和当前页面信息结合，可以评测每个页面的运算复杂度；内存和 App 运行时间结合，可以观察内存和使用时长的关系进而分析是否发生内存泄漏；FPS 和卡顿信息结合，可以评估这次卡顿发生时 App 的性能究竟下降到什么程度。

流量消耗
----

移动端用户对于流量非常敏感，美团外卖偶尔会收到用户投诉说短时间内消耗了巨大流量的问题，因此我们思考能不能在 App 本地统计用户的流量消耗，并且上报给后台。这个统计不必精确到每个 API，能够粗略地归类计算出总的流量消耗即可。我们对于流量统计的维度是：**自然日 + 请求来源 + 网络类型**。为什么有了服务端流量监控（例如 CAT），还需要在客户端本地监控流量呢？本地流量能够统计由用户端发出的全部网络请求，而这点服务端监控是很难做到的。一个例子是并非所有的网络请求都会上报服务端监控；另一个例子是由于网络原因可能造成用户仅仅消耗了上行流量，但这些请求并没有到服务端。

在 iOS 中我们通过注册 NSURLProtocol 实现流量统计：

```
- (void)connectionDidFinishLoading:(NSURLConnection *)connection
{
    [self.client URLProtocolDidFinishLoading:self];

    self.data = nil
    if (connection.originalRequest) {
        WMNetworkUsageDataInfo *info = [[WMNetworkUsageDataInfo alloc] init]
        self.connectionEndTime = [[NSDate date] timeIntervalSince1970]
        info.responseSize = self.responseDataLength;
        info.requestSize = connection.originalRequest.HTTPBody.length
        info.contentType = [WMNetworkUsageURLProtocol getContentTypeByURL:connection.originalRequest.URL andMIMEType:self.MIMEType];
    [[WMNetworkMeter sharedInstance] setLastDataInfo:info]
    [[WMNetworkUsageManager sharedManager] recordNetworkUsageDataInfo:info]
}


```

}

在 Android 中我们通过基于 Aspectj 的 AOP 方式拦截网络请求 API 实现流量统计：

```
@Pointcut("target(java.net.URLConnection) && " +
        "!within(retrofit.appengine.UrlFetchClient) " +
        "&& !within(okio.Okio) && !within(butterknife.internal.ButterKnifeProcessor) " +
        "&& !within(com.flurry.sdk.hb)" +
        "&& !within(rx.internal.util.unsafe.*) " +
        "&& !within(net.sf.cglib..*)" +
        "&& !within(com.huawei.android..*)" +
        "&& !within(com.sankuai.android.nettraffic..*)" +
        "&& !within(roboguice..*)" +
        "&& !within(com.alipay.sdk..*)")
protected void baseCondition() {

}

@Pointcut("call (org.apache.http.HttpResponse org.apache.http.client.HttpClient.execute(org.apache.http.client.methods.HttpUriRequest))"
        + "&& target(org.apache.http.client.HttpClient)"
        + "&& args(request)"
        + "&& !within(com.sankuai.android.nettraffic.factory..*)"
        + "&& baseClientCondition()"
)
void httpClientExecute(HttpUriRequest request) {

}


```

统计到总的流量消耗后，我们还希望对流量进行粗略的归类，方便定位问题。有两个因素是我们关心的：第一是请求来源，即流量消耗是来自 API 请求，H5 还是 CDN 的；第二是网络类型，即 Wifi、4G 还是 3G 流量。对于流量来源，我们首先通过域名做下简单的归类。以 iOS 为例，示例代码如下：

```
- (NSString *) regApiHost {
    return _regApiHost ? _regApiHost :@"^(.*\\.)?(meituan\\.com|maoyan\\.com|dianping\\.com|kuxun\\.cn)$";
}

- (NSString *) regResHost {
    return _regResHost ? _regResHost : @"^(.*\\.)?(meituan\\.net|dpfile\\.com)$";
}

- (NSString *) regWebHost {
    return _regWebHost ? _regWebHost : @"^(.*\\.)?(meituan\\.com|maoyan\\.com|dianping\\.com|kuxun\\.cn|meituan\\.net|dpfile\\.com)$";
}


```

但是某些域名可能既部署了 API 服务，又部署了 Web 服务。对于这类域名，我们还通过校验返回包的 MIMEType 作进一步的区分。以 iOS 为例，示例代码如下：

```
+ (BOOL)isPermissiveWebURL:(NSURL *)URL andMIMEType:(NSString *)MIMEType
{
    NSRegularExpression *permissiveHost = [NSRegularExpression regularExpressionWithPattern:[[WMNetworkMeter sharedInstance] regWebHost]
                                                                                options:NSRegularExpressionCaseInsensitive
                                                                                  error:nil];
    NSString *host = URL.host;
    return ([MIMEType isEqualToString:@"text/css"] || [MIMEType isEqualToString:@"text/html"] || [MIMEType isEqualToString:@"application/x-javascript"] || [MIMEType isEqualToString:@"application/javascript"]) && (host && [permissiveHost numberOfMatchesInString:host options:0 range:NSMakeRange(0, [host length])]);
}


```

页面加载时间
------

要测量页面加载时间，我们要解决两个问题。第一，如何衡量一个页面的加载时间；第二，如何尽量不写或少写代码来实现测速。先看第一个问题，以 Android 为例，在 Activity 的创建加载过程中，会执行很多操作，例如设置页面主题，初始化页面布局，加载图片，获取网络数据或读写数据库等等。上述操作的任何一个环节出现性能问题都可能导致画面不能及时显示，影响用户体验。Hertz 将这些可能发生的操作抽象为下图所示的测速模型：

![](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2016/d8f483a5.png)

其中 T1 指页面初始化到第一个 UI 元素显示的时间，这个 UI 元素一般是指数据加载时的等待动画之类的。T2 是指网络请求时间，这个时间的开始点有可能早于 T1 的结束点。T3 是加载到数据后，为 UI 填充数据并重新渲染完成的时间。T 是整个页面从初始化到最终 UI 绘制完成的时间。

对于第二个问题，如果每个时间点都需要人工写代码埋点的话，效率非常低并且容易出错。因此，**Hertz 通过一个配置文件配置每个页面对应的 API，在 API 请求的基类中统一埋点**。这个方案当然还有优化空间，例如 hook 关键节点上的 API 调用注入埋点代码。

```
[{
  "page": "MainActivity",
  "api": [
    "/poi/filter",
    "/home/head",
    "/home/rcmdboard"
  ]
},
{
  "page": "RestaurantActivity",
  "api": [
    "/poi/food"
  ]
}]


```

此外，还有一个问题是如何判定 UI 渲染完成？在 Android 中，Hertz 的做法是在 Activity 的 rootView 中插入一个 FrameLayout，并且监听这个 FrameLayout 是否调用了 dispatchDraw 方法实现的。当然，这个方案的缺点是由于插入了一级 View 导致层级嵌套变深。

```
@Override
protected void dispatchDraw(Canvas canvas) {
    super.dispatchDraw(canvas);
    if (!mIsComplete) {
        mIsComplete = mCallback.onDrawEnd(this, mKey);
    }
}


```

在 iOS 中我们采取了不同的做法，Hertz 在配置文件中指定最终渲染页面的某个元素的 tag，并在网络请求成功后开启 CADisplayLink 检查该元素是否出现在根节点下面。

```
- (void)tick:(CADisplayLink *)link
{
    [_currentTrackRecordArray enumerateObjectsUsingBlock:^(WMHertzPageTrackRecord * _Nonnull record, NSUInteger idx, BOOL * _Nonnull stop) {
        if ([self findTag:record.configItem.tag inViewHierarchy:record.rootView]) {
            [self endPageRenderEvent:record];
        }
    }];
}


```

卡顿
--

目前主流移动设备均采用双缓存 + 垂直同步的显示技术。大概原理是显示系统有两个缓冲区，GPU 会预先渲染好一帧放入一个缓冲区内，让视频控制器读取，当下一帧渲染好后，GPU 会直接将视频控制器的指针指向第二个容器。这里，GPU 会等待显示器的 VSync（即垂直同步）信号发出后，才进行新的一帧渲染和缓冲区更新。

大多数手机的屏幕刷新频率是 60HZ，如果在 1000⁄60=16.67ms 内没有将这一帧的任务执行完毕，就会发生丢帧现象，这便是用户感受到卡顿的原因。这一帧的绘制任务包括 CPU 的工作和 GPU 的工作两部分，CPU 负责计算显示的内容，例如视图创建、布局计算、图片解码、文本绘制等等，随后 CPU 将计算好的内容提交给 GPU，由 GPU 进行变换、合成、渲染。

除了 UI 绘制外，系统事件、输入事件、程序回调服务、以及我们插入的其它代码也都在主线程中执行，那么一旦在主线程里添加了操作复杂的代码，这些代码就有可能阻碍主线程去响应点击、滑动事件，以及阻碍主线程的 UI 绘制操作，这就是造成卡顿的最常见原因。

在了解了屏幕绘制原理和卡顿形成的原因后，很容易想到通过检测 FPS 就可以知道 App 是否发生了卡顿，也能够通过一段连续的 FPS 帧数计算丢帧率来衡量当前页面绘制的质量。然而实践发现 FPS 的刷新频率非常快，并且容易发生抖动，因此直接通过比较通过 FPS 来侦测卡顿是比较困难的。而检测主线程消息循环执行的时间就要容易的多了，这也是业内常用的一种检测卡顿的方法。因此，**Hertz 在实践中采用的就是检测主线程每次执行消息循环的时间，当这一时间大于阈值时，就记为发生一次卡顿。**

![](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2016/141e6c5d.png)

在实践中我们发现，有的卡顿连续性耗时较长，例如打开新页面时的卡顿；而有的卡顿连续性耗时相对较短但频次较快，例如列表滑动时的卡顿。因此，我们采用了 “N 次卡顿超过阈值 T” 的判定策略，即一个时间段内卡顿的次数累计大于 N 时才触发采集和上报：例如卡顿阈值 T=2000ms、卡顿次数 N=1，可以判定为单次耗时较长的卡顿；而卡顿阈值 T=300ms、卡顿次数 N=5，可以判定为频次较快的卡顿。

```
Runnable loopRunnable = new Runnable() {
    @Override
    public void run() {
        if (mStartedDetecting && !isCatched) {
            nowLaggyCount++;
            if (nowLaggyCount >= N) {
                blockHandler.onBlockEvent();
                isCatched = true;
                ...
            }
        }
    }
};

public void onMainLoopFinish(){
    if(isCatched){
        blockHandler.onBlockFinishEvent(loopStartTime,loopEndTime);
    }
    resetStatus();
    ...
}


```

当检测到卡顿后，如何定位到造成卡顿的问题呢？如果能抓取到卡顿发生时程序的调用堆栈和运行日志，是不是很酷？的确，通过抓取堆栈可以非常有效地帮我们定位到造成卡顿的 “问题代码”。

在实践中我们发现抓取堆栈有两个需要注意的问题。

第一个问题是堆栈抓取的时机。抓取堆栈的时机必须是在卡顿发生当时，而不是之后，否则不能准确抓到造成卡顿的代码，因此在子线程中当卡顿还没有结束时，我们就会抓取堆栈。

第二个问题是堆栈如何归类，卡顿堆栈的归类和 Crash 堆栈不同，以最内层代码归类显然是不合适的，因为外层不同的业务逻辑代码在最内层的调用堆栈有可能是相同的。以最外层代码归类也是不合适的，因为最外层代码有可能是业务逻辑代码，也有可能是系统调用。

目前 Hertz 的做法是按照最内层归类的原则，并匹配一些简单的规则，以命中规则的类名来归类。

Hertz 非常重视 SDK 的可扩展性和易用性，在设计之初我们就做了很多考量。SDK 的框架如下图所示，整体上分为三层：最上层是接口层，提供极少量的对外暴露的方法，以及环境和配置参数等。第二层是业务层，包含了页面测速、卡顿检测和参数采集等所有的核心逻辑。第三层是数据适配层，将业务层产生的数据封装为统一的数据结构，并通过适配器适配到不同的输出通道上。

![](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2016/abad6bfc.png)

设计上我们第一个考量就是接口的易用性，Hertz 内置了三种运行模式：开发模式、测试模式和线上模式。开发者只需要指定一种模式，Hertz 就可以开始工作了。各种模式预设了 SDK 运行所需要的参数，例如采样频率、卡顿阈值、上报通道开关等，而监控指标的采集、卡顿的侦测、页面测速等逻辑都在内部自动执行。以 Android 为例，示例代码如下：

```
final HertzConfiguration configuration = new HertzConfiguration.Builder(this)
        .mode(HertzMode.HERTZ_MODE_DEBUG)
        .appId(APP_ID)
        .unionId(UNION_ID)
        .build();
Hertz.getInstance().init(configuration);


```

设计上我们第二个考量是 SDK 的可扩展性。以数据适配层为例，目前内置了五种适配通道，可以将采集到的监控数据适配到不同的数据通道。根据选择的工作模式不同，数据将被适配到服务端监控通道，生成测试报告，或者只在 App 本地输出日志和提示。这种设计带来的一个好处是，**如果需要新增一种数据输出通道，既可以在上层添加一个拦截器，也可以只改动 SDK 极少量的代码来添加一个适配器**。同样的，性能采集模块和页面测速模块的设计也遵循这种思路。

美团外卖在接入 Hertz 后，初步具备了发现、定位性能问题的能力，在开发期、测试期、线上期都对 Hertz 进行了实际验证。

开发期应用
-----

在开发期接入 Hertz，相当于集成了一个离线的性能检测工具，当检测到异常时，Hertz 将这些数据直接反馈给开发者，如下图所示：

![](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2016/786543f3.png)

运行时采集的数据会输出到日志中，而 App 的页面上也会插入一个浮层来展示当前的 FPS、CPU、内存等基本信息。如果检测到卡顿发生，会弹出提示页面并列出当前的执行堆栈。目前从卡顿检测结果来看，大部分堆栈日志可以比较明显的定位到有问题的代码，只要略微查看代码和分析原因，这些问题都能很容易的优化。

下面是初始化复杂 UI 造成卡顿的例子：

```
android.content.res.StringBlock.nativeGetString(Native Method)
android.content.res.StringBlock.get(StringBlock.java:82)
android.content.res.XmlBlock$Parser.getName(XmlBlock.java:175)
android.view.LayoutInflater.inflate(LayoutInflater.java:470)
android.view.LayoutInflater.inflate(LayoutInflater.java:420)
android.view.LayoutInflater.inflate(LayoutInflater.java:371)
com.sankuai.meituan.takeoutnew.controller.ui.PoiListAdapterController.getView(PoiListAdapterController.java:77)
com.sankuai.meituan.takeoutnew.adapter.PoiListAdapter.getView(PoiListAdapter.java:26)
android.widget.HeaderViewListAdapter.getView(HeaderViewListAdapter.java:220)


```

下面是使用 Gson 反向解析字符串时造成卡顿的例子：

```
com.google.gson.Gson.toJson(Gson.java:519)
com.meituan.android.common.locate.util.GoogleJsonWrapper    $MyGson.toJson(GoogleJsonWrapper.java:236)
com.sankuai.meituan.location.collector.CollectorJson    $MyGson.toJson(CollectorJson.java:216)
com.sankuai.meituan.location.collector.CollectorFilter.saveCurrentData(CollectorFilter.java:67)
com.sankuai.meituan.location.collector.CollectorFilter.init(CollectorFilter.java:33)
com.sankuai.meituan.location.collector.CollectorFilter.<init>(CollectorFilter.java:27)
com.sankuai.meituan.location.collector.CollectorMsgHandler.recordGps(CollectorMsgHandler.java:134)
com.sankuai.meituan.location.collector.CollectorMsgHandler.getNewLocation(CollectorMsgHandler.java:81)
com.meituan.android.common.locate.LocatorMsgHandler$1.handleMessage(LocatorMsgHandler.java:29)


```

下面是主线程读写数据库造成卡顿的例子：

```
android.database.sqlite.SQLiteConnection.nativeExecuteForLastInsertedRowId(Native Method)
android.database.sqlite.SQLiteConnection.executeForLastInsertedRowId(SQLiteConnection.java:782)
android.database.sqlite.SQLiteSession.executeForLastInsertedRowId(SQLiteSession.java:788)
android.database.sqlite.SQLiteStatement.executeInsert(SQLiteStatement.java:86)
de.greenrobot.dao.AbstractDao.executeInsert(AbstractDao.java:306)
de.greenrobot.dao.AbstractDao.insert(AbstractDao.java:276)
com.sankuai.meituan.takeoutnew.db.dao.BaseAbstractDao.insert(BaseAbstractDao.java:25)
com.sankuai.meituan.takeoutnew.log.LogDataUtil.insertIntoDb(LogDataUtil.java:243)
com.sankuai.meituan.takeoutnew.log.LogDataUtil.saveLogInfo(LogDataUtil.java:221)
com.sankuai.meituan.takeoutnew.log.LogDataUtil.saveLog(LogDataUtil.java:116)
com.sankuai.meituan.takeoutnew.log.LogDataUtil.saveLogInfo(LogDataUtil.java:112)
com.sankuai.meituan.takeoutnew.ui.page.main.order.OrderListFragment.onPageShown(OrderListFragment.java:306)
com.sankuai.meituan.takeoutnew.ui.page.main.order.OrderListFragment.init(OrderListFragment.java:151)
com.sankuai.meituan.takeoutnew.ui.page.main.order.OrderListFragment.onCreateView(OrderListFragment.java:81)


```

从上报的具体问题来看，大部分日志可以比较明显的定位到有问题的代码，只要略微查看代码和分析原因，这些问题都能很容易优化。

测试期应用
-----

传统的性能测试大多依赖于第三方工具，产生的数据和开发实测的数据有较大出入，此外，这些测试往往只给出一些指标的数据，而不能帮助开发者定位到问题所在。我们在测试阶段使用 Hertz 采集性能数据，测试手段可以是人工测试，也可以是自动化测试或者 monkey 测试。得到性能数据后，通过脚本处理后会发出一个简单的测试报告。

![](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2016/7e321db0.png)

当然这种形式的测试报告仍然需要手工来导出日志和执行脚本，未来我们会在此基础上开发一套自动化的测试工具。

上线期应用
-----

对于卡顿检测，除了在开发期和测试期 Hertz 能立即将问题反馈给开发者外，在灰度或线上运行时 Hertz 也会将数据上传到服务端，目前上报通道是公司内部的 CAT（已经开源，详情请参考[深度剖析开源分布式监控 CAT](http://tech.meituan.com/CAT_in_Depth_Java_Application_Monitoring.html)一文）。可以看到堆栈的归类和展示和我们熟悉的 Crash 监控非常类似，按照前面提到的归类原则，卡顿堆栈按照发生的次数排列，并且可以按照版本、操作系统、设备过滤，比较符合开发者的使用习惯。

![](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2016/027a0f8e.png)

对于流量的统计，我们每天会上报到服务端全网用户的流量消耗数据，并输出一个报表，列出全网流量消耗 Top100 的用户。如果发现异常，可以进一步根据后端日志和客户端诊断日志来排查具体是哪个网络请求导致的流量异常。

![](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2016/87f21f09.png)

对于页面测速数据和 FPS、CPU、内存等基础指标，Hertz 也会将数据上报到 CAT，评测 App 整体的性能状况。

![](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2016/3d9d3cb5.png)

性能优化是每一个成熟的 App 都必须认真对待的话题，而性能优化的痛点往往在于不能及时发现问题，或者发现了问题却不能定位问题。美团外卖以监控数据指导性能优化的思路，在实践中开发和完善了 App 性能监控方案 Hertz，并且在性能数据的监控和应用方面做了一些探索和验证。

目前 Hertz 的监控指标包括了 FPS、CPU 使用率、内存占用、卡顿、页面加载时间、网络请求流量等，而耗电量、App 冷启动，以及 Exception 等监控后续会逐步加入到 Hertz 的监控目标中去。**性能监控的指标在未来可能会复用多个现有工具，并且在此基础上逐步完善**。

Hertz 的卡顿侦测和堆栈抓取能够非常有效地帮助开发者定位性能问题，但是目前的卡顿侦测策略还有很多优化的空间。例如是否可以**根据设备不同设定不同的阈值，以及在 App 运行的不同时期设置不同的策略**。而对于堆栈的归类，目前的规则只是简单地匹配类名前缀，如何更精准、更合理的分类也是我们未来要更多考虑的问题。当然，这些优化还需要更多的数据样本做支撑。

建立可视化的、友好的性能测试工具也同样非常重要，例如一个可实时查看，也可翻阅历史报告的 Web 页面。同时，Hertz 在设计上可以很容易的和自动化测试手段相结合，或者在集成阶段自动生成测试报告，然而在这方面我们才仅仅做了一些初步的尝试。**当我们具备了准确采集性能数据的能力之后，如何更好地应用到包括测试环节在内的整个开发流程中，仍然需要长期的探索和实践**。

本文主要介绍美团外卖在 Hertz 的实践过程中总结的一些思路和实现手段，而围绕 App 性能监控还有很多有趣的，和更深入的主题并没有涉及。例如如何平衡性能监控工具和工具本身所带来的性能问题，性能优化的具体技巧和手段，以及对性能数据做进一步分析从而建立起异常设备的监控体系等等。未来我们也将在这些问题上做进一步探索、实践和分享。

1.  [BlockCanary](https://github.com/markzhai/AndroidPerformanceMonitor).
2.  [Leakcanary](https://github.com/square/leakcanary).
3.  [Watchdog](https://github.com/wojteklu/Watchdog).
4.  [iOS-System-Services](https://github.com/Shmoopi/iOS-System-Services).
5.  guoling, [微信 iOS 卡顿监控系统](http://mp.weixin.qq.com/s/M6r7NIk-s8Q-TOaHzXFNAw).