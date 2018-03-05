# Interview
陌陌：

1.如何停止一个线程

2.handler.postdelay();前面有个runnable()未结束，后面的是否会有延迟

postDelay()一个10秒钟的Runnable A、消息进队，MessageQueue调用nativePollOnce()阻塞，Looper阻塞；
紧接着post()一个Runnable B、消息进队，判断现在A时间还没到、正在阻塞，把B插入消息队列的头部（A的前面），然后调用nativeWake()方法唤醒；
MessageQueue.next()方法被唤醒后，重新开始读取消息链表，第一个消息B无延时，直接返回给Looper；
Looper处理完这个消息再次调用next()方法，MessageQueue继续读取消息链表，第二个消息A还没到时间，计算一下剩余时间（假如还剩9秒）继续调用nativePollOnce()阻塞；
直到阻塞时间到或者下一次有Message进队；
MessageQueue会根据post delay的时间排序放入到链表中，链表头的时间小，尾部时间最大。因此能保证时间Delay最长的不会block住时间短的。当每次post message的时候会进入到MessageQueue的next()方法，会根据其delay时间和链表头的比较，如果更短则，放入链表头，并且看时间是否有delay，如果有，则block，等待时间到来唤醒执行，否则将唤醒立即执行。

所以handler.postDelay并不是先等待一定的时间再放入到MessageQueue中，而是直接进入MessageQueue，以MessageQueue的时间顺序排列和唤醒的方式结合实现的。使用后者的方式，我认为是集中式的统一管理了所有message，而如果像前者的话，有多少个delay message，则需要起多少个定时器。前者由于有了排序，而且保存的每个message的执行时间，因此只需一个定时器按顺序next即可。


猫眼：

为什么内部类会有外部类的引用及意义？

内存优化

线程池的调度原理（当线程池满了，还有别的请求怎么办）

handler第二个消息怎么先于第一个

okhttp缓存怎么做的

rxjava subscribeon observeon区别
flatmap map currentmap 区别

okhttp中怎么实现的长连接

不同的域名可以使用同一个长连接吗

leakcanary的原理
只能做activity的检查其他的呢 怎么处理

java default和private的区别

volitate可以实现原子性吗为什么？

除了线程池和rxjava怎么管理线程

linearlayout和relativelayout的区别 那个更好 为什么？

设置wrapcontent 和 matchparent的时候 是在onmeasure还是在onlayout中做的

如何获取xml中的层级

finlize方法作用

自定义view需要重写那些方法

自定义构造函数参数的意义

Activity中使用handler加thread可能会内存泄漏 有什么方法可以避免这种情况？

美团：

jvm垃圾回收，gc root是什么
handler的postdelay实现原理，messagequene的阻塞唤醒原理，内存泄露的原因
synchronized的用法 static synchronized的区别
线程池
service，intentservice的区别


我觉得，IntentService 为什么可以处理耗时任务？ 应该从源码上面来分析，IntentService 是直接继承与 Service的，继承Service后 它的代码一共就100多行。
内部在 onCreate()时，新建了一个HandlerThread 实例。
（HandlerThread 是一个Thread的 子类，HandlerThread 内部 有点线我们的UI线程，内部一个Looper loop循环一直轮询消息 获得消息 处理消息。）
而IntentService, 内部有一个Handler子类 ServiceHandler，它的Looper用的是这个HandlerThread 的Looper,IntentSerivce 在onStart()通过发送Message,ServiceHandler在处理Message 调用的是 onHandleIntent。 所以

简单的说一个IntentService,内部就创建了一个线程，通过Android提供的 Handler Message Looper,这些消息处理的类 构成了一个消息处理的模型。所以IntentService 的onHandleIntent 这个方法其实是在IntentService 中开辟的一个子线程中处理的。

三叉树求最大深度
view绘制流程以及viewgroup的绘制流程，draw和ondraw的区别及用法

http://blog.csdn.net/yanbober/article/details/46128379


view的measure()方法：

每个View控件的实际宽高都是由父视图和自身决定的，实际的测量时在onMeasure()方法中 所以View的子类需要重写onMeasure()方法

//final方法，子类不可重写
    public final void measure(int widthMeasureSpec, int heightMeasureSpec) {
        ......
        //回调onMeasure()方法
        onMeasure(widthMeasureSpec, heightMeasureSpec);
        ......
    }

这个方法的两个参数都是父View传递过来的，代表了父View的规格，前两位MODE,三中EXACTLY AT_MOST UNSPECIFIED

