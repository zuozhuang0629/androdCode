# Kotlin 中的运算符符重载

## 前言

我们先看运算法符重载在Kotlin官网的解释：Kotlin 允许您为类型上预定义的一组运算符提供自定义实现。这些运算符具有预定义的符号表示（如<code>+</code>or <code>*</code>）和优先级。要实现运算符，请为相应类型提供具有特定名称的成员函数或扩展函数。这种类型成为二元运算的左侧类型和一元运算的参数类型。

<code>operator</code>  要重载运算符，请使用修饰符标记相应的函数：

~~~
interface IndexedContainer{
    operator fun get(index :Int)
}
~~~

覆盖运算符重载时，可以省略<code>operator</code>：

~~~
class OrdersList: IndexedContainer {
    override fun get(index: Int) { /*...*/ }
}
~~~

## 一元运算

### 一元前缀运算符

| 表达式 | 翻译成 |
| --- | --- |
|  +a|  a.unaryPlus()|
|  -a| a.unaryMinus() |
|  !a| a.not() |

该表表示了，当编译器处理例如一个表达式+a时，它会执行一下步骤

- 确定<code>a</code>的类型，让他成<code>T</code>

- 查找<code>unaryPlus()</code>带有<code>operator</code>修饰符且没有参数的函数接收者<code>T</code>，这意味着成员函数或扩展函数

- 如果函数不存在或不明确，则为编译错误

- 如果函数存在且其返回类型为 <code>R</code>，那就表达式 <code>+a</code> 具有类型 <code>R</code>

注意 这些操作及所有其他操作都针对基本类型做了优化，不会为它们引入函数调用的开销

我们来看一下如何重载一元减运算符的实例：

~~~
data class Point(val x: Int, val y: Int)

operator fun Point.unaryMinus() = Point(-x, -y)

val point = Point(1,2)

fun main(){
    println(-point)
    // 输出“Point(x=-10, y=-20)”
}

~~~

### 递增与递减

| 表达式 | 翻译成 |
| --- | --- |
|  a++|  a.inc() + 见下文|
|  a--| a.dec()  + 见下文|

<code>inc()</code>和<code>dec()</code>函数必须返回一个值，它用于赋值给使用<code>++</code>或<code>--</code>操作的变量。它们不应该改变在其上调用 <code>inc()</code> 或 <code>dec()</code> 的对象。
 
编译器执行一下步骤来解析后缀行的操作符，列如 <code>a++</code>

- 确定a的类型，令其为T

- 查找一个使用与类型为T的接受者、带有operator修饰符的无参数函数inc()

- 检测函数的返回类型是T的子类

计算表达式的步骤是：

- 把 a 的初始值存储到临时存储 a0 中

- 把 a0.inc() 结果赋值给 a

- 把 a0 作为表达式的结果返回

### 二元操作

算术运算符如下：

| 表达式 | 翻译成 |
| --- | --- |
|  a + b|  a.plus(b)|
|  a - b| a.minus(b)  |
|  a * b|  a.times(b)|
|  a / b| a.div(b) |
|  a % b|  a.rem(b)|
|  a .. b| a.rangeTo(b)|

对于此表中的操作，编译器只是解析成翻译为列中的表达式

<b>示例：</b>

下面是一个从给定值起始的 Counter 类的示例，它可以使用重载的 + 运算符来增加计数：

~~~
data class Counter(val dayIndex: Int) {
    operator fun plus(increment: Int): Counter {
        return Counter(dayIndex + increment)
    }
}

~~~

<code>In</code>操作符

| 表达式 | 翻译成 |
| --- | --- |
|  a in b|  b.contains(a)|
|  a !in b| !b.contains(a)  |

对于 <code>in</code> 和 <code>!in</code>，过程是相同的，但是参数的顺序是相反的
 
### 索引访问操作符

| 表达式 | 翻译成 |
| --- | --- |
|  a[i]|  a.get(i)|
|  a[i, j]| a.get(i, j)  |
|  a[i_1, ……, i_n]|  a.get(i_1, ……, i_n)|
|  a[i] = b| a.set(i, b) |
|  a[i, j] = b|  a.set(i, j, b)|
|  a[i_1, ……, i_n] = b| a.set(i_1, ……, i_n, b))|

方括号转换为调用带有适当数量参数的 <code>get</code> 和 <code>set</code>

### 调用操作符

| 表达式 | 翻译成 |
| --- | --- |
|  a()|  a.invoke()|
|  a(i)| a.invoke(i)  |
|  a(i, j)| a.invoke(i, j)  |
|  a(i_1, ……, i_n)| a.invoke(i_1, ……, i_n)  |

圆括号转换为调用带有适当数量参数的 <code>invoke</code>

### 广义赋值

| 表达式 | 翻译成 |
| --- | --- |
|  a += b|  a.plusAssign(b)|
|  a -= b|  a.minusAssign(b)|
|  a *= b|  a.timesAssign(b)|
|  a /= b|  a.divAssign(b)|
|  a %= b|  a.remAssign(b)|

对于赋值操作，例如 <code>a += b</code>，编译器执行以下步骤:

- 如果优劣的函数可用
    - 如果相应的二元函数（即<code>plusAssign()</code> 对应于<code>plus()</code>也可以用）
    - 确保其返回值类型是Unit，否则报告错误
    - 生成 a.plusAssign(b)代码

- 否则试着生成 <code>a = a + b </code>的代码（这里包含类型检测：<code>a+b</code>的类型必须是a的子类型）

注意： 赋值在Kotlin中不是表达式

### 相等与不等操作符
  
| 表达式 | 翻译成 |
| --- | --- |
|  a == b|  a?.equals(b) ?: (b === null)|
|  a != b|  !(a?.equals(b) ?: (b === null))|

这些操作符只使用函数 <code>equals(other: Any?): Boolean</code>，可以覆盖它来提供自定义的相等性检测实现。不会调用任何其他同名函数（如 equals(other: Foo)）
 
注意：=== 和 !==（同一性检测）不可重载，因此不存在对他们的约定

这个 == 操作符有些特殊：它被翻译成一个复杂的表达式，用于筛选 <code>null</code> 值。 <code>null == null </code>总是 <code>true</code>，对于非空的 <code>x</code>，<code>x == null</code> 总是 <code>false </code>而不会调用 <code>x.equals()</code>

### 比较操作符

| 表达式 | 翻译成 |
| --- | --- |
|  a > b|  a.compareTo(b) > 0|
|  a < b|  a.compareTo(b) < 0|
|  a >= b|  a.compareTo(b) >= 0|
|  a <= b|  a.compareTo(b) <= 0|

所有的比较都转换为对 <code>compareTo</code> 的调用，这个函数需要返回 <code>Int </code>值
