# 面试必问系列之----Java中 == ，equals()和hashCode()的区别

## 概括
在我们先了解==，equals和hashCode的区别之前，我们先了解一些基本概念，基本数据类型和引用数据类型

- 基本数据类型  
    java中有八大基本数据类型，包括 boolean（布尔型）、float（单精度浮点型）、char（字符型）、byte（字节型）、short（短整型）、int（整型）、long（长整型）和 double （双精度浮点型）共 8 种，详见表 1 所示。

    |  类型名称     | 关键字      |占用内存   |取值范围|
    |  :----:      |  :----:    |:----:     | :----:  |
    |  字节型       | byte      |1 字节     |-128~127|
    |  短整型       | short     |2 字节     |-32768~32767|
    |  整型         | int       |4 字节     |-2147483648~2147483647|
    |  长整型       | long      |8 字节     |-9223372036854775808L~9223372036854775807L|
    |  单精度浮点型  | float    |4 字节     |+/-3.4E+38F（6~7 个有效位）|
    |  双精度浮点型  | double   |8 字节     |+/-1.8E+308 (15 个有效位|
    |  字符型       | char      |2 字节     |ISO 单一字符集|
    |  布尔型       | boolean   |1 字节     |true 或 false|

- 引用数据类型
    简单来说，所有的非基本数据都是引用类型，大致包括：类、接口类型、数组类型、枚举类型、注解类型、字符串类型

## == 的比较  

==的比较分基本数据类型和引用数据类型

1. 基本数据数据类型（int float double bety bool chat long short）是对值得比较

~~~code
int a = 1；
int b = 2；
int c = 1；

System.out.println(a == b);
System.out.println(a == c);

输出：
false
true
~~~

2. 引用数据类型是对地址的比较，而非值本身

~~~
 //用户类
class User{
    //用户名
    String name;
    //年龄
    int age;

    public User( String name,int age) {
        _name = name;
        _age = age;
    }
}

User user1 = new User("zhangsan",16);
User user2 = new User("lisi",20);
User user3 = new User("zhangsan",16);
User user4 = user1;

System.out.println(user1 == user2);
System.out.println(user1 == user3);
System.out.println(user1 == user4);

输出：
false
false
true
~~~
## equals比较
在分析之前我们先一下一下代码的结果  
代码A:
~~~
class Man {
    public Cat(String name) {
        this.name = name;
    }

    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}

Man c1 = new Man("zhangsan");
Man c2 = new Man("zhangsan");
System.out.println(c1.equals(c2)); 

输出：
false
~~~
代码B:
~~~
String s1 = new String("lisi");
String s2 = new String("lisi");
System.out.println(s1.equals(s2)); 

输出：
true
~~~
为什么都是引用数据类型，结果去不一致？我们先看一下equals()方法，equals()是根类Object中的方法，代码如下
~~~
public boolean equals(Object obj) {
    return (this == obj);
}
~~~
由此可知，我们在不重写<code>equals()</code>方法情况下，方法内部是用==来进行比较，比较对象的地址。那么在代码A中<code>Man</code>是引用数据类型，所以我们不重写<code>equals()</code>方法的情况下比较的是引用地址，所以代码A的输出结果为<code>false</code>，那么为什么在代码B中为什么输出结果是true呢？原因很简单，因为<code>String</code>重写了<code>equals()</code>方法，我们看一下<code>String</code>重写<code>equals()</code>的方法
~~~
public boolean equals(Object anObject) {
    //首先判读的地址的判断，地址一样值就会一样
    if (this == anObject) {
        return true;
    }
    //判断是否为string类型
    if (anObject instanceof String) {
        String anotherString = (String)anObject;
        int n = value.length;
        //判断string的长度
        if (n == anotherString.value.length) {
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = 0;
            //进行每个字符的判断
            while (n-- != 0) {
                if (v1[i] != v2[i])
                    return false;
                i++;
            }
            return true;
        }
    }
    return false;
}
~~~
从上面的代码我们可以看到，把object的equals()中的地址比较改成了值比较。
## equals 重写原则
对象内容的比较才是设计equals()的真正目的，Java语言对equals()的要求如下，这些要求是重写该方法是必须遵循：
1. 对称性
~~~
如果a.equals(b)返回true，那么b.equals(a)也应该返回true
~~~
2. 自反性
~~~
a.equals(a)必须返回true
~~~
3. 类推性
~~~
如果a.equals(b)返回true，而且a.equals(c)返回true，那么b.equals(c)也应该返回true
~~~
4. 一致性
~~~
如果a.equals(b)返回true，只要a,b内容一直不变，无论多少次调用equals(),都应该反应true
~~~
5. 对称性
~~~
如果a.equals(b)返回是true，那么b.equals(a)也应该返回是true
~~~
<b>注意:</b>        
    <b>在任何情况下a.equals(null)必须返回false</b>  
    <b>在任何情况下a.equals(不属于a的其他类型)必须返回false</b>

## hashCode()
<code>hashCode()</code>方法返回的就是一个数值，从方法的名称上就可以看出，其目的是生成一个hash码。hash码的主要用途就是在对对象进行散列的时候作为key输入，据此很容易推断出，我们需要每个对象的hash码尽可能不同，这样才能保证散列的存取性能。事实上，<code>Object</code>类提供的默认实现确实保证每个对象的hash码不同（在对象的内存地址基础上经过特定算法返回一个hash码）。Java采用了哈希表的原理。哈希（Hash）实际上是个人名，由于他提出一哈希算法的概念，所以就以他的名字命名了。 哈希算法也称为散列算法，是将数据依特定算法直接指定到一个地址上。初学者可以这样理解，<code>hashCode</code>方法实际上返回的就是对象存储的物理地址（实际可能并不是）

<b>简单来说:
    <code>hashCode()</code>是为了集合操作快速，而根据一定规则而设计的散列码，为了方便比较插入数据，用于<code>HashMap</code>,<code>HashSet</code>,<code>HashTable</code></b>     

那么<code>hashCode()</code>与<code>equal()</code>有什么区别和联系呢？   
在java中对于<code>eqauls</code>方法和<code>hashCode</code>方法是这样规定的：

- 同一对象上多次调用<code>hashCode()</code>方法，总是返回相同的整型值

- 如果<code>a.equals(b)</code>，则一定有<code>a.hashCode()</code> 一定等于 <code>b.hashCode()</code>

- 如果<code>!a.equals(b)</code>，则<code>a.hashCode()</code> 不一定等于 <code>b.hashCode()</code>。此时如果<code>a.hashCode()</code> 总是不等于 <code>b.hashCode()</code>

- <code>a.hashCode()==b.hashCode()</code> 则 <code>a.equals(b)</code>可真可假

- <code>a.hashCode()！= b.hashCode()</code> 则 <code>a.equals(b)</code>为假。

## 为什么重写equals()必须重写hashCode()
重写<code>hashCode()</code>是为不相等的对象（equals为false）产生不相等的散列码，而相等的对象（equals为true）必须拥有相等的散列码

我们以<code>HashSet</code>为例，<code>HashSet</code>使用的是<code>HashMap</code>的<code>put</code>方法，而<code>hashMap</code>的<code>put</code>方法，使用<code>hashCode()</code>用<code>key</code>作为参数计算出<code>hashCode</code>，然后进行比较，如果相同，再通过<code>equals()</code>比较<code>key</code>值是否相同，如果相同，返回同一个对象。

所以，如果类使用再散列表的集合对象中，要判断两个对象是否相同，除了要覆盖<code>equals()</code>之外，也要覆盖<code>hashCode()</code>函数，否则，<code>equals()</code>无效。

所以，在<code>HashSet</code>中，一定要同时重写<code>hashCode()</code>和<code>equals()</code>，<code>HashSet</code>底层是由于<code>HashMap</code>的数据结构（数组+链表/红黑树）的比较逻辑决定的。
 




  

















