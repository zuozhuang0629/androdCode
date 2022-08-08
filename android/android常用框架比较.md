# Android 框架

## 前言

随着Android技术的成熟，Android应用框架设计得到了越来越多企业和开发者的重视，并衍生了Android框架师的职业。好的框架设计会带来很多好处，比如更易维护、扩展等等。目前Android的框架主要有MVC、MVP、MVVM、MVI。

## MVC模式

MVC(Model-View-Controller 模型-视图-控制器),这是Android最早提出的设计模式，他用一种业务逻辑、数据、界面显示分离的方法组织代码，Android中MVC的角色定义如下：

- Model(模型层)     :负责处理数据的加载或存储
- View(视图层)      :负责呈现从数据层传递的数据渲染工作，一般采用xml文件或者Java自定义View，当然也可以使用JavaScript+Html方式来作为View层
- Controller(控制层):负责Model层和View层的连接，在Android中的控制层通常在Activity、Fragment
  
MVC简单来说就是通过Controller层来操作Model层的数据，并且返回View层来展示
图片
MVC的缺点：

- Activity在Android中并不是标准的Controller，它首要的职责是负责加载应用的布局和初始化用户界面，接受并处理来自用户的操作请求，并做出响应，随着界面及逻辑的复杂度不断提升，Activity的职责不断增加，以至于Activity变的很臃肿

- View层和Model层相互耦合，不易维护和开发

为了解决这些问题，产生了MVP和MVVM，MVI这三种框架，这里不敲点了，想必读者已经非常了解了，接下来我们MVP模式

## MVP模式

MVP(Model-View-Presenter)是MVC的演化版，MVP角色定义如下：

- Model(模型型): 主要提供数据的存取功能。Presenter通过Model层来存储、获取数据

- View(视图层):负责展示数据、处理用户事件。在Android中Activity、Fragment也属于View层但是负责展示不负责处理逻辑

- Presenter(通知层):作为View层和Model层之间沟通的桥梁，它从Model层获取的数据返回给View层，使的View层和Model层没有耦合

在MVP中Presenter层完全将Model层和View层进行了解耦，主要逻辑在Presenter层里实现。而且Presenter层与View层没有直接的关联，而是定义好的接口来进行交互，从而使得更变View时可以保持Presenter的不变，更易开发与维护。
图片

### MVP模式的应用
我们这里举一个小例子来简单实现一下MVP模式，我们就模拟一下获取用户信息，展示信息的案例

1. 实现Model层

~~~
//定义一个用户类
class UserInfo{
    String name;
    int age;
}
~~~

2. 定义一个View接口的基类

~~~
public interface BaseView<T>{
    void setPresenter(T presenter);
}
~~~

3. 实现Presenter层

~~~
//一般习惯定义一个Contract接口存放Presenter接口和View接口，减少接口过多的问题
public interface IUserInfoContract{
    interface Presenter{
     //获取用户信息
     UserInfo getUserInfo();
     //添加用户信息
     void adddUserInfo(UserInfo userInfo);
    }

    interface View extends BaseView<Presenter>{
        //显示用户信息
        void showUserInfo(UserInfo userInfo);
    }
}
~~~

4. 我们Activity中展示

~~~
class UserInfo extends Activity implements IUserInfoContract.Presenter
~~~