后三十位为大小 当然可以通过setMeasuredDimension设置任意大小


这个方法对View的成员变量mMeasuredHight进行赋值和mMeasuredWidth赋值，赋值结束 measure结束


如果不重写赋值则调用getDefaultSize()方法

View实际为嵌套的 而且measure是递归传递的，所以每个View都需要measure,实际能够嵌套view的都是viewgroup的子类，所以viewgroup定义了 measureChildren,measurechild,measurechildwithmargins

 区别是measurechildwithmargins是否把margin和padding也作为子视图的大小


protected void measureChildWithMargins(View child,
            int parentWidthMeasureSpec, int widthUsed,
            int parentHeightMeasureSpec, int heightUsed) {
        //获取子视图的LayoutParams
        final MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();
        //调整MeasureSpec
        //通过这两个参数以及子视图本身的LayoutParams来共同决定子视图的测量规格
        final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
                mPaddingLeft + mPaddingRight + lp.leftMargin + lp.rightMargin
                        + widthUsed, lp.width);
        final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
                mPaddingTop + mPaddingBottom + lp.topMargin + lp.bottomMargin
                        + heightUsed, lp.height);
        //调运子View的measure方法，子View的measure中会回调子View的onMeasure方法
        child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
    }

该方法就是对父视图提供的measurespace参数结合自身的LayoutParams参数进行了调整 然后再调用child.measure()方法

measure回调onMeasure()方法


ViewRootImpl的performTraversals中的measure执行完成后 就会接着执行view.layout



ViewGroup的layout方法实质上还是调用了view父类的layout方法，所以看view的layout


public void layout(int l, int t, int r, int b) {
        ......
        //实质都是调用setFrame方法把参数分别赋值给mLeft、mTop、mRight和mBottom这几个变量
        //判断View的位置是否发生过变化，以确定有没有必要对当前的View进行重新layout
        boolean changed = isLayoutModeOptical(mParent) ?
                setOpticalFrame(l, t, r, b) : setFrame(l, t, r, b);
        //需要重新layout
        if (changed || (mPrivateFlags & PFLAG_LAYOUT_REQUIRED) == PFLAG_LAYOUT_REQUIRED) {
            //回调onLayout
            onLayout(changed, l, t, r, b);
            ......
        }
        ......
    }


View的layout方法可以重写 而ViewGroup的layout是不能重写的 viewgroup只能重写onlayout() 抽象 所以继承ViewGroup必须重写onLayout（）方法

所以在自定义ViewGroup控件中，onLayout配合onMeasure方法一起使用可以实现一些自定义view的复杂布局

自定义view首先调用onMeasure进行测量，然后调用onLayout方法动态配合获取子view和子view的测量大小。然后进行layout布局，重载onLayout的目的就是安排其children在父View的具体位置。

重载onLayout通常做法就是写一个for循环调用每一个子view的layout()函数，传入不同参数l,t,r,b来确定每个子视图在父视图中的显示位置

一般情况下layout过程会参考measure过程中计算得到的mMeasuredWidth和mMeasuredHeight来安排子View在父View中显示的位置，但这不是必须的，measure过程得到的结果可能完全没有实际用处，特别是对于一些自定义的ViewGroup，其子View的个数、位置和大小都是固定的，这时候我们可以忽略整个measure过程，只在layout函数中传入的4个参数来安排每个子View的具体位置。

自定义控件的写法

measure操作完成后得到的是对每个View经测量过的measuredWidth和measuredHeight，layout操作完成之后得到的是对每个View进行位置分配后的mLeft、mTop、mRight、mBottom，这些值都是相对于父View来说的。

第一步，对View的背景进行绘制。


第三步，对View的内容进行绘制。


第四步，对当前View的所有子View进行绘制，如果当前的View没有子View就不需要进行绘制。

View的draw方法中的dispatchDraw(canvas)


这是一个空方法。因为每个View的内容部分是各不相同的，所以需要由子类去实现具体逻辑。


第六步，对View的滚动条进行绘制。


美团对基础非常看重，每一面都有一个小算法题，一二面基本都是聊基础，但是说基础的同时也不缺深度，感觉非常不错。

