# 1.1 环境 #


##1.2Java基础##
Java基本语法、面向对象相关的基本概念与思想，常用String类的api，异常处理，IO基础，容器，多线程，内存管理与垃圾回收，
 

 

Android四大基本组件介绍与生命周期
http://www.cnblogs.com/bravestarrhu/archive/2012/05/02/2479461.html

 


## Activity 四种启动模式 ##

##Intent##

## Context##

## Service  ##

## Service 传递数据、绑定 Service ##

 
## 跨应用启动 Service   AIDL ##


## 广播接收器 BrocastReceiver ## 
BroadcastReceiver 的广播类型与不同的注册方式的区别

## Log ##


## Android 权限 ##



# 3. 用户界面优化 #

## Fragment ##
Fragment的生命周期，Fragment与Activity之间的关系

## LinearLayout 等基本布局 ##


## RecyclerView##


## View基本控件 ##
可下拉的PinnedHeaderExpandableListView的实现
http://blog.csdn.net/singwhatiwanna/article/details/25546871


## Menu ##


## 下拉刷新 ##



## 自定义的视图（View） ##
Android Canvas绘图详解（图文）
http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2012/1212/703.html



Android - Canvas 简单总结
http://www.cnblogs.com/hwgt/p/5416866.html

Android画图Path的使用
http://www.cnblogs.com/tt_mc/archive/2012/12/07/2807518.html


自定义View
http://www.cnblogs.com/lhyz/p/4430409.html



## Android视图动画 ##


## 资源文件配置xml ##


## Drawable ##


## ViewPager ##


## DrawerLayout  ##


## 侧边Menu ##


## 导航栏 ##


## SurfaceView ##


## Toast和Notification ##


## 动画Animation ##

动画相关也是必须掌握的，不管是矢量动画还是属性动画的api都应该熟练，一些简单的动画应该随手就能写出来才行。

4. 系统功能

触摸事件

硬件传感器


安全机制


5. 数据存储

文件读写：读取Assets与raw文件夹中的数据

首选项SharedPreference 


SQLite

应用间数据传递ContentProvider


XML/JSON

6. 网络通信


网络编程相关的基础知识要掌握，如http协议相关，如
http method, status code, request & response, 
http cache, request header, params 等，
Android请求网络相关的api，虽然现在成熟的网络请求库很多，
但是自己应该试着用 HttpUrlConnection 封装一个网络库，哪怕封装的很烂，自己也要尝试着写一下。


异步任务处理

Http、


Socket


ZXing 

 

8. Android 主流开源库

 

事件总线分发Event Bus 

 

视图切换SwitchLayout 

ORM

 

Html、XML等解析

非空格式验证、手机、邮件等格式的验证框架 Android Validation

图片的单点、触摸缩放的库 PhotoView 的使用

播放 Gif 动画图片的控件GifView 的使用


9. Android NFC应用开发
。。。


10. Android 测试

Android UiAutomator

Monkey


11. Android DEMO开发

新闻客户端

自动检测并更新到最新版本

微博

新浪微博有api，我就基于新浪微博api写个简单的微博客户端，有多简单呢？我甚至只能查看微博，其他啥都干不了，完成了查看这一步，再接着慢慢完善其他功能，不要觉得写一个微博客户端遥不可及。如果微博需要登录授权，可能稍难点，

有更简单的直接读取数据的，如知乎日报，如对糗百进行数据抓包，写一个糗百的简易客户端，这类就完全不用授权，

再比如我写个天气的客户端，关于天气现成的接口不要太多。

