---
theme: channing-cyan
---
携手创作，共同成长！这是我参与「掘金日新计划 · 8 月更文挑战」的第二十二天，[点击查看活动详情](https://juejin.cn/post/7123120819437322247 "https://juejin.cn/post/7123120819437322247")

# 重写Android之View的工作原理系列（一）

## 前言

本系列主要介绍两个方面，首先介绍View的工作原理，懂了原理后我们在来实现自定义View。

我们知道在Android中，View是很重要的知识点，我们使用的`TextView`、`ImageView`等都是继承与`View`，当然这些都是Android的`GUI`库提供的基本控件，有时候并不能满足我们的日常开发，我们也需要自定义`View`，通过自定义`View`可以实现复杂的一些效果，所以为了我们更好的自定义`view`，我们需要先了解`View`的工作原理

## ViewRoot和DecorView

我们了解一下几个基本的概念：

- Activity
  
`Activity`只是控制生命周期和处理事件，并不是控制视图显示，真正控制视图的`Window`。一个`Activity`包含一个`Window`，`Window`才是真的窗口。

- Window

`Window`是视图的承载器，内部持有一个`DecorView`，这个`DecorView`才是`View`的根布局。

`Window`是一个抽象类，实际在`Acitiy`中持有的是其子类`PhoneWindow`。`PhoneWindow`中一个内部类`DecorView`，通过`DecorView`来加载`Activity`中设置的布局。`Window` 通过`WindowManager`将`DecorView`加载其中，并将`DecorView`交给`ViewRoot`，进行视图绘制以及其他交互。

- DecorView

`DecorView`是`FrameLayout`的子类，它是Android视图树的根节点视图。`DecorView`作为顶级`View`，一般情况下它内部包含一个竖直方向的`LinearLayout`，在这个`LinearLayout`里面有上下三个部分，上面是个`ViewStub`,延迟加载的视图（应该是设置`ActionBar`,根据Theme设置），中间的是标题栏(根据Theme设置，有的布局没有)，下面的是内容栏。我们可以看一下下面的具体布局结构：


![381661246970_.pic.jpg](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6db8bb1d5adf462d9415dce20c56f5c9~tplv-k3u1fbpfcp-watermark.image?)

- ViewRoot

所有`View`的绘制以及事件分发等交互都是通过它来执行或传递的。`ViewRoot`对应于`ViewRootImlp`类，它是链接`WindowManager`和`DecorView`的纽带，`View`的三大流程是通过`ViewRoot`来完成的。

 View的绘制流程是从`ViewRoot`的`performTraversals`方法开始的，它经过`measure`、`layout`和`draw`三个过程才能最终将一个`View`绘制出来，其中`measure`用来测量`View`的宽和 高，`layout`用来确定`View`在父容器中的放置位置，而`draw`则负责将`View`绘制在屏幕上。针对`performTraversals`的大致流程：

![performTraversals.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c0efb1b152544889a5e08a29a8ceee25~tplv-k3u1fbpfcp-watermark.image?)

如上图所示，`performTraversals`会依次调用`performMeasure`、`performLayout`和`performDraw`三个方法，这三个方法分别完成顶级`View`的`measure`、`layout`和`draw`这三大流程。其 中在`performMeasure`中会调用`measure`方法，在`measure`方法中又会调用`onMeasure`方法，在`onMeasure`方法中则会对所有的子元素进行`measure`过程，这个时候`measure`流程就从父容 器传递到子元素中了，这样就完成了一次`measure`过程。接着子元素会重复父容器的`measure`过程，如此反复就完成了整个`View`树的遍历。同理，`performLayout`和`performDraw`的 传递流程和`performMeasure`是类似的，唯一不同的是，`performDraw`的传递过程是在`draw`方法中通过`dispatchDraw`来实现的，不过这并没有本质区别。

`measure`过程决定了`View`的宽/高，`Measure`完成以后，可以通过`getMeasuredWidth`和`getMeasuredHeight`方法来获取到`View`测量后的宽/高，在几乎所有的情况下它都等同于 `View`最终的宽/高，但是特殊情况除外，这点在本章后面会进行说明。