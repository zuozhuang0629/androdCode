  # Java中传递方式？   
   ## 结论： Java中只有值传递，没有引用传递
    
在开始之前,我们先搞懂以下两个概念  
* 形参&实参
* 值传递&引用传递  
### 形参&实参
  方法的定义可能会用到 参数（有参的方法），参数在程序语言中分为：
  * **实参（实际参数）** ：用于传递给函数/方法的参数，必须有确定的值。
  * **形参（形式参数）** ：用于定义函数/方法，接收实参，不需要有确定的值。

  ```
    String hello = "Hello!";
    // hello 为实参
    sayHello(hello);
    // str 为形参
    void sayHello(String str) {
        System.out.println(str);
    }
```

### 值传递&引用传递
程序设计语言将实参传递给方法（或函数）的方式分为两种
* **值传递** ：方法接收的是实参值的拷贝，会创建副本。
* **引用传递**： 方法接受的直接是实参所引用的对象在堆中的地址，不会创建副本，对形参的修改会影响到实参
  
## 为什么Java只有值传递？
### 案例1:传递基本数据类型参数
~~~
    public static void main(String[] args) {
    int num1 = 10;
    int num2 = 20;
    swap(num1, num2);
    System.out.println("num1 = " + num1);
    System.out.println("num2 = " + num2);
}

public static void swap(int a, int b) {
    int temp = a;
    a = b;
    b = temp;
    System.out.println("a = " + a);
    System.out.println("b = " + b);
} 
~~~

输出结果：
~~~
    a=20
    b=10
    num1=10
    num2=20
~~~
分析：  
在swap()方法中，<code>a</code>，<code>b</code>的制进行交换，影不影响<code>num1</code>，<code>num2</code>。因为，<code>a</code>，<code>b</code>的值，只是将<code>num1</code>，<code>num2</code>的值进行复制过来的。也就是说，<code>a</code>，<code>b</code>相当于<code>num1</code>，</code><code>num2</code>的副本，副本的内容无论怎么修改，都不会影响到原数据本身

![alt 属性文本](./1.jpeg)
