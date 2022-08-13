# Kotlin 泛型使用

## 泛型的定义和作用

Kotlin泛型与Java中的泛型一样，都是一种语法糖，泛型只存在源代码中，到了class级别就会被擦除掉。泛型就是将类型参数化，可用在类、接口、方法上。它提供了代码灵活性和减少类型强转的烦恼。

## 泛型类

泛型类就在泛型的类，我们先声明一个泛型类：

~~~
class Test<T>(t: T) {
    var value = t
}
~~~

那么我们在创建的时候就需要制定类型参数，要不然编译器会报错，所有的泛型都会在编译之前检测，真正编译成class文件后，在class文件中是不存在泛型的感念。

~~~
val test: Test<Int> = Test<Int>(1)
// 或者
val test = Test(1) // 编译器会进行类型推断，1 类型 Int，所以编译器知道我们说的是 Test<Int>。
~~~

我们在看看传入其他的类型：

~~~
class Test<T>(t : T) {
    var value = t
}

fun main(args: Array<String>) {
    var testInt = Test<Int>(10)
    var testString = Test<String>("Test run")

    println(testInt.value)
    println(testString.value)
}
~~~

输出结果：

~~~
10
Test run
~~~

定义泛型变量的时候，可以显式声明类型参数，也可以隐式不声明，让编译器自动推定类型参数。

## 泛型方法

在定义泛型方法的时候，泛型参数需要放在方法名的前面：

~~~
fun <T> testIn(value: T) = Test(value)

// 以下都是合法语句
val test1 = testIn<Int>(1)
val test11 = testIn(1)     // 编译器会进行类型推断
~~~

在调用泛型方法的时候，和泛型类是一样的，如果编译器可以自动推定类型参数，也可以省略类型参数。

在kotlin中泛型可以和when使用，一边做出不同类型的相应处理

~~~
fun main(args: Array<String>) {
    val age = 23
    val name = "lisi"
    val bool = true

    doSome(age)    // 整型
    doSome(name)   // 字符串
    doSome(bool)   // 布尔型
}

fun <T> doSome(some: T) {

    when (content) {
        is Int -> println("整型数字为 $content")
        is String -> println("字符串转换为大写：${content.toUpperCase()}")
        else -> println("T 不是整型，也不是字符串")
    }
}

~~~

输出结果：

~~~
整型数字为 23
字符串转换为大写：LISI
T 不是整型，也不是字符串
~~~

## 泛型约束

当我们咋定义一个泛型类、泛型接口或者泛型方法的时候我们不允许任何类型的都可以使用，那么这个时候我们可以对泛型进行约束。

泛型约束：分为<code>上界约束</code>和<code>下界约束</code>，但是在kotlin中没有下界约束

### 上界约束

上界约束就是类型参数必须要是定义类型的类或子类，我们通过<code>:</code>来对泛型类型上线进行约束，例如：

~~~
//定义一个父类
open class User(){

}

//继承User类
class Lisi : User(){

}

//限制参数类型的上边界，必须是User类或者其子类
class Test<T : User>(){

}

fun main(){
    val test = Test<Int>()//编译器报错
    val test2 = Test<User>()//正常
    val test3 = Test<Lisi>()//正常
}
~~~

在设置了泛型上边界的时候，参数类型必须是设置上界的类型及其子类。

## 型变

我们要知道在Kotlin中是没有通配符的，但是他有两个其他的：<code>声明处型变</code>与<code>类型投射</code>

### 声明处型变

声明处的类型变异使用协变注解修饰符：<code>in</code>、<code>out</code>。

消费者 <code>in</code>, 生产者 <code>out</code>

使用<code>out</code>使得一个类型参数协变，协变类型参数只能用作输出，可以作为返回值类型但是无法作为入参的类

~~~
// 定义一个支持协变的类
class Test<out A>(val a: A) {
    fun foo(): A {
        return a
    }

    //编译器会在A的地方标红，提示移除out，因为out的泛型不能作为参数使用，只能作为输出来使用
    fun foo(b :A): A {
        return a
    }
}

fun main(args: Array<String>) {
    var strCo: Test<String> = Test("a")
    var anyCo: Test<Any> = Test<Any>("b")
    anyCo = strCo
    println(anyCo.foo())   // 输出 a
}

~~~

理解了out那么in就好理解了，它们是相反的。

in 使得一个类型参数逆变，逆变类型参数只能用作输入，可以作为入参的类型但是无法作为返回值的类型：

~~~
// 定义一个支持逆变的类
class Test<in A>(a: A) {
    //编译器会在返回值A的地方标红，原理我就不说了
     fun foo(a: A) :A{
            return  a
        }
}

fun main(args: Array<String>) {
    var strDCo = Test("a")
    var anyDCo = Test<Any>("b")
    strDCo = anyDCo
}
~~~

## 星型投影

有的时候你会发现在定义泛型的时候是这样<code><*></code>,这个被称之为星型投影，用来表明不知道关于泛型实参的任何信息。

<code><*></code>在修饰容器的时候，只能读不能写，相当于
<code><out Any?></code>

比如：<code>MutableList<*></code>表示的是 <code>MutableList<out Any?></code>

~~~
fun demo() {
    val list = mutableListOf<String>()
    list.add("s1")
    list.add("s2")
    reportInfo(list)
}

fun reportInfo(info: MutableList<*>) {
    //不能add，赋值报错
    //info[1] = "s0"
    
    //可以get
    println(info[1])
    print(info)
}


@JvmStatic
fun main(args: Array<String>) {
    demo()
    println("")
}
~~~

