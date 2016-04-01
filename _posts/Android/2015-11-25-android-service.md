---
layout: post
title: Android Servcie 后台服务总结笔记
category: Android
tags: [android-service]
---


Android通过Activity提供了界面给用户，而App进入后台时用户就不能通过界面操作App了。但某些场景下，用户确实不需要界面，但仍需使用App，比如播放音乐或者获得博文更新的通知。Android通过Service达到这个目的。
因此，Service就被称作**『后台服务』**。

## 创建 IntentService 以在后台逐个处理用户命令

关于 IntentService 需要说的是：

- 继承自 `Service`；
- 就像启动 Activity，**发送 intent 就可以启动 IntentService**；
*（服务的 intent 又被称作命令）*
- **接收到首个 intent 时创建一个 IntentService 实例**，它在内部创建并启动一个 `HandlerThread`，**在 thread 中处理 intent，所以不会阻塞 UI 线程，处理完毕后 IntentService 自行销毁**；
*（英文手册中称 HandlerThread 为 worker thread，以区分 UI thread）*
- 如果在 IntentService 自毁前，又陆续收到 intent，那么这些 **intent 将被保存在 handler thread 的队列中，按顺序逐个处理**，执行完队列中的全部 intent 后，才会自毁。

<!-- more -->

### 实现 IntentService

```java
public class HelloService extends IntentService {
  public HelloService() {
      super("HelloService");
  }

  @Override
  protected void onHandleIntent(Intent intent) {
      // Normally we would do some work here, like download a file.
  }
}
```

### 启动 IntentService
就像 Activity，通过 intent 启动一个 service，同时需要在 manifest 中声明该 service 为 app 的一个组件：

```java
Intent intent = new Intent(this, HelloService.class);
startService(intent);

<application ... >
  <service android:name=".HelloService" />
</application>
```