一面： String相关的问了很多、HashMap 的原理、Java 的引用类型、Activity 启动模式、Activity 生命在各种情况下的生命周期、二分查找、单例模式、Handler 机制、系统中用 Handler 的地方等等等等，看着都是些常见的，但是问的非常的细，很多都是平时容易忽略的东西。面试官人很好，一直告诉我不要紧张，因为我一直在喝水，我其实是非常饿，最后让等二面的时候再次嘱咐我不要紧张，二面尽量往你会的方向引导，真的很感谢那个面试官。

二面：二面聊的比较多的是 Java 相关的，基础是一方面，然后聊到热修复的时候随便问到了我 类加载器与Java 虚拟机的双亲委托模型，也是有一个算法，写完后让我继续优化这个算法，我是真优化不动了，面试官一看就是很耿直的人，告诉我老大今天开会让我回去等电话，这次我觉得应该不是委婉的拒绝。

三面：等了很久，终于等到了 Hr 的电话安排我去三面，三面的面试过那天刚好在校招，于是我就被带到校招的地方去了，我觉得在面试过看到我简历那一刻开始就没打算要我了，整个过程非常短，基本没聊技术，自我介绍环节都省略了，简单聊了下然后让我做一个算法题，我写完后正犹豫，他有收卷的意思，然后就说让回去等，我心里知道肯定没戏了...




一面

简历上写的项目问了一遍，然后开始问知识点。

1. volley的源代码，在图片缓存部分讨论了挺长时间，http中缓存机制，Last-Modify的作用等。
2. fragment的生命周期
3. service一些知识
4. 事件分发机制
5. Binder实现机制，Stub类中asInterface函数作用，BnBinder和BpBinder区别。
6. gradle中buildToolsVersion和TargetSdkVersion的区别是什么
7. 手机适配一些方案
8. hashmap的实现原理
9. 静态方法是否能被重写
这些大概聊了1个半小时，开始的时候还有些紧张，慢慢聊开了，就好多了，面试官的语速有点快，老是需要面试官重复一遍，我也不经意间语速也变快了，不过能看出来面试官还是很厉害的。

二面

1. 3次握手和4次挥手的原因，以及为什么需要这样做。
2. 数据结构，搜索二叉树的一些特性，平衡二叉树。
3. hashmap是如何解决hash冲突的
4. 进程与线程区别
5. 写了一个二分查找和单例模式
6. http中的同步和异步
7. 聊了一些项目上做的东西,问了问职业规划
由于二面面试官不是做Android，本来面试我的人临时开会去了，所以这一轮面试没怎么问android相关知识，不过二面面试官一直是微笑，所以这一轮很轻松，更像是一起讨论问题。 
面试完已经是下午4:30了，由于面试当天是星期五，而周五美团的会议比较多，所以等了会，二面面试官说三面面试官在开会，面试另约时间，我还是说这次一次面试完吧，这一等就等了2个半小时，期间hr跟我说三面面试官是个大牛。

三面

1. 我认为Android做的优秀的几个地方，然后又根据我说的问了问比较深入问题。
2. Android是如何进行资源管理的。
3. java比较重要的几个特性
4. 网络五层结构，每一层协议，由于我网络不是很好，还问了一些其他的问题（例如MAC地址和ip地址的区别等）。
5. 为什么离开原来公司，以及职业规划，然后因为面试完大概就晚上8点了，就先让我回去，下周让hr跟我联系，我想这是应该通过面试了吧。
美团技术还是很厉害的，从面试官的水平就可以看出来，尤其是外卖核心部门，办公环境是不错，但是感觉就是有点乱，不知道是不是因为今天面试的人很多，基本上一直有很多人来回走动，有一些嘈杂。

public static int binSearch(int srcArray[],int key){
    int mid;
    int start = 0;
    int end = srcArray.length - 1;

    while (start <= end){

        mid = (end - start)/2 + start;

        if (key < srcArray[mid]){
            end = mid -1;
        }else if (key > srcArray[mid]){
            start = mid +1;
        }else {
            return mid;
        }
    }

    return -1;
}


百度：
ArrayList与LinkedList区别 分别是怎么扩容的？

设计模式都会几种？写出一两个

所有的进程间通信方式(Android开发与艺术第二章)

简历上最近的一个项目 所用到的技术要点

快速排序

java泛型 https://www.zhihu.com/question/20400700

Binder

HashMap  https://zhuanlan.zhihu.com/p/27325430

为什么要离职？


http://blog.csdn.net/a296777513/article/details/73610719

https://www.jianshu.com/p/c70989bd5f29

插件化博客：
http://weishu.me/2016/01/28/understand-plugin-framework-overview/

