# 重学Android之----Gradle（一）

## 前言

我们在开始学习Geadel之前，我们应该先了解什么是Gradel，Gradel是一个开源构建自动化工具，设计灵活。详情了解请看官网 [什么是 Gradle？](https://docs.gradle.org/current/userguide/what_is_gradle.html#what_is_gradle)

## Gradle版 HelloWorld

我们在新建项目的时候会发现在app.gradle中发现一下代码:

~~~
task clean(type: Delete) {
    delete rootProject.buildDir
}
~~~

定义一个<code>Clean</code>的任务，这个任务是<code>Delete</code>类型，<code>delete rootProject.buildDir</code> 相当于 <code>delete(rootProject.buildDir)</code>，这是<code>Groovy</code>语法，在<code>Groovy</code>中只要不引起歧义，函数的调用是可以去掉括号。

上面的代码的意思是删除项目的根目录<code>build</code>目录

#### 我们来实现一下Hello World

我们在app.gradle中添加以下代码

~~~
task hello{
    println 'Hello World'
}
~~~

添加以上代码后，在AS中就会出现一个绿色三角，或者右键方法名，点击run，就可以直接运行，我们点击一下直接运行一下，具体操作如图所示：

![1660118408375.jpg](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/351a94cf01ba41fcb54505ade75a8a63~tplv-k3u1fbpfcp-watermark.image?)

运行后我们就会看到HelloWorld的打印

![1660140330621.jpg](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8e6c599473b1480ea06e45eb95498457~tplv-k3u1fbpfcp-watermark.image?)
## Gradle Wrapper

<code>Wrapper</code>其实就是<code>Gradle</code>的一层包装，便于在开发中统一版本，避免多人开发因<code>Gradle</code>版本不统一造成的不必要问题

在Android项目中，<code>Gradle</code>的配置在<code>gradle-wrapper.properties</code>文件中，所有任务都会在这个文件中配置，那我们来看看该文件的配置
|字段名|说明|
|---|---|
|distributionBase|下载的Gradle压缩包解压后存储的主目录|
|distributionUrl|Gradle发行版压缩包的下载地址|
|distributionPath|相对于distributionBase的解压后的Gradle的解压包的路径|
|zipStorePath|同distributionPath，只不过是存放zip压缩文件包的|
|zipStoreBase|distributionBase，只不过是存放zip压缩文件包的|

我们在开发过程中，是比较关注<code>distributionUrl</code>字段，这个字段决定了你的项目中依赖那个<code>Gradle</code>版本。例如
<code>https://services.gradle.org/distributions/gradle-7.3.3-bin.zip</code>我们通常会把<code>bin</code>改为<code>all</code>，这样在开发过程中，我们就可以看<code>Gradle</code>的源代码了。

例如，在最新的Android Stuido中，默认的<code>Gradle</code>版本是<code>7.+</code>
~~~
distributionBase=GRADLE_USER_HOME
distributionUrl=https\://services.gradle.org/distributions/gradle-7.3.3-bin.zip
distributionPath=wrapper/dists
zipStorePath=wrapper/dists
zipStoreBase=GRADLE_USER_HOME
~~~

<code>distributionUrl</code>是<code>Gradle</code>的下载路劲，如果在运行<code>./gradlew</code>的时候发现Android Studio一直被卡着不动，可能是因为官方的<code>Gradle</code>的地址被封，建议修改其他的镜像地址，或者是VPN代理来下载

### 自定义Wrapper Task

<code>gradle-wrapper.properties</code>是由<code>Wrapper Task</code>生成的，那么我们是否可以自定义配置<code>Wrapper Task</code>达到我们配置<code>gradle-wrapper.properties</code>的目的呢？我们只需要在<code>build.gradle</code>构建文件中录入如下脚本代码：

~~~
task wrapper(type:wrapper){
    gradeVersion = '7.3'
}
~~~

这样我们在执行<code>gradle warpper</code>的时候，就会默认生成7.3版本的<code>warpper</code>了，而不用使用<code>--gradle version 7.3</code>进行指定了，同样也可以配置其他参数：

~~~
task wrapper(type:wrapper){
    gradeVersion = '7.3'
    srchiveBase = 'GRADLE_USER_HOME'
    srchivePath = 'wrapper/dists'
    distributionPath = 'wrapper/dists'
    distributionUrl ='https\://services.gradle.org/distributions/gradle-7.3.3-all.zip'
}
~~~
