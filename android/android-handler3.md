---
theme: channing-cyan
---
携手创作，共同成长！这是我参与「掘金日新计划 · 8 月更文挑战」的第十六天，[点击查看活动详情](https://juejin.cn/post/7123120819437322247 "https://juejin.cn/post/7123120819437322247")

#  重学Android之----Android消息机制系列(三)

## 前言

上一章节中我们已经了解了ThreadLocal，理解ThreadLocal后有助于我们来理解Looper的工作原理。本章节开始讲解消息队列的工作原理

## 消息队列的工作原理

我们在了解整个消息队列的工作原理，我们先从整个机制的最小单位Message开始，Message表示一个消息，可以理解为线程通讯的数据单元，可以传递数据。

### Message 消息载体

我们看一下Message中重要的成员变量：

~~~
//可以用来区分不同的消息，然后在Handler中做不同的处理
public int what;

//可以传递数据，当需要传递的数据仅仅是int类型时，可以用arg1和arg2，而不需要使用复杂的obj和data
public int arg1;
public int arg2;

//可以传递数据，可以传递任意的Object对象
public Object obj;

//保存发送消息的Handler对象，可以在创建Message时指定，在后面会有用到target这个参数，所以特别说明
@UnsupportedAppUsage
Handler target;

//Runable对象，可以在创建Message时指定，另外在Handler使用post形式发送消息时的callback参数最终也是保存在 Message的这个参数中
@UnsupportedAppUsage
Runnable callback;

//Message对象，用来使Message组成链表的形式
@UnsupportedAppUsage
Message next;

//主要是给Message加一个对象锁，不允许多个线程同时访问Message类和obtain方法，保证获取到的sPool是最新的
public static final Object sPoolSync = new Object();

//存储我们循环利用Message的单链表。在Handler消息机制中说过Message的数据结构，因此这里sPool只是链表的头节点
private static Message sPool;

//单链表的链表的长度，即存储的Message对象的个数
private static int sPoolSize = 0;
~~~

我们在看一下Message的构造方法：

~~~
// 空参数构造方法，直接new Message()创建对象
Message(){} 

// 使用静态的obtain()方法获取，内部会重复使用回收的Message对象，比较常用
static Message obtain() 

// 根据传递的Message对象复制一个新的Message对象
static Message obtain(Message orig) 

// 给Message指定Handler，会将 h 赋值给 成员变量 target
static Message obtain(Handler h) 

// 给Message指定Handler和Runable，会将 h 赋值给 成员变量 target，callback赋值给Message的成员变量 callback
static Message obtain(Handler h, Runnable callback) 

// 给Message指定Handler和what，赋值给Message中对应的成员变量
static Message obtain(Handler h, int what) 

// 给Message指定Handler、what和Object类型的值，赋值给Message中对应的成员变量
static Message obtain(Handler h, int what, Object obj) 

// 指定Message更多的属性
static Message obtain(Handler h, int what, int arg1, int arg2) 

// 指定Message更多的属性
Message obtain(Handler h, int what, int arg1, int arg2, Object obj) 
~~~

虽然Message有公有构造函数，但是建议使用其提供的obtain系列函数来获取Message对象，这种创建方式会重复利用缓存池中的对象而不是直接创建新的对象，从而避免在内存中创建太多对象，避免可能的性能问题。

我们来看一下obtain的源码实现：

~~~
public static Message obtain() {
    //给对象加锁，保证同一时刻只有一个线程使用Message
    synchronized (sPoolSync) {
         // 判断sPool链表是否是空链表
        if (sPool != null) {

            //将链表头节点移除作为重用的Message对象，第二个节点作为新链表（sPool ）的头节点
            Message m = sPool;
            sPool = m.next;
            m.next = null;

            //清除标志位
            m.flags = 0;  

            //链表的长度减一
            sPoolSize--;
            return m;
        }
    }
    //直接创建一个Message对象返回
    return new Message();
}
~~~

解析：

从obtain()的源代码中我们可以知道,它是静态方法,而且只有在spool == null 的情况下才会new出一个Message(),返回一个Message对象,如果在不为空的情况下,Message的对象都是从Message对象池里面拿的实例从而重复使用的,这也为了Android中的Message对象能够更好的回收。

### Handler 发送消息

Handler的构造较多，最终会调用的是两个：

~~~
// 两个参数的构造方法
public Handler(Callback callback, boolean async) {

    ...

    // 通过Looper中的方法myLooper()获取与当前线程绑定的Looper，具体实现在Looper部分说明

    mLooper = Looper.myLooper();

    // 当前线程不是Looper线程，也就是没有调用Looper.prepare()给线程创建Looper对象
    if (mLooper == null) {

        // 抛出异常
        throw new RuntimeException(
        "Can't create handler inside thread that has not called Looper.prepare()");
    }

    // 让Handler持有当前线程消息队列(MessageQueue)的引用
    mQueue = mLooper.mQueue;

    // 主要用于Handler的消息发送的回调，优先级是比handlerMessage()方法高，但不常用
    mCallback = callback;

    // 是否异步，如果为true则会调用Message中的setAsynchronous(boolean async)方法，设置参数为true
    mAsynchronous = async;
}

// 三个参数的构造方法
public Handler(Looper looper, Callback callback, boolean async) {

    // 直接赋值，获取Looper中的MessageQueue引用让Handler持有
    mLooper = looper;
    mQueue = looper.mQueue;
    mCallback = callback;
    mAsynchronous = async;
}
~~~

Handler的发送消息可以分为send和oost，但是post底层也是通过send来实现，我们直接来看send源码：

~~~
// 1 sendMessage(Message msg)
// 2 sendEmptyMessage(int what)
// 3 sendEmptyMessageDelayed(int what, long delayMillis)
// 4 sendEmptyMessageAtTime(int what, long uptimeMillis)
// 5 sendMessageDelayed(Message msg, long delayMillis)

// 6 前面5个发送消息的方法最终调用的都是这个(第6个)方法
public boolean sendMessageAtTime(Message msg, long uptimeMillis) {

    // 将Handler中的MessageQueue引用赋值给新的MessageQueue对象
    MessageQueue queue = mQueue;

    // 判断MessageQueue是否为null，如果为null抛出一个 RuntimeException 异常
    if (queue == null) {
        RuntimeException e = new RuntimeException(
            this + " sendMessageAtTime() called with no mQueue");
        Log.w("Looper", e.getMessage(), e);
        return false;
    }

    // 调用Handler的enqueueMessage()方法，并将时间传递过去,内部实现后面会详解
    return enqueueMessage(queue, msg, uptimeMillis);
}

// 7 这个方法表示将消息对象(Message)插入到消息队列(MessageQueue)的最前面
public final boolean sendMessageAtFrontOfQueue(Message msg) {
    MessageQueue queue = mQueue;
    if (queue == null) {
        RuntimeException e = new RuntimeException(
            this + " sendMessageAtTime() called with no mQueue");
        Log.w("Looper", e.getMessage(), e);
        return false;   
    }
    // 传的时间值为0
    return enqueueMessage(queue, msg, 0);
}
~~~

我们在一下Handler的enqueueMessage()源码：

~~~
private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {

    // 特别注意：将当前对象(Handler对象)赋值给Message的target变量
    msg.target = this;

    // 根据是否异步调用Message的setAsynchronous()方法
    // 在上面Handler的构造方法中提到过
    if (mAsynchronous) {
        msg.setAsynchronous(true);
    }

    // 调用MessageQueue的enqueueMessage()方法对Message进行排序，后续会详解
    return queue.enqueueMessage(msg, uptimeMillis);
}
~~~

我们看到Handler发送消息时，没有做什么过多的操作，我们需要注意：<b>enqueueMessage()方法中把Message的target绑定了Handler</b>，然后再吧Message放入MessageQueue中。

### MessageQueue原理分析

消息队列在Android中就是MessageQueue，MessageQueue主要包含两个操作：插入和读取。读取操作本身会伴随着删除操作，插入和读取对应的方法为enqueueMessage和next。

- enqueueMessage：往消息队列中插入一条消息
  
- next：从消息队列中取出一条消息并将其从消息队列中移
除。

我们之前也有提过，尽管MessageQueue叫消息队列，但是内部实现却通过一个单链表的数据结构来维护消息列表，单链表在插入和删除上
比较有优势。

我们来看一下enqueueMessage和next的元源码实现：

~~~
boolean enqueueMessage(Message msg, long when) {

    // 消息对象的目标是 null 时直接抛出异常，因为这意味这个消息无法进行分发处理，上面讲解过
    if (msg.target == null) {
        throw new IllegalArgumentException("Message must have a target.");
    }

    //在这里使用synchronized同步代码块，防止多线程导致的异步操作混乱
    synchronized (this) {

         //消息正在使用时抛出异常，消息不能并发使用。
        if (msg.isInUse()) {
            throw new IllegalStateException(msg + " This message is already in use.");
        }

        // 正在退出消息循环时，回收消息对象。
        if (mQuitting) {
            IllegalStateException e = new IllegalStateException(
                    msg.target + " sending message to a Handler on a dead thread");
            Log.w(TAG, e.getMessage(), e);
            msg.recycle();
            return false;
        }

        // 指定消息正在使用
        msg.markInUse();

        // Message的when设置为参数的when
        msg.when = when;

         // 定义一个临时变量保存队列的头(队首的消息)
        Message p = mMessages;
        boolean needWake;

       // 如果队列为null或者when为0或者when<p.when，那么这个消息就作为队列的头，并和原来的队列连接起来
        if (p == null || when == 0 || when < p.when) {
            // New head, wake up the event queue if blocked.
            msg.next = p;
            mMessages = msg;
            needWake = mBlocked;
        } else {

            // 如果队列不为空，就将消息摆放到队列中合适的位置
            needWake = mBlocked && p.target == null && msg.isAsynchronous();
            Message prev;

            //遍历队列，按时间顺序将消息排列
            for (;;) {
                prev = p;
                p = p.next;

                // 当满足 p = null (到了最后一个)或者 when < p.when 时(根据执行时间找到合适了位置了)跳出循环
                if (p == null || when < p.when) {
                    break;
                }
                if (needWake && p.isAsynchronous()) {
                    needWake = false;
                }
            }

             // 将Message插入到队列中
            msg.next = p; // invariant: p == prev.next
            prev.next = msg;
        }

        // We can assume mPtr != 0 because mQuitting is false.
        // 如果需要唤醒，调用native方法，让轮询器继续取消息
        if (needWake) {
            nativeWake(mPtr);
        }
    }
    return true;
}
~~~

了解了enqueueMessage的源码后，我们在看一下next元源码：

~~~
Message next() {
    // 定义变量保存native代码中的消息队列地址值
    final long ptr = mPtr;

    // 如果地址值为0，表示队列不存在，返回null
    if (ptr == 0) {
        return null;
    }

    // 保存IdleHandler对象的集合的大小，初始化时设置为 -1
    int pendingIdleHandlerCount = -1; // -1 only during first iteration
    
    // native 需要用到的变量. 初始化为 0, 如果大于 0, 表示还有消息待处理(未到执行时间). -1表示阻塞等待
    int nextPollTimeoutMillis = 0;

    for (;;) {
        if (nextPollTimeoutMillis != 0) {
            
            // 如果nextPollTimeoutMillis != 0的话，调用Binder.flushPendingCommands()
            Binder.flushPendingCommands();
        }

        // 进入native层，有可能会阻塞next()方法执行，
        // 等待nextPollTimeoutMillis的时间后开始执行
        nativePollOnce(ptr, nextPollTimeoutMillis);

        synchronized (this) {
            // 记录当前时间
            final long now = SystemClock.uptimeMillis();

            // 初始化变量prevMsg为null，msg为mMessges（第一个消息）
            Message prevMsg = null;
            Message msg = mMessages;
            if (msg != null && msg.target == null) {

                // msg.target == null表示此消息为消息屏障（通过MessageQueue#postSyncBarrier()方法发送来的）
                // 如果发现了一个消息屏障，会循环找出第一个异步消息（如果有异步消息的话），所有同步消息都将忽略（平常发送的一般都是同步消息）
                do {
                    prevMsg = msg;
                    msg = msg.next;
                } while (msg != null && !msg.isAsynchronous());
            }
            if (msg != null) {
                // 找到了可执行的消息
                if (now < msg.when) {
                    // 找到了，但是Message还没到执行的时间，就设置一个等待时间
                    nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
                } else {
                    // 消息到了执行的时间，设置不在阻塞
                    mBlocked = false;
                    // 操作链表
                    if (prevMsg != null) {
                        prevMsg.next = msg.next;
                    } else {
                        mMessages = msg.next;
                    }
                    // 设置msg的下一个为null
                    msg.next = null;
                    if (DEBUG) Log.v(TAG, "Returning message: " + msg);
                    // 设置msg正在使用
                    msg.markInUse();
                    // 返回找到的Message(需要现在处理的Message)
                    return msg;
                }
            } else {
                // 没有找到可执行的消息，设置 nextPollTimeoutMillis 为 -1，会一直阻塞，直到被唤醒
                nextPollTimeoutMillis = -1;
            }

            /************** 走到这里表示没有找到可以返回的消息 ***************/

            // 如果队列需要退出，调用方法销毁队列，并返回null
            if (mQuitting) {
                dispose();
                return null;
            }

            /************** 开始处理IdleHandler，用于空闲的时候处理不紧急事件 ***************/

            // 只有在第一次循环的时候才会走这步，因为 pendingIdleHandlerCount 的初始值是 -1
            if (pendingIdleHandlerCount < 0
                    && (mMessages == null || now < mMessages.when)) {
                pendingIdleHandlerCount = mIdleHandlers.size();
            }
            if (pendingIdleHandlerCount <= 0) {
                // 集合中没有IdleHandler对象(没有需要空闲时处理的逻辑)，继续等待，并跳出此次循环
                mBlocked = true;
                continue;
            }
            // 没有跳出循环，表示集合中有元素，
            // 如果保存IdleHandler对象的数组为null，就创建一个
            if (mPendingIdleHandlers == null) {
                mPendingIdleHandlers = new IdleHandler[Math.max(pendingIdleHandlerCount, 4)];
            }
            // 将集合中的IdleHandler对象复制到数组中
            mPendingIdleHandlers = mIdleHandlers.toArray(mPendingIdleHandlers);
        }

        // 遍历数组
        for (int i = 0; i < pendingIdleHandlerCount; i++) {
            final IdleHandler idler = mPendingIdleHandlers[i];
            mPendingIdleHandlers[i] = null; // release the reference to the handler
            boolean keep = false;
            try {
                // 执行IdleHandler的回调
                keep = idler.queueIdle();
            } catch (Throwable t) {
                Log.wtf(TAG, "IdleHandler threw exception", t);
            }
            if (!keep) {
                // 如果keep为false，表示该IdleHandler可以从集合中移出，那么就移除集合中的IdleHandler对象
                // 否则不移除，空闲时间再次可能调用
                synchronized (this) {
                    mIdleHandlers.remove(idler);
                }
            }
        }

        // 重置值
        pendingIdleHandlerCount = 0;
        nextPollTimeoutMillis = 0;
    }
}
~~~

可以发现next方法是一个无限循环的方法，如果消息队列中没有消息，那么next方法会一直阻塞在这里。当有新消息到来时，next方法会返回这条消息
并将其从单链表中移除。

### Looper的工作原理

我们开始了解Looper的具体实现，Looper在Android的消息机制中扮演着消息循环的角色，具体来说就是它会不
停地从MessageQueue中查看是否有新消息，如果有新消息就会立刻处理，否则就一直阻塞在那里。首先看一下它的构造方法，在构造方法中它会创建一个
MessageQueue即消息队列，然后将当前线程的对象保存起来，如下所示：

~~~
// 构造方法用private修饰，表示不能其他类不能创建，需要一个boolean类型的参数
private Looper(boolean quitAllowed) {
    // 在构造方法中与MessageQueue绑定
    // 将boolean类型的参数quitAllowed传给MessageQueue的构造
    // 我们在MessageQueue中说过这个参数的作用，true表示队列不能退出，false表示能退出
    mQueue = new MessageQueue(quitAllowed);
    // 初始化mThread变量为当前线程
    mThread = Thread.currentThread();
}
~~~

我们知道Handler的工作需要Looper，没有Looper的线程就会报错，那么如何为一个线程创建Looper呢？其实很简单，就是通过Looper.prepare()即可为当前线程创建一个Looper，然后接着通过Looper.loop()来开启消息循环，如下所示：

~~~
new Thread("Thread#2") {
    @Override
    public void run() {
        Looper.prepare();
        Handler handler = new Handler();
        Looper.loop();
    };
}.start();
~~~

Looper除了prepare方法外，还提供了prepareMainLooper方法，这个方法主要是给主线程也就是ActivityThread创建Looper使用的，其本质也是通过prepare
方法来实现的。由于主线程的Looper比较特殊，所以Looper提供了一个getMainLooper方法，通过它可以在任何地方获取到主线程的Looper。

我们在看一下prepare()方法：

~~~
// 准备方法，调用带参数的prepare()方法
public static void prepare() {
    // 参数为true，表示队列可以退出
    prepare(true);
}

// 带一个参数 quitAllowed 的prepare()方法
private static void prepare(boolean quitAllowed) {
    // 先从sThreadLocal获取当前线程的Looper对象，如果获取到了，表示当前线程已经有一个Looper了
    // 抛出一个异常，表示在一个线程当中只能创建一个Looper对象
    if (sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    // 当前线程没有创建过Looper，那么就创建一个Looper并指定与Looper绑定的MessageQueue可以退出
    sThreadLocal.set(new Looper(quitAllowed));
}
// 这个方法是专门为主线程创建Looper的
public static void prepareMainLooper() {
    // 同样调用带参数的prepare()方法创建Looper，但是参数为false，
    // 表示与Looper绑定的MessageQueue可以退出，也就是主线程的MessageQueue不能退出
    prepare(false);
    // 进入同步代码块
    synchronized (Looper.class) {
        // 判断成员变量 sMainLooper 是否为null，如果不为null，表示主线程已经创建过了，抛出异常
        if (sMainLooper != null) {
            throw new IllegalStateException("The main Looper has already been prepared.");
        }
        // sMainLooper 为null，调用myLooper()方法给 sMainLooper赋值
        sMainLooper = myLooper();
    }
}
~~~

在看一下myLooper()方法：

~~~
// 从sThreadLocal中取出当前线程的Looper对象并返回
public static @Nullable Looper myLooper() {
    return sThreadLocal.get();
}
// 获取与当前线程绑定的MessageQueue对象
public static @NonNull MessageQueue myQueue() {
    return myLooper().mQueue;
}
// 获取当前Looper所在的线程
public @NonNull Thread getThread() {
    return mThread;
}
~~~

Looper最重要的一个方法是loop方法，只有调用了loop后，消息循环系统才会真正地起作用，它的实现如下所示：

~~~
public static void loop() {
    // 获取与当前线程绑定的Looper对象
    final Looper me = myLooper();
    // 为null，表示当前线程没有Looper对象
    // 还不是Looper线程，抛出异常，没有调用Looper.prepare()方法
    if (me == null) {
        throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
    }
    // 是Looper线程，初始化queue指向当前线程的MessageQueue对象
    final MessageQueue queue = me.mQueue;
    // 确保这个线程是运行在本地进程
    Binder.clearCallingIdentity();
    // 保存一个用于跟踪的身份令牌
    final long ident = Binder.clearCallingIdentity();

    // 进入无限循环
    for (;;) {
        // 调用MessageQueue的next()方法从消息队列中去消息
        // 有可能会阻塞，next()方法在上一篇博客中有详细说明
        Message msg = queue.next(); // might block
        if (msg == null) {
            // 没有消息表示消息队列是退出状态，直接返回
            return;
        }

        // 如果调用了setMessageLogging(@Nullable Printer printer)方法
        // 那么就调用Printer接口中的方法打印日志信息
        final Printer logging = me.mLogging;
        if (logging != null) {
            logging.println(">>>>> Dispatching to " + msg.target + " " +
                    msg.callback + ": " + msg.what);
        }

        final long traceTag = me.mTraceTag;
        if (traceTag != 0 && Trace.isTagEnabled(traceTag)) {
            Trace.traceBegin(traceTag, msg.target.getTraceName(msg));
        }
        try {
            // 找到Message了，调用Handler中的dispatchMessage(msg)方法，分发和处理消息
            // msg.target表示Message的目标Handler，前面的博客强调过这个变量
            msg.target.dispatchMessage(msg);
        } finally {
            if (traceTag != 0) {
                Trace.traceEnd(traceTag);
            }
        }

        if (logging != null) {
            logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
        }

        // 获取一个新的身份令牌，和原来的身份令牌进行比较
        final long newIdent = Binder.clearCallingIdentity();
        if (ident != newIdent) {
            // 如果两个身份令牌不同，打印一个错误级别很高的日志(What The Fuck)
            Log.wtf(TAG, "Thread identity changed from 0x"
                    + Long.toHexString(ident) + " to 0x"
                    + Long.toHexString(newIdent) + " while dispatching to "
                    + msg.target.getClass().getName() + " "
                    + msg.callback + " what=" + msg.what);
        }
        // 释放/回收消息
        msg.recycleUnchecked();
    }
}
~~~

Looper的loop方法的工作过程也比较好理解，loop方法是一个死循环，唯一跳出循环的方式是MessageQueue的next方法返回了null。当Looper的quit方法被
调用时，Looper就会调用MessageQueue的quit或者quitSafely方法来通知消息队列退出，当消息队列被标记为退出状态时，它的next方法就会返回null。也就是
说，Looper必须退出，否则loop方法就会无限循环下去。loop方法会调用MessageQueue的next方法来获取新消息，而next是一个阻塞操作，当没有消息
时，next方法会一直阻塞在那里，这也导致loop方法一直阻塞在那里。如果MessageQueue的next方法返回了新消息，Looper就会处理这条消息：
msg.target.dispatchMessage(msg)，这里的msg.target是发送这条消息的Handler对象，这样Handler发送的消息最终又交给它的dispatchMessage方法来处理了。但是
这里不同的是，Handler的dispatchMessage方法是在创建Handler时所使用的Looper中执行的，这样就成功地将代码逻辑切换到指定的线程中去执行了

### 再来Handler的工作原理

在上面我们知道，Handler发送消息的过程仅仅是向消息队列中插入了一条消息，MessageQueue的next方法就会返回这条消息给Looper，Looper收到消息后就开
始处理了，最终消息由Looper交由Handler处理，即Handler的dispatchMessage方法会被调用，这时Handler就进入了处理消息的阶段。dispatchMessage的实现如
下所示：
~~~
public void dispatchMessage(Message msg) {
    if (msg.callback != null) {
        handleCallback(msg);
    } else {
        if (mCallback != null) {
            if (mCallback.handleMessage(msg)) {
                return;
            }
    }

    handleMessage(msg);
    }
}
~~~

dispatchMessage的逻辑很简单，首先，检查Message的callback是否为null，不为null就通过handleCallback来处理消息。Message的callback是一个Runnable对象，实际上就是Handler的post方法
所传递的Runnable参数。handleCallback的逻辑也是很简单，如下所示：

~~~
private static void handleCallback(Message message) {
    message.callback.run();
}
~~~

其次，检查mCallback是否为null，不为null就调用mCallback的handleMessage方法来处理消息。Callback是个接口，它的定义如下:

~~~
public interface Callback {
    public boolean handleMessage(Message msg);
}
~~~

通过Callback可以采用如下方式来创建Handler对象：Handler handler = new Handler(callback)。那么Callback的意义是什么呢？源码里面的注释已经做了说
明：可以用来创建一个Handler的实例但并不需要派生Handler的子类。在日常开发中，创建Handler最常见的方式就是派生一个Handler的子类并重写其
handleMessage方法来处理具体的消息，而Callback给我们提供了另外一种使用Handler的方式，当我们不想派生子类时，就可以通过Callback来实现。
最后，调用Handler的handleMessage方法来处理消息。Handler处理消息的过程可以归纳为一个流程图:

