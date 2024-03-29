---
theme: channing-cyan
---
携手创作，共同成长！这是我参与「掘金日新计划 · 8 月更文挑战」的第十四天，[点击查看活动详情](https://juejin.cn/post/7123120819437322247 "https://juejin.cn/post/7123120819437322247")


## 前言

本章主要讲述的内容是Android的消息机制。从开发的角度来讲，`Handler`是Android消息机制的山层接口，在使用这个开发过程中只需要和`Handler`交互即可。`Handler`的使用过程比较简单，通过`Handler`可轻松将一个任务线程切换到Handler所在的线程。

很多人认为`Handler`的作用就是更新UI，更新UI只是`Handler`的一个使用场景，例如：你在进行一些耗时操作，网络请求、读写文件等，当操作完成之后，我们可能需要更新UI，但是由于Android的限制，更新UI只能在只能在主线程中执行，否则就会抛出异常，这时候`Handler`就可以讲UI的操作回到主线程中执行。

> 本质来说，Handler不是专门用跟新UI的，他只是能用来更新UI

Android消息机制主要是`Handler`的运行机制，在我们了解`Handler`运行机制之前我们先了解一下，在`Handler`中几个角色

## Handler中角色

在`Handler`消息机制中，有几个不得不提的几个组成元素：`Handler`、`Looper`、`MessageQueue`、`Message`。

- Handler   
  Handler的主要作用是发送和处理消息

- MessageQueue  
  用来存储消息的队列，采用单链表的数据结构

- Looper    
  是一个循环，负责检查消息队列中是否有消息，并负责取出消息

- Message   
  消息的载体
 
`Handler`的运行机制需要底层的`MessageQueue`和`Looper`的支撑。`MessageQueue`它虽然叫消息队列，但是内部存储结构并不是真正的队列，而是采用单链表的数据结构来存储消息列表。`Looper`就是一个循环，由于`MessageQueue`只是一个消息的存储单元，存储的就是`Message`，但是自己不能处理消息，`Looper`就填补了这个处理消息的功能，`Looper`就会以无限循环的方式来遍历`MessageQueue`是否有新的消息处理，有的话就会调用绑定的`Handler`进行处理，否则就一直等待。


## Android消息机制的概括

上面我们简单的讲了`Handler`、`MessageQueue`和`Looper`的工作过程，这三个实际是一个整体，只不过我们在开发过程中，接触到比较多是`Handler`而已。`Handler`主要作用是讲一个一个任务切到某个指定的线程中执行，前面我也这说了，在Android开发的要求中更新UI必须要在住线程中，否则会抛出异常。那么校验是够在主线程是怎么实现的呢？校验工作时再`ViewRootImpl`的`checkThrad`方法：

~~~
final Thread mThread;
public ViewRootImpl(Context context, Display display) {
  mThread = Thread.currentThread();
} 

void checkThread() {
  if (mThread != Thread.currentThread()) {
      throw new CalledFromWrongThreadException(
              "Only the original thread that created a view hierarchy can touch its views.");
  }
}
~~~

针对`checkThread()`方法中抛出的异常信息，我们在开发的过程中应该都遇到过吧，就是由于这个限制，因为有了这个限制，所以我们在更新UI必须在主线程中，但是Android中有不建议在主线程中进行耗时的操作，否则就是出现`ANR(无响应)`的提示。所以我们在进行耗时操作的时候需要再子线程，当我们更新UI的时候再通过H`andler`回到主线中，这就是系统为什么提供`Handler`，主要原因就是解决在子线程中无法更新UI的问题。

> 不管是runOnUiThread()、RxJava还是协程底层都是通过Handler来实现线程切换的

我们往后衍生一下，我们知道Android不允许在子线程中更新UI，那这个设计是为什么呢？这是因为在Android中UI不是线程安全的。如果在多线程中并发访问可能导致UI控件处于预科预期的情况。为什么不上锁？上锁就会影响性能，也会让逻辑变得复杂，所以采用了简单和高校的方法：采用单线程(主线程)来处理UI，对于开发也就使用`Handler`这个简单一举两得。

## Handler的工作原理简单概括

我们描述一下Handler的工作原理。Handler创建的时候会采用当前的Looper来构建的内部的循环系统。如果当前线程没有Looper就会抛出异常：

~~~
public Handler(@Nullable Callback callback, boolean async) {
   ...
    mLooper = Looper.myLooper();
    if (mLooper == null) {
        throw new RuntimeException(
            "Can't create handler inside thread " + Thread.currentThread()
                    + " that has not called Looper.prepare()");
    }
   ...
}

~~~

解决这个问题其实很简单，只需要创建一个Looper即可，或者在创建Handler的时候传入一个Looper也可以。具体会在后面的章节详细。

Handler创建完成后，这个时候其内部就会一个消息系统，Looper和MessageQueue就可以和Handler一起工作了。


我们知道Handler发送消息有两个send和post方法，我们先看一下源码：

~~~
public final boolean postDelayed(@NonNull Runnable r, long delayMillis) {
//这里我们看到postDelayed还是调用自身的sendMessageDelayed，我们接着去看sendMessageDelayed
    return sendMessageDelayed(getPostMessage(r), delayMillis);
}
~~~

~~~
public final boolean sendMessageDelayed(@NonNull Message msg, long delayMillis) {
    if (delayMillis < 0) {
        delayMillis = 0;
    }
    return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
}
~~~

我们发现psot内部调用的也是send来完成的。我们来看一下send的工作过程。当我们调用send时候，它会调用`MessageQueue`的`enqueueMessage`方法将和这个消息放入消息队列中，然后Looper就会发现有新的消息，Looper就会处理这个消息，最终消息中的Runnable或者Handler的handleMessage就会被调用。

> 注意：Looper运行是在Handler创建所在的线程中，这样一来Handler中的业务逻辑就被切换到创建Handler所在线程中去执行

我们看一下流程图：

![3057947d23453017950e3e8b7d2c0e49_1027x603.webp](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b47bad101ee04839b9112cce3abc2974~tplv-k3u1fbpfcp-watermark.image?)

下一篇我们将分析Android的消息机制
