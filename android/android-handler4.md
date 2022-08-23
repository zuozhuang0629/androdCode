#  重学Android之----Android消息机制系列(四)

## 前言

我们在学习了Android的消息机制，也就是Handler的消息机制，我们来通过一些常见面试问题，在巩固一下我们之前的学习的知识。

## 一个线程有几个Handler？

一个线程可以有多个Handler，因为有多个Handler，所以在一个线程中可以通过给其他Handler发送消息来进行线程的切换。

线程和Handler是一对多的关系

## 一个线程有几个Looper？如何保证？

一个线程只能有一个Looper，我们知道Looper的创建是和当前的有关系，他们是一一绑定的关系。

## 如何创建Message对象

有三种创建方法，分别为：
~~~
1、Message message = new message();
2、Message message = Message.obtain();
3、Message message = handler.obtainMessage();
~~~

但是我们在实际开发的时候是不建议使用new方法。因为在Message内部保存了一个缓存消息池，每个new Message在使用完成系统会放入消息池中，会占较多的内存。我们推荐使用obtain方法来从缓存池获取Message，在使用完成后系统会调用recycle进行回收

## MessageQueue怎么保证内部线程安全

内部是上了锁，使用了synchronized关键字，我们放入消息的方法enqueueMessage内部使用了synchronized，也在获取消费消息，queue.next()内部也是用了synchronized。

## 为什么Looper死循环不会导致应用卡死

使用了epoll机制。epoll在Linux中是一个重要的机制，Android的Handler也是用了这个机制。epoll就是进行了阻塞和唤醒机制。在java层，在调用next(),会调用nativePollOnce(long ptr, int timeoutMillis)，当没有消息的时候timeoutMillis = -1表示一直阻塞。当有消息来到的时候就会调用nativeWake()，进行唤醒。

## 主线程为什么不用初始化Looper

应用在启动的过程中就已经初始化了Looper。
~~~
public static void main(String[] args) {
    ...
 // 初始化主线程Looper
    Looper.prepareMainLooper();
    ...
    // 新建一个ActivityThread对象
    ActivityThread thread = new ActivityThread();
    thread.attach(false, startSeq);

    // 获取ActivityThread的Handler，也是他的内部类H
    if (sMainThreadHandler == null) {
        sMainThreadHandler = thread.getHandler();
    }

    ...
    Looper.loop();
 // 如果loop方法结束则抛出异常，程序结束
    throw new RuntimeException("Main thread loop unexpectedly exited");
}
~~~

## 能不能让一个Message加急被处理

可以，有一种一步消息可以更快的处理消息

我们在Looper的消息很多，当用户点击按钮，要立即处理UI的消息，要不然用户会有卡顿的感觉。

什么是同步屏障，同步屏障就是处理上面所说的效果。

我们希望在有UI消息的时候，能够优先处理。我们知道加入消息是通过enqueueMessage方法，我们在处理这种的时候，我们需要另个一个方法postSyncBarrier


