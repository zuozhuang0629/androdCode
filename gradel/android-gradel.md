# 重学Android之----Gradel（一）

## 前言

我们在开始学习Geadel之前，我们应该先了解什么是Gradel，Gradel是一个开源构建自动化工具，设计灵活。详情了解请看官网 [什么是 Gradle？](https://docs.gradle.org/current/userguide/what_is_gradle.html#what_is_gradle)

## Gradle版 HelloWorld

我们在新建项目的时候会发现在app.gradle中发现一下代码:

~~~
task clean(type: Delete) {
    delete rootProject.buildDir
}
~~~


上面的代码的意思是删除项目的根目录build目录