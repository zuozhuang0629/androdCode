  # Java中传递方式？   
   ## 结论： Java中只有值传递，没有应用传递
    
    在开始之前,我们先搞懂以下两个概念  
    
---

    * 形参&实参
    * 值传递&引用传递  
### 形参&实参
  
  ```
    String hello = "Hello!";
    // hello 为实参
    sayHello(hello);
    // str 为形参
    void sayHello(String str) {
        System.out.println(str);
    }
```