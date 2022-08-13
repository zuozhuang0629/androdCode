# 重学Android之----Gradle（二）

## Gradle 日志

我们这里先介绍Gradle日志，Gradle日志和Java、Android的差不多，也是分一些等级，以便区分不同类型日志信息，不至于被大量日志搞得晕头转向。

## 日志级别

上面已经说Gradle中日志和大部分语言差不多，但是Gradle有两个独特的级别，<code>QUIET</code>和<code>LIFECYCLE</code>，以便用于标记重要的进度级别的日志信息，具体级别分类如下：

|级别|日志级别|
|---|---|
|ERROR|错误消息|
|QUIET|重要消息|
|WARNING|警告消息|
|LIFFCYCLE|进度消息|
|INFO|信息消息|
|DEBUG|测试消息|

如上表中所示，一共有6中日志级别及其作用，现在我们在看看如何使用，要使用他们，显示我们想要显示级别的日志，就要通过命令行选项中的日志开关来控制。

<b>注意：在Windows中需要配置Gradle的环境变量</b>

在Windows中和在Mac中使用的还有区别
|平台|关键字|
|---|---|
|Windows|gradle|
|Mac|./gradlew|

那我们来通过命令行开关选项来试用一下吧。之前的Task Hello代码中修改一下：
~~~
task hello {
    println 'hello world'
    logger.debug("This is Debug Log Message")
    logger.info("This is Info Log Message")
    logger.warn("This is Warn Log Message")
    logger.lifecycle("This is Lifecycle Log Message")
    logger.quiet("This is Quiet Log Message")
    logger.error("This is Erroe Log Message")
}
~~~

在AndroidStudio中Terminal(终端)输入<code>./gradlew -q tasks</code>就会输出我们之前所写的<code>hello world</code>,和<code>QUIET</code>及以上的日志消息,结果如下：

![WechatIMG37.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8a0bb3db1dbc4ab98596812340a0aad6~tplv-k3u1fbpfcp-watermark.image?)

我们只需要在命令行加上这些选项即可控制使用，如下表：
|开关选项|输出日志的级别|
|---|---|
|无选项|输出 LIFECYCLE 及更高|
|-q|输出 QUIET 及更高|
|-i|输出 INFO 及更高|
|-d|输出 DEBUG 及更高|

## 输出错误堆栈信息

在使用Gradle构建的时候，难免会有一些问题导致你的构建失败，这时就需要你根据日志分析问题。除了上面的信息之外，Gradle还提供了堆栈信息的打印。

默认情况下，堆栈信息的输出是关闭的，我们需要通过命令行开打开它，这样在构建失败的时候，Gradle才会输出错误堆栈信息以便我们定位和解决问题。

|命令行选项|用于|
|---|---|
|无选项|没有堆栈信息输出|
|-s 或 --stacktrace|输出关键性的堆栈信息|
|-S 或 --full-stacktrace|输出全部堆栈信息|

虽然-S会输出全部的信息，但是太长了，非常的难以定位，但-s比较精简，可以快速定位，比较对剑使用

## 使用日志信息调试

在编写Gradle脚本的时候，我们也需要输出一些日志，检验逻辑是否正确，这个时候我们就可以使用Gradle的日志功能

除了print方法，我们也可以使用logger更灵活的控制不同级别的日志

~~~logger
logger.quiet('quiet 日志')
logger.error('error 日志')
logger.warn('warn 日志')
logger.lifecycle('lifecycle 日志')
logger.info('info 日志')
logger.debug('debug 日志')
~~~

## Gradle命令行

如果我们不知道Gradle下的所有命令，我们可以通过帮助来了解。在命令后添加-h或者--help，有的时候也会Gradle Wrapper为例：

~~~
./gradlew -?
./gradlew -h
./gradlew -help
~~~

## 查看所有可执行的Tasks

有时候我们不知道如何构建一个功能，不知道执行哪个Task，这时候就需要查看哪些Task，就可以执行如下代码：

~~~
./gradlew tasks
~~~

## 强制刷新依赖

我们在开发中不可避免的需要一些第三方库，像Maven这类工具都是有缓存的，因为不能每次编译的时候都需要重新下载第三方库，本地有缓存的话，会先使用缓存，没有在下载。有时候刷新一下，我们就可以使用命令行在强制刷新一下。

~~~
./gradlew --refresh-dependencies assemble
~~~

## 多任务调用

我们有的时候需要同事运行多个任务，比如在执行jar之前我们需要先clean一下，那这种情况只需要按顺序空格分开即可

~~~
./gradlew clean jar
~~~

## 通过任务名字缩写执行

有的时候我们的任务名字长的离谱，全部写比较麻烦和耗时，我们就可以使用驼峰命名法的缩写来调用

~~~
方法名：doSomeThink
./gradlew dst
~~~