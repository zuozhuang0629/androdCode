# 详解 Kotlin 的 equals() , == 和 ===

## 概括
我们在了解Kotlin中的<code>
 equals()</code>, <code>== </code>和<code>===</code>请先了解Java中的<code>equals()</code>和<code>==</code>，链接:
 [面试必问系列之----Java中 == ，equals()和hashCode()的区别](https://juejin.cn/post/7127558753443905550)

在我们了解<code>equals()</code> ,<code> ==</code> 和 <code>===</code>需要先了解两个概念<code>结构相等</code>和<code>引用相等</code> 

Kotlin 中有两种类型的相等性：
- 结构相等（<code>==</code>和<code>!=</code>）
- 引用相等（<code>===</code>和<code>!==</code>）
  
## 结构相等(<code>==</code>和<code>!=</code>)
在我们
相等运算符是检查比较的值是否相等的运算符。这类似于 Java 的 <code>equals()</code> 方法，为啥这么说呢，在表达式<code>a == b</code>会被翻译为：

~~~code
a?.equals(b) ?: (b === null)
~~~
这我知道，<code>==</code>在<code>a</code>不是<code>null</code>的情况下，则会调用<code>a?.equals(b)</code>, <code>?:</code>有点类似于Java中的三木表达式，稍后会详细解说<code>?:</code>和<code>===</code>。这里的<code>equals()</code>方法和Java中一样，在<code>基本数据类型</code>（例如<code>Int</code>）则是对<b>值比较</b>，在<code>引用数据类型</code>的时候则进行的是<b>对象地址进行比较</b>

对<code>equals()</code>不太来接可以参考这篇 [面试必问系列之----Java中 == ，equals()和hashCode()的区别](https://juejin.cn/post/7127558753443905550)

学习完结构相等(<code>==</code>和<code>!=</code>)那么我们看一下一下的代码在巩固一下
~~~
val a = 1
val b = 1
System.out.println(a == b);
输出结果：
true
解析：
因为a，b都是基本书籍类型，所以比较的是值

val a = "a"
val b = "a"
System.out.println(a == b);
输出结果：
true
解析：
因为string是重写了equals()方法内部比较的是值

class User(val name:String){}
val zhangsan = User("zhangsan")
val lisi = User("lisi")
val zhangsan2 = zhangsan
System.out.println(zhangsan == lisi);
System.out.println(zhangsan == zhangsan2);
输出结果 ：
false
true
解析：
因为User是引用数据类型，所以比较的是地址
~~~
### ?:(Elvis 操作符)
当我们有一个可空的引用<code>b</code> 时，我们可以说“如果<code>b</code> 非空，我使用它；否则使用某个非空的值”：
~~~
val l: Int = if (b != null) b.length else -1
~~~  
除了完整的<code>if</code>表达式，这还可以通过 Elvis 操作符表达，写作<code>?:</code>:
~~~
val l = b?.length ?: -1
~~~
如果<code>?:</code>左侧表达式非空，elvis 操作符就返回其左侧表达式，否则返回右侧表达式。 请注意，当且仅当左侧为空时，才会对右侧表达式求值。

请注意，因为 <code>throw</code> 和 <code>return</code> 在 <code>Kotlin</code>中都是表达式，所以它们也可以用在 elvis 操作符右侧。这可能会非常方便，例如，检测函数参数：
~~~
fun foo(node: Node): String? {
    val parent = node.getParent() ?: return null
    val name = node.getName() ?: throw IllegalArgumentException("name expected")
    // ……
}
~~~
那么我们在之前的代码，代码的意思就是如果<code>a</code>不是<code>null</code>则调用<code>equals(Any?)</code>方法，否则也是<code>a</code>是<code>null</code> 检测<code>b</code>时候与<code>null</code>引用相等

<b>注意:当与 <code>null</code> 显式比较时完全没必要优化你的代码：<code>a == null</code> 会被自动转换为 <code>a === null</code></b>

## 引用相等(===或!===)
<code>引用相等(===或!===)</code>用于比较对象的引用是否指向同一个对象，运行时如果是<code>基本数据类型</code> 时<code>===</code> 等价于 <code>==</code>，但是基本数据类型时，不建议使用<code>===</code>，编译器会标黄警告

概念很简单，我们也来看看代码实现效果
~~~

val a = 1
val b = 1
System.out.println(a === b);
输出结果：
true
解析：
因为a，b都是基本书籍类型，所以比较的是值

val a = "a"
val b = "a"
System.out.println(a === b);
输出结果：
true
解析：
因为string是重写了equals()方法内部比较的是值

class User(val name:String){}
val zhangsan = User("zhangsan")
val lisi = User("lisi")
val zhangsan2 = zhangsan
System.out.println(zhangsan === lisi);
System.out.println(zhangsan === zhangsan2);
输出结果 ：
false
true
解析：
因为User是引用数据类型，所以比较的是地址
~~~
