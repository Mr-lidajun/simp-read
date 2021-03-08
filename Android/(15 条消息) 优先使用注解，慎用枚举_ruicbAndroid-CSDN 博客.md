> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/My_TrueLove/article/details/70519234)

### 文章目录

*   [1. 纯常量场景](#1__15)
*   [2. 分组且存在对应关系](#2__29)
*   [3. 小结](#3__93)
*   *   [3.1 万能的枚举](#31__99)
    *   [3.2 枚举对于性能的损耗](#32__148)
    *   [3.3 寻找替代方案](#33__162)
*   [4. 分组常量](#4__221)
*   *   [4.1 注解一般使用步骤](#41__232)
    *   [4.2 补充](#42__283)
*   [5. 总结](#5__326)

转载请注明出处：

[http://blog.csdn.net/my_truelove/article/details/70519234](http://blog.csdn.net/my_truelove/article/details/70519234)

之前写过一篇 [使用枚举代替常量，简化工作！](http://blog.csdn.net/my_truelove/article/details/52074493)，介绍了使用枚举如何使得程序更加优雅、健壮，该文章一度成为我博客访问量最高的一篇。如果还没看，建议先扫一眼，再继续看本文。

虽然我今天要打脸，介绍如何使用注解，慎用枚举，但其实在之前的文章最后，我很明显的提议大家分场景的使用：

> 最后，声明一点，我所说的使用枚举替换常量，是针对类似于 “常量之间存在关联” 的情况，并不是说以后所有常量都写成枚举，毕竟官方是不推荐使用枚举的。所以在实际开发中，还需要根据实际使用场景去斟酌，杜绝滥用。

为了让读者更好的区分常量、注解和枚举的使用场景，我将分别就不同的场景，为大家介绍相应的使用方案：

*   纯常量场景：常量只是作为全局配置数据使用；
*   分组常量场景：归属于同一分组的常量；
*   分组且存在对应关系的常量场景：常量归属于同一分组，且另一方面常量之间存在对应关系。

1. 纯常量场景
========

这里的常量，主要是用来作为全局的配置项使用，如下 `GlobalContants`：

```
public class GlobalContants {
    public static final int MAX_THREAD_NUM = 3;
    public static final String DOWNLOAD_DIR = "download";
    public static final float PI = 3.14f;
}

```

大概就这个意思，记得大学期间这种方式使用最多了，很基础的东西，不再展开讲了。

2. 分组且存在对应关系
============

我们先来看看 **分组且存在对应关系** 的常量我们应该如何处理，假设你已经看过我那篇推荐枚举的文章了，我就直接说了。

```
public enum TaskStatus {

    UN_KNOW(-1, "未知", "#84807f"),
    UN_START(0, "未开始", "#e2d512"),
    PROGRESSING(1, "进行中", "#12ea2f"),
    COMPLETED(2, "已完成", "#c30910");

    int mCode;
    String mDesc;
    String mColor;

    TaskStatus(int code , String desc , String color){
        mCode = code;
        mDesc = desc;
        mColor = color;
    }

    public static TaskStatus getTaskStatus(int status) {
        for (TaskStatus taskStatus : values()) {
            if (status == taskStatus.mCode) {
                return taskStatus;
            }
        }
        return UN_START;
    }
}

```

上述代码，我们用枚举实现了一个常量分组，且常量之间存在对应关系的功能。

这里的常量分组，指的就是一组任务状态，简化上述代码来看：

```
public enum TaskStatus {
    UN_KNOW,
    UN_START,
    PROGRESSING,
    COMPLETED
}

```

我们一眼就能看出 `TaskStatus` 包含这样一组状态。

而常量间对应关系，指的是：

```
UN_KNOW(-1, "未知", "#84807f"),
UN_START(0, "未开始", "#e2d512"),
PROGRESSING(1, "进行中", "#12ea2f"),
COMPLETED(2, "已完成", "#c30910");

```

以 `UN_KNOW` 状态为例，分别由对应的状态码、描述和色值组成，以此实现常量间的对应关系。

很难想象，在这种场景下，如果你不使用枚举，而是：

*   在 `GlobalContants` 中定义状态码
*   在 `string.xml` 中定义描述
*   在 `colors.xml` 中定义色值

我想，终有一天你会隐约听到代码松动的声音，那真的不是幻觉。

3. 小结
=====

哦，这不是总结，还没完呢。

至此，你已经明白常量和枚举的使用了，确实在对应场景下，像上述那样使用没有任何问题。现在我们梳理下，抛出问题，来引出注解的使用。

3.1 万能的枚举
---------

现在，有这样一个场景，**我们仅仅需要根据一组状态中的某一状态执行相应的操作**，不需要多个常量之间相互对应。

你自然会想到枚举的简化版：

```
public enum TaskStatus {
    UN_KNOW,
    UN_START,
    PROGRESSING,
    COMPLETED
}

```

然后定义了一个任务辅助类：

```
import com.ruicb.enumdemo.TaskStatus;

public class TaskHelper {

    public static void doSth(TaskStatus status){
        switch (status){
            case UN_KNOW:
                //do something
                break;
            case UN_START:
                break;
            case PROGRESSING:
                break;
            case COMPLETED:
                break;
        }
    }
}

```

然后如此调用：

```
TaskHelper.doSth(TaskStatus.COMPLETED);

```

这样使用枚举，十分优雅：

*   保证了类型安全：调用者无法随意传一个 int 值；
*   代码可读性非常高；

但是…

3.2 枚举对于性能的损耗
-------------

事实上你也知道，官方是不建议使用枚举的，这个在文章开头简单说了。

想详细了解的，可以参考该博文：[http://blog.csdn.net/hp910315/article/details/48975655](http://blog.csdn.net/hp910315/article/details/48975655)

同时，官方发布了有关枚举的视频 `The price of ENUMs`，你可以直接看视频：

[YouTube](https://youtu.be/Hzs6OBcvNQE)  
[优酷](http://v.youku.com/v_show/id_XMTMxOTM1NjY1Ng==.html)

总之，与静态常量相比，枚举会增大应用程序编译后的 dex 文件，同时应用在运行时的内存占用也会升高。在资源有限的移动设备上，大量的使用枚举无疑是致命的。

3.3 寻找替代方案
----------

既然如此，我们不使用枚举不就行了吗？反正只是简单的单一状态，我单独定义一个常量类，实现分组：

```
/**
 * 任务状态常量
 */
public class TsakContants {
    public static final int UN_KNOW = -1;
    public static final int UN_START = 0;
    public static final int PROGRESSING = 1;
    public static final int COMPLETED = 2;
}

```

然后修改 `doSth()` 方法：

```
public class TaskHelper {

    public static void doSth(int status){
        switch (status){
            case TsakContants.UN_KNOW:
                //do something
                break;
            case TsakContants.UN_START:
                break;
            case TsakContants.PROGRESSING:
                break;
            case TsakContants.COMPLETED:
                break;
        }
    }
}

```

调用起来也没有区别：

```
TaskHelper.doSth(TsakContants.COMPLETED);

```

上述代码达到了和枚举一样的效果，完全可以不使用枚举。可是，让我再回到使用枚举的好处：

*   保证了类型安全：调用者无法随意传一个 int 值；
*   代码可读性非常高；

反过来对比看，向上面那样使用常量会有什么问题:

*   首先，我们无法保障类型安全，用户可以在调用时传入任何一个 int 值：

```
TaskHelper.doSth(100);

```

*   其次代码可读性很差，IDE 只是提示传入 `int` 类型的参数。此时如果不看方法体，调用者根本不知道该传什么值。

那么问题来了？在一组常量的情况下，我们使用枚举太重，使用常量不安全、可读性差，我们该怎么办？

当然是使用注解了！哈哈，兜了一圈终于轮到注解，是时候为其正名啦。

4. 分组常量
=======

假设你知道注解的用法，不知道也没关系，很简单。比如定义一个注解：

```
public @interface TaskStatus {

}

```

是的，类似于接口的定义，加个 `@` 符号。

4.1 注解一般使用步骤
------------

那么对于上面的枚举和常量都不 work 的场景，如何使用枚举解决呢？

1.  定义注解类，添加常量

```
@Retention(RetentionPolicy.SOURCE)
public @interface TaskStatus {
    int UN_KNOW = -1;
    int UN_START = 0;
    int PROGRESSING = 1;
    int COMPLETED = 2;
}

```

和接口成员一样，注解类的成员默认就是 `public static final` 修饰的。

`@Retention(RetentionPolicy.SOURCE)` 表示告诉编译器，该注解是源代码级别的，生成 class 文件的时候这个注解就被编译器自动去掉了。

2.  修改 `doSth()` 方法：

```
public class TaskHelper {

    public static void doSth(@TaskStatus int status){
        switch (status){
            case TaskStatus.UN_KNOW:
                //do something
                break;
            case TaskStatus.UN_START:
                break;
            case TaskStatus.PROGRESSING:
                break;
            case TaskStatus.COMPLETED:
                break;
        }
    }
}

```

可以看出，`doSth()` 方法的参数是 int 类型的，但是使用 `@TaskStatus` 进行了注解，这样外界就无法传递 `TaskStatus` 之外的成员作为参数了。

3.  调用

```
TaskHelper.doSth(TaskStatus.UN_START);

```

在调用时，IDE 会提示 `@TaskStatus int status`，提醒我们传入 `TaskStatus` 类型的值。

同时，调用者如果再随便传入一个 `int` 值，虽然可以运行，但代码会爆红，lint 检查将会给与警告：

> Must be one of: TaskStatus.UN_KNOW, TaskStatus.UN_START, TaskStatus.PROGRESSING, TaskStatus.COMPLETED

如此，保证了类型安全。但也只是警告，仍然可以运行，但总比没有警告强多了。

4.2 补充
------

其实这种用法在 Android 源码中屡见不鲜，比如 `Resources` 下的 `getDrawable()` 方法：

```
public Drawable getDrawable(@DrawableRes int id) throws NotFoundException {
        final Drawable d = getDrawable(id, null);
        //...
        return d;
    }

```

使用时，我们一般会这么用：

```
getResources().getDrawable(R.drawable.ic_launcher);

```

虽然是 `int` 值，但是当我们这么用时：

```
getResources().getDrawable(111);

```

就会爆红并提示：

> Expected resource of type drawable

再补充一点，为了防止在我们定义的常量值重复，我们还可以使用 Android 注解支持库（support-annotations）中的 `@IntDef` 来限定常量不允许重复：

```
@IntDef({
        TaskStatus.UN_KNOW,
        TaskStatus.UN_START,
        TaskStatus.PROGRESSING,
        TaskStatus.COMPLETED
})
@Retention(RetentionPolicy.SOURCE)
public @interface TaskStatus {
    int UN_KNOW = -1;
    int UN_START = 0;
    int PROGRESSING = 1;
    int COMPLETED = 2;
}

```

以上，就这些。

5. 总结
=====

好了，为注解正名了。现在看完本文之后再思考，你应该清楚什么场景下该使用那种方案了吧。

文章开头所说的三种场景，注意区分开就好了。之所以使用注解，是因为在一组常量的情况下，枚举太重，不要轻易使用；其次，使用常量会导致类型安全、可读性差等问题。

就酱，周末愉快。

扫描下方二维码，关注我的公众号，及时获取最新文章推送！  
![](https://img-blog.csdnimg.cn/20181210011015948.jpg)