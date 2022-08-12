# Handler 消息原理

## 前言

该文章主要分析Handler消息机制的原理，在我们了解Handler消息机制之前我们先了解一下，在Handler中几个角色

## Handler中角色

在Handler消息机制中，有几个不得不提的几个组成元素：Handler、Looper、MessageQueue、Message。

- Handler   
  Handler的主要作用是发送和处理消息

- MessageQueue  
  用来存储消息的队列，采用单链表的数据结构

- Looper    
  是一个循环，负责检查消息队列中是否有消息，并负责取出消息

- Message   
  消息的载体

## 基本使用

我们在了解了基本角色后，我们先看看基本的使用：

~~~

~~~