### IntentService 的工作原理
阅读[ IntentService 源码 ](http://grepcode.com/file/repository.grepcode.com/java/ext/com.google.android/android/5.1.1_r1/android/app/IntentService.java) 很容易理解它的工作原理：
它在内部维护了一个  HandlerThread 消息循环。当系统的其它组件（比如 Activity）调用 `startService(intent)` 时，系统就会执行 Service 的生命周期方法 `onStartCommand(...)`。IntentService 实现了 onStartCommand(...)，把 intent 封装成一个 Message 发送给 handler 处理，而 handler 调用了抽象方法 `onHandleIntent(Intent intent)`，因此我们实现 onHandleIntent(...) 就可以在由 IntentService 建立的 handler thread 中处理用户发出的命令；处理完后就调用了 `stopSelf(msg.arg1)` 以结束自己.

由于 HandlerThread 消息队列的特性，只能逐个处理收到的 intent，而 handler 每处理完一个 intent，都会调用 `stopSelf(int)`，为什么处理完队列的最后一个 intent 时，`IntentService.onDestroy()` 才被调到？应该是内部会做判断，并未继续追踪代码以了解其判断细节。

IntentService 的 Class OverView 就是这样描述的：

> IntentService is a base class for Services that handle asynchronous requests (expressed as Intents) on demand. Clients send requests through startService(Intent) calls; the service is started as needed, handles each Intent in turn using a worker thread, and stops itself when it runs out of work.

如果不熟悉消息循环，可以参阅 [Android_Message_Handler_消息处理机制总结笔记](http://segmentfault.com/a/1190000003862319)

### IntentService 的生命周期方法调用

如果 `onHandleIntent(...)` 仅仅是睡眠5s，那么用户连续发送4个请求，过一会儿再发送1个请求，生命周期方法的调用情况如下：

```xml
22:19:49.069: onCreate()       // 调用startService()，如果service未被创建，则onCreate()执行1次；
22:19:49.088: onStartCommand() // 第1个intent被封装成message发送给handler；
22:19:49.117: handleMessage()  // handler处理第1个intent；
22:19:49.720: onStartCommand() // 再次调用startService()，service运行中，所以onCreate()不被调用；
22:19:50.270: onStartCommand() // 第3个intent，放入handler的work queue；
22:19:50.828: onStartCommand() // 第4个intent；
22:19:54.118: handleMessage()  // 5s后，处理完第1个intent，处理第2个；
22:19:59.120: handleMessage()  // 5s后，处理第3个；
22:20:04.122: handleMessage()  // 5s后，处理第4个；
22:20:09.125: onDestroy()      // 队列中的intent都被处理完后，stopSelf()才能真正调用到onDestroy();

22:20:37.042: onCreate()       // service被销毁后，调用startService()，onCreate()会被调用。
22:20:37.046: onStartCommand()
22:20:37.246: handleMessage()
22:20:42.249: onDestroy()      // 只发送了一个intent，所以5s后intent处理完，onDestroy()被调用。
```

### 继承 Service 以在后台并发处理用户命令
IntentService 继承自 Service，实现了必须的 Service 生命周期方法，继承 IntentService 只需要一个构造器，并覆写 `onHandleIntent(Intent intent)`即可。并且大多数情况下，IntentService 能满足我们的需求。
如果需要并发执行用户的请求命令，可以模仿 IntentService，自定义一个 Service 子类。和 IntentService 的区别在于，需要在 `onStartCommand(...)`中为每一个 intent 新建一个 thread，以实现并发。另外一点区别是，我们必须调用 `stopService(Intent intent)` 结束服务。

**上述讲到的 service 都是由系统的其它组件（比如 Activity）调用 startService() 创建并启动， 这是 service 的使用方式之一，称为 Started。** 一个 started service 执行单一的任务，不会返回结果给调用者，执行结束后就自行销毁。我们至多能在 started service 中发送 Notification 到状态栏，或者发送 Broadcast 给某个 BroadcastReceiver. **如果想进一步和 service 通信，诸如发送请求、接收反馈，需要采用 service 的另一种使用方式： Bound Service.**

## 创建绑定服务 Bound Service

关于 bound service 需要说的是：

官方文档称 bound service 提供了 **client-server interface**，以允许应用程序组件与 service 交互：service 是 server 端，它实现了一些 public method 供 client 端使用；应用程序组件（比如 activity）就是 client 端。问题是，client 端怎么调用 service 提供的方法呢？

不同于 start 一个 service；更不是在 client 端 new 一个 service 实例，然后调用 service 的 public method。
client 端需要通过 android 提供的机制绑定到 service 端，**由系统建立 client-server 的连接（connection），当连接建立后，系统通过回调方法传递给 client** 一个 `IBinder`，借助于 `IBinder` client 就可以和 service 通信交互，也就达成了所谓的『绑定』。

> A bound service **offers a client-server interface** that allows components to interact with the service, send requests, get results ...
> **Android system creates the connection** between the client and service, ..., to **deliver the IBinder** that the client can use to communicate with the service.

那么问题是，这种『机制』是什么？`IBinder` 又是什么？

### 实现 Bound Service 的机制

- 定义一个继承 `Binder` 的私有子类，在方法 `IBinder onBind(Intent intent)` 中返回 `Binder` 子类的实例。
- client 需要实现接口 `ServiceConnection` 的回调方法，绑定时调用 `bindService(...)`，解除绑定时需要调用 `unbindService(...)`。
- 绑定成功后，`ServiceConnection` 的回调方法 `onServiceConnected(..., IBinder service)` 被执行，client 获得 IBinder。

```java
public class LocalService extends Service {
    // Binder given to clients
    private final IBinder mBinder = new LocalBinder();

    public class LocalBinder extends Binder {
        LocalService getService() {
            // Return this instance of LocalService so clients can call public methods
            return LocalService.this;
        }
    }
    
    @Override
    public IBinder onBind(Intent intent) {
        return mBinder;
    }
    
    /** method for clients */
    public int getRandomNumber() {
      return mGenerator.nextInt(100);
    }    
}    


public class BindingActivity extends Activity {
    LocalService mService;
    
    /** Defines callbacks for service binding, passed to bindService() */
    private ServiceConnection mConnection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName className, IBinder service) {
            // We've bound to LocalService, cast the IBinder and get LocalService instance
            LocalBinder binder = (LocalBinder) service;
            mService = binder.getService();
        }
    };
    
    // Call a method from the LocalService.
    int num = mService.getRandomNumber();
}
```

详情参阅 Android developer guide: bound services 这篇文章的两部分内容：[『1. Extending the Binder class』](http://developer.android.com/guide/components/bound-services.html#Binder)， [『2. Binding to a Service』](http://developer.android.com/guide/components/bound-services.html#Binding). 上述代码段截取自该文章。

目前对 `IBinder` 仍不理解，只是记住了『用于进程内和进程间调用。不要直接实现这个接口，而是需要继承 `Binder`』。mark两篇文章以待后续学习，[『1. 图解Android Binder 和 Service』](http://www.cnblogs.com/samchen2009/p/3316001.html), [『2. Android开发：什么是IBinder』](http://blog.csdn.net/niu_gao/article/details/6453218).

Android提供了3种 bound 方法：(1)继承 Binder class，针对 service 为应用私有的情况；(2)使用 Messenger；(3)使用AIDL。 *上述只是第1种方法，后两种还不理解 TODO。*

### Bound Service 的生命周期方法调用

```xml
D/Activity1: onCreate bindService()     // client调用bindService()以绑定一个service；
D/Service: onCreate()                   // 如果service未被创建，则onCreate()执行1次；
D/Service: onBind()                     // 返回IBinder以允许client绑定；
D/Activity1: onServiceConnected()       // 绑定成功时执行该回调函数，client就获得了IBinder，借助于它可以与service通信；

D/Activity2: onCreate bindService()     // 第2个client尝试绑定service；
D/Activity2: onServiceConnected()       // 由于service已运行，所以onCreate()不被调用；
D/Activity2: onDestroy unbindService()  // 第2个client销毁时需要解除绑定；因第1个client仍绑定，所以service不会被销毁；

D/Activity3: onCreate bindService()     // 第3个client...
D/Activity3: onServiceConnected()
D/Activity3: onDestroy unbindService()

D/Activity1: onDestroy unbindService()  // 第1个client解除绑定；
D/Service: onDestroy()                  // 所有client解除绑定后，系统会销毁该service。
```
细心的你应该发现了，即使在 service 被销毁后，`onServiceDisconnected()` 也没被调到，为什么呢？ [解释：『only called in extreme situations (crashed / killed)』](http://stackoverflow.com/a/5293050/2722270)。
所以当 client 解除与 service 的绑定并销毁自己时，最好在自己的生命周期方法内清理与 service 有关的资源。

特别需要注意的是，不要在 `onResume()` 和 `onPause()` 生命周期方法内 bind/unbind service。比如，从 activity1 跳转到 activity2，service 和 activity1 解除了绑定，但 activity2  还未来得及绑定 service，将导致没有任何 client 与 service 绑定，service 会被销毁。

> Note: You should usually not bind and unbind during your activity's onResume() and onPause()...

## 创建前台服务

某些播放器经常出现在下拉状态栏打开的抽屉中（notification drawer）中，用户可以控制音乐的播放/暂停/上首/下首，点击图标后还可以进入App的播放界面。这很可能就是 Foreground Service 的一种实现。

所谓前台服务，就是在创建 service 的时候（或者其它适当的时候），调用 `startForeground (int id, Notification notification)` 发送一个 notification 到状态栏（notification area），notification 包含一个 `PendingIntent`，在用户点击 notification 时启动一个 activity。
[参阅 Running a Service in the Foreground](http://developer.android.com/guide/components/services.html#Foreground)

## Service 和 Thread 的关系

Service运行于 UI 线程，所以如果有耗时的操作，需要在 service 中创建工作线程来完成。

> Caution: A service runs in the main thread of its hosting process—the service does not create its own thread and does not run in a separate process (unless you specify otherwise).

## [使用 Bound Service 完成后台下载任务的 Demo](http://segmentfault.com/q/1010000004001588)


## 参考

[链接 Android developer guide: services](http://developer.android.com/guide/components/services.html)
[链接 Android developer guide: bound services](http://developer.android.com/guide/components/bound-services.html)
[链接 Service 源码](http://grepcode.com/file/repository.grepcode.com/java/ext/com.google.android/android/5.1.1_r1/android/app/Service.java)
[链接 IntentService 源码](http://grepcode.com/file/repository.grepcode.com/java/ext/com.google.android/android/5.1.1_r1/android/app/IntentService.java)
[链接 Android_Message_Handler_消息处理机制总结笔记](http://segmentfault.com/a/1190000003862319)
[onServiceDisconnected problem not called](http://stackoverflow.com/a/5293050/2722270)
[『1. 图解Android Binder 和 Service』](http://www.cnblogs.com/samchen2009/p/3316001.html), [『2. Android开发：什么是IBinder』](http://blog.csdn.net/niu_gao/article/details/6453218)
