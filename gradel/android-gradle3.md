# 重学Android之----Gradle（三）


## 前言

我们知道<code>Gradle</code>是<code>Groovy</code>语言来编写的，<code>Groovy</code>是一种基于<code>JVM（java虚拟机</code>的敏捷开发语言，它结合了<code>Python</code>、<code>Ruby</code>和<code>Smalltalk</code>语言的许多强大特性，<code>Groovy</code>代码能够与<code>Java</code>代码很好地结合，也能用于扩展现有代码。由于其运行在 <code>JVM </code>上的特性，<code>Groovy</code>也可以使用其他非<code>Java</code>语言编写的库。

本章将会带你学习<code>Groovy</code>的基础语法

## 字符串

任何一门语言都会有字符串，基本也会从字符串学起，相比<code>Java</code>而言，<code>Groovy</code>非常方便，比如字符串的运算、求值、正则等。

> 在<code>Groovy</code>中，和<code>Kotlin</code>一样，分号可以去掉，没有强制要求。所以在<code>Gradle</code>脚本中没有分号，这是<code>Groovy</code>的特性，不是<code>Gradle</code>的特性。

在<code>Groovy</code>中，字符串可以使用<code>单引号</code>或者<code>双引号</code>，者都已定义一个字符串常量，但他们有所区别

- 单引号：只能标记纯粹的字符串常量
- 双引号：可以在字符串中进行表达式运算

看一下实例：


```Groovy
task printStr {
    def str1 = "hello"
    def str2 = "world"
    println str1 + str2 + "1"
    println "${str1}${str2}2"
    println "$str1 $str2 3"//只有一个变量可以去去掉{}
    println 'hello world4'
    
}
```
输出：
~~~
helloworld1
helloworld2
helloworld 3
hello world4
~~~

解析：

双引号的字符串是可以做表达式的操作，所以在双引号中可以用<code> $ </code>或者<code> ${} </code>的形式声明调用变量，这里是和<code>Kotlin</code>的一样，单引号就只能做简单的字符串声明，不能进行表达式。

## 集合

<code>Groovy</code>中的集合完全兼容了<code>Java</code>的集合，还进行了扩展。

### List

在创建<code>List</code>的时候，和<code>Kotlin</code>中一样不需要使用 <code>new</code>

~~~
task printList {
    def numlist = [1,2,3,4,5,6]
    println numlist.getClass().name
    for (def i = 0; i< numlist.size();i++){
        println numlist.get(i)
    }

}
~~~

输出：

~~~
java.util.ArrayList
1
2
3
4
5
6
~~~

解析：

我们声明了一个<code>list</code>，不需要用<code>new</code>，直接<code>[]</code>表示，这里我<code>Kotlin</code>，再使用<code>for</code>循环遍历一遍。

我们发现numlist是一个
ArrayList类型。那么对于ArrayList中可以操作的和java一样，但是语法和Kotlin一致。

~~~
task printList {
    def numlist = [1,2,3,4,5,6]
    println numlist[1]
    println numlist[-1]
    println numlist[-2]
    println numlist[1..3]
}
~~~

输出：

~~~
2
6
5
[2, 3, 4]
~~~

解析：

<code>Groovy</code>提供了下标索引的方式访问，就像数组一样，还提供了<b>负下标</b>和<b>范围索引</b>。

- 负小标：代表从右边开始，-1 代表右边的一个
- 范围索引：代表是一个区间范围,..来分开。1..3表示1到3的范围

当然在迭代方面可以使用上面的<code>for</code>循环，也可以下面的<code>each</code>

~~~
numlist.each{
    println it
}
~~~

这都是和Kotlin中一样，不过多解释。

## Map

<code>Map</code>和<code>List</code>相似，<code>Map</code>是以<code>键值对<K,V></code>的形式。
我们看一下使用。

~~~
task printMap{
    def map = [1:"one",2:"two"]
    println map.getClass().name
    println map[1]
    println map.size()
}
~~~

输出：

~~~
java.util.LinkedHashMap
one
2
~~~

解析：
我们发现map是LinedHashMap。用法也和Kotlin一样

## 方法

<code>Groovy</code>方法不像<code>Java</code>使用<code>void</code>也不想<code>Kotlin</code>使用<code>fun</code>声明，这里是使用<code>def</code>。

### 括号可以省略

在Java或者Kotlin中调用方法都需要()来调用方法，我们直接看一下实例：

~~~
task testFun{
    sum(1,2)
    sum 1,2
}

def sum(int a, int b){
    println a+b
}

~~~
输出：
~~~
3
3
~~~

解析：

~~~
调用方法的时候可以加上括号，也可以不用加，这样的语法会更简洁
~~~

### return可以省略
在方法中需要返回数据，java中就需要使用return来返回，但在Groovy中就可以省略，这样Groovy就可以吧方法的最后一句代码作为返回值

~~~
task testFun{
    def sum1 = sum 1,2
    def sum2 = sum 8,2
    println sum1
    println sum2
}

def sum(int a, int b){
    a+b
}
~~~

输出：

~~~
3
10
~~~

解析：

我们在<code>sum</code>方法中没有使用<code>return</code>返回<code>a+b</code>，但是结果还是返回了<code>a+b</code>

## 代码快可以作为参数传递

代码快就是一段被花括号包围的代码，也就就是闭包。在Groovy中允许他们作为参数传递，我们就一List中的each来举例

~~~
numlist.each({println it})

//如果最后一个参数是闭包，也可以放到外面
numlist.each(){println it}

//也可以省略圆括号
numlist.each{
    println it
}
~~~

## JavaBean

<code>JavaBean</code>就类似于<code>Kotlin</code>的<code>Data</code>，我们不需要重复生成<code>getter</code>、<code>settter</code>方法

~~~
task hellobean{
    User user =new User()
    //也可以
    //def user =new User()
    user.name = "lisi"
    println user.name

}

class User {
   String name
}
~~~

输出：

~~~
lisi
~~~


> 注意:在创建对象的时候new是不能省略的，这个和List不同。
> 可以用类型来持有，也可以用def来持有

我们在看一下下面这个例子：
~~~
task hellobean{
    User user =new User()
    //也可以
    //def user =new User()
    user.name = "lisi"
    println user.name
    println user.age
}


class User {
   String name

   int getAge(){
    20
   }
}

~~~

解析：

我们定义了一个<code>getAge()</code>方法，我们在调用的时候可以省略<code>get</code>直接<code>age</code>，这里也是和<code>Kotlin</code>语法一样。
