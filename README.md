# Performance-optimization

# 一. 性能优化方向:

##### 1.内存优化
##### 2.卡顿优化
##### 3.布局优化
##### 4.启动优化
##### 5.包体积优化
##### 6.网络优化
##### 7.电量优化
##### 8.业务角度优化
##### 9.编译优化

# 二. 性能优化难点:

##### 1.没有统一的标准
##### 2.用户的机器环境相关性较大

# 三. 文档资料:

## 专题文档:

[苹果官方 Performance 专题](https://developer.apple.com/library/archive/navigation/#section=Topics&topic=Performance)

## 书籍文档:
[High Performance iOS](./doc/OReilly.High.Performance.iOS.Apps.2016.6.pdf)

[Pro iOS App Performance Optimization](./doc/Pro_ios_apps_performance_optimization.pdf)

## WWWDC 文档:

[WWDC文档](./doc/WWDC/)

# 四. 业界方案:

[微信读书 iOS 性能优化总结](http://wereadteam.github.io/2016/05/03/WeRead-Performance/)

工具及方法：

##### 数据收集方法：

现网用户的卡顿状况通过接入bugly卡顿监控，通过下发配置，对现网用户进行抽样检测，bugly的依据是监控主线程Runloop的执行，观察执行耗时是否超过预定阀值(默认阀值为3000ms)。在监控到卡顿时会立即记录线程堆栈到本地，在App从后台切换到前台时，执行上报。

##### 使用到的工具：

##### 1. 内存泄露检测工具 ****MLeakFinder**** 

[TODO 原理介绍]

##### 2. FPS/SQL性能监测工具条
该工具条是在DEBUG模式下，以浮窗的形式。实时展示当前可能存在问题的FPS次数和执行时间较长的SQL语句个数，随时查看FPS低于某个阈值时的堆栈信息，再结合当时的使用场景，开发人员使用起来非常便利，可以很快定位到引起卡顿的场景和原因

因此在DEBUG阶段，我们监测了每一条SQL语句的执行速度，一旦执行时间超出某个阈值，就会表现在工具条的数字上，点击后可以进一步查询到具体的SQL操作以及实际耗时。

##### 3. UI/DataSource主线程检测工具

由于大部分UI操作是非线程安全，所以在非UI线程中操作UI可能会导致app突然丢动画，UI操作偶尔响应特别慢，莫名的crash 这些问题。UI/DataSource主线程检测工具通过 hook UIView 的 ****-setNeedsLayout****，****-setNeedsDisplay****，****-setNeedsDisplayInRect**** 三个方法，确保它们都是在主线程执行。
