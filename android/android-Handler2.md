---
theme: channing-cyan
---
携手创作，共同成长！这是我参与「掘金日新计划 · 8 月更文挑战」的第十五天，[点击查看活动详情](https://juejin.cn/post/7123120819437322247 "https://juejin.cn/post/7123120819437322247")

#  重学Android之----Android消息机制系列(二)

## 前言

上篇我们已经了解了`Handler`消息机制做了概括性的描述。通过流程也大致了解了`Handler`的工作过程，本章将会`Handler`的消息机制的实现做一个全面的分析。

为了更好理解`Looper`的工作原理，我们先来理解一下`ThreadLocal`。

## ThreadLocal的工作原理

`ThreadLocal`是一个线程内部的数据存储，通过它可以在指定的线程中存储数据，数据存储之后，只有在指定线程中获取数据，其他线程是无法获取到数据。日常中运用场景不多，一般运行在指定的一些场景,通过`ThreadLocal`可以实现一些复杂的场景。一般来说，当某些数据是一线程作为作用域并且不同线程具有不同的数据副本来的时候，就可以采用`ThreadLocal`。对于`Handler`来说，它需要获取当前线程的`Looper`，那么说`Looper`的作用域就是线程，不同的线程会有不同线程具有不同的`Looper`，这个时候就可用`ThreadLoacl`就可以简单实现线程中的存取了。

> 如果不采用`ThreadLocal`，那么系统就必须提供一个全局的哈希表提供`Handler`查找指定线程的`Looper`，这样一来就必须提供一个类似`LooperManager`的管理类，所以系统选择了`ThreadLoacl`。

大家可能对`ThreadLocal`的知识还是有点抽象，我们就举个例子来演示：
~~~
//首先我们声明了一个Boolean类型的ThreadLocal
private val mBoolThreadLocal = ThreadLocal<Boolean>()

fun main(){

    //主线程中设置mBoolThreadLocal的值为true
    mBoolThreadLocal.set(true)

    println("mainThread---mBoolThreadLocal=${mBoolThreadLocal.get()}")

    Thread {
        //再在子线程1中设置为false
        mBoolThreadLocal.set(false)
        println("Thread#1---mBoolThreadLocal=${mBoolThreadLocal.get()}")

    }.start()


    Thread {
        //再在子线程2中直接获取mBoolThreadLocal的值
        println("Thread#3---mBoolThreadLocal=${mBoolThreadLocal.get()}")
    }.start()
}
~~~

输出：

~~~
mainThread---mBoolThreadLocal=true
Thread#1---mBoolThreadLocal=false
Thread#3---mBoolThreadLocal=null
~~~

解析：

我们在日志中可以看出，在不同线程中访问同一个`ThreadLocal`对象，但是获取的值却是不同，这就是`ThreadLocal`的特性。这是因为在不同线程中访问`ThreadLocal`的`get`方法，`ThreadLocal`内部会从各自的线程获取`ThreadLocalMap`，然后根据不同的线程中维护一套数据的副本，并且彼此互不干扰。

### ThreadLocal 源码分析

上面我们介绍了使用方法和工作过程，下面我们分析一下`ThreadLocal`的内部实现，`ThreadLocal`是一个泛型类，我们只要搞清楚`ThreadLoca`的`set`和`get`方法就能明白它的工作原理

- 首先我们先看`ThreadLocal`的`set`方法：

~~~
public void set(T value) {
    //拿到当前Thread对象
    Thread t = Thread.currentThread();
    
    //从当前Thread对象中拿出以ThreadLocal为key的ThreadLocalMap容器
    ThreadLocalMap map = getMap(t);
    
    if (map != null)
        //如果容器已经初始化，则直接把当前ThreadLocal对象作为Key，value作为map的value存入map
        map.set(this, value);
    else
        //如果容器未被初始化，则在初始化的同时，存储当前键值对。
        createMap(t, value);
}
~~~

解析：

通过`getMap(t)`方法获取到一个简单的类似的`HashMap`的容器`ThreadLocalMap`，该容器是一个`Entry数组`，初始化长度是16。`Entry`的`key`类型为`ThreadLocal`，`value`类型则为存入的内容（比如：`Looper`）。

![2247636628-748182864ba94cc4_fix732.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cc9528427d6f4f5ea220f00f5c83a6a0~tplv-k3u1fbpfcp-watermark.image?)

既然结构类似`Map`，那么我们也看一下`ThreadLocalMap.set()`方法：
~~~
private void set(ThreadLocal<?> key, Object value) {

  // We don't use a fast path as with get() because it is at
  // least as common to use set() to create new entries as
  // it is to replace existing ones, in which case, a fast
  // path would fail more often than not.

  Entry[] tab = table;
  int len = tab.length;

  //通过规则获取key的索引位置(hashcode来计算)
  int i = key.threadLocalHashCode & (len-1);

  for (Entry e = tab[i];
       e != null;
       //从下标i开始，寻找k == key的entry
       e = tab[i = nextIndex(i, len)]) {
      ThreadLocal<?> k = e.get();

      if (k == key) {
        //若找到，则直接赋值。
          e.value = value;
          return;
      }

      if (k == null) {
        //k==null表示threadLocal被gc回收（因为entry对threadLocal是弱引用，弱引用不会影响垃圾回收），将这个位置分配给当前的key。
          replaceStaleEntry(key, value, i);
          return;
      }
  }

//若以上都没发生，则此时i下标所代表的Entry一定是个空，所谓找到了 一个“插槽”，那就直接赋值entry即可。
  tab[i] = new Entry(key, value);
  int sz = ++size;
  if (!cleanSomeSlots(i, sz) && sz >= threshold)
      //清理一遍其余插槽，该扩容就扩容。确保插槽足够
      rehash();
}
~~~

解析：

`ThreadLocalMap.set()`方法很简单，我们在简单解释一下`for`循环中逻辑：

1. 遍历当前`key`值对应的桶中`Entry`数据为空，这说明散列数组这里没有数据冲突，跳出`for`循环，直接`set`数据到对应的桶中
   
2. 如果`key`值对应的桶中`Entry`数据不为空
   
   1. 如果`k == key`，说明当前`set`操作是一个替换操作，做替换逻辑，直接返回
   2. 如果`key == null`，说明当前桶位置的`Entry`是过期数据，执行`replaceStaleEntry()`方法(核心方法)，然后返回
   
3. `for`循环执行完毕，继续往下执行说明向后迭代的过程中遇到了`entry`为`null`的情况
   
    1. 在`Entry`为`null`的桶中创建一个新的`Entry`对象
    2. 执行`++size`操作

4. 调用`cleanSomeSlots()`做一次启发式清理工作，清理散列数组中`Entry`的`key`过期的数据


- 我们再看`ThreadLocal`的`get`方法：
  
~~~
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}
~~~

其实逻辑也很简单，也是先获取`ThreadLocalMap`对象，如果`ThreadLocalMap`获取的值不为`null`那么就直接返回，要么就会调用`setInitialValue`进行初始化，我们看一下`setInitialValue`的源码：

~~~
 private T setInitialValue() {
    T value = initialValue();
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
    return value;
}

 protected T initialValue() {
    return null;
}
~~~

我们发现`setInitialValue()`会一开始调用`initialValue()`方法，`initialValue()`方法这个方法会返回`null`，因为是`protected`修饰，所以可以重写，然后会获取`ThreadLocalMap`，不为`null`就存入，`ThreadLocalMap`为`null`就会创建一个


其实内部也是`ThreadLocalMap.getEntry()`方法，我们直接来看`ThreadLocalMap.getEntry()`的源码：

~~~
private Entry getEntry(ThreadLocal<?> key) {
    //根据key计算位置下标
    int i = key.threadLocalHashCode & (table.length - 1);
    Entry e = table[i];
    // 根据下标获取的元素不为null，并且和本次key相同
    if (e != null && e.refersTo(key))
        return e;
    else
        return getEntryAfterMiss(key, i, e);
}

private Entry getEntryAfterMiss(ThreadLocal<?> key, int i, Entry e) {
    Entry[] tab = table;
    int len = tab.length;

    //向后遍历直到遇到空插槽
    while (e != null) {
        //找到了直接返回
        if (e.refersTo(key))
            return e;
        //旧条目需要清除
        if (e.refersTo(null))
            expungeStaleEntry(i);
        else
            i = nextIndex(i, len);
        //下一个条目
        e = tab[i];
    }
    return null;
}

~~~

`ThreadLocalMap.getEntry()`方法也是比较简单，如果遍历到空槽还没有找到`key`，则表示查询失败，返回`null`


<b>我们从`ThreadLocal`的`set`和`get`方法可以看出，它们所操作的对象都是当前线程的`ThreadLocalMap`，因此在不同线程中访问同一个T`hreadLocal`的`set`和`get`方法，所以对`ThreadLocal`所做的读、写操作都是限制于各自线程内部，这就是为什么`ThreadLocal`可以再多个线程中互不干扰存储和修改数据</b>
