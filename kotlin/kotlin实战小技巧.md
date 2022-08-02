# Kotlin实战开发小技巧
## Kotlin代码简洁技巧

对函数的简洁     
常规函数的声明：
~~~
fun sum(a:Int,b:Int):Int{
    return a+b
}
~~~
简洁方法一：
可以省略return直接=返回值
~~~
fun sum(a:Int,b:Int):Int = a+b
~~~
简洁方法二：
可以省略返回数据类型
~~~
fun sum(a:Int,b:Int)= a+b
~~~