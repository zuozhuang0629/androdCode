# Android 知识总结
## Java 知识总结
### Java 基础
* Java中传递方式？   
   结论： **<font color=red>Java中只有值传递，没有应用传递</font>**   
    在开始之前,我们先搞懂以下两个概念  
    * 形参&实参
    * 值传递&引用传递  
**形参&实参**
  
  ```
    String hello = "Hello!";
    // hello 为实参
    sayHello(hello);
    // str 为形参
    void sayHello(String str) {
        System.out.println(str);
    }
```
* Java 注解
* Java 范型
* Java 反射
* Java Jvm
* Java 回收机制
* 数据结构
* 线程