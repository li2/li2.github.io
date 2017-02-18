---
layout: post
title: 「Efficient Android Threading 笔记」- C1 Android Components and the Need for Multiprocessing
description: 
category: Android
tags: [android-thread]
---

by weiyi.li @2017-02-18 17:05:52 @[li2.me](li2.me)

## Chapter 1 Android Components and the Need for Multiprocessing 

Explains how the structure of the Android runtime and how the various components of an Android application affect the use of threads and multiprocessing. 

### Android-specific asynchronous mechanisms

Application Framework defines a set of Android-specific asynchronous mechanisms that applications can utilize to simplify the thread management: `HandlerThread`, `AsyncTask`, `IntentService`, `AsyncQueryHandler`, and `Loaders`. All these mechanisms will be described in this book. 
Android Framework 定义了一套其特有的异步处理机制以简化线程管理。

### Application and Four components

Application Architecture: The cornerstones of an application are the `Application` object and the Android components: `Activity`, `Service`, `BroadcastReceiver`, and `ContentProvider`. 

应用架构的基石是 `Application` 对象和 Android 四大组件。

These entities have different responsibilities and lifecycles, but they all represent application **entry points**, where the application can be started. Once a component is started, it can trigger another component, and so on, throughout the application’s lifecycle. A component is trigged to start with an Intent, either within the application or between applications.  

四大组件有不同的职责和生命周期，但都是 App 的入口（需要在 manifest 清单中声明）。在应用内或者应用间，通过 Intent 启动一个组件。

The `Intent` specifies **actions** for the receiver to act upon — for instance, sending an email or taking a photograph—and can also provide data from the sender to the receiver. An Intent can be explicit or implicit.

Intent 指定了接收者要做的事情，而且还可携带数据。**所以 Intent 非常重要**。


Components and their lifecycles are Android-specific terminologies, and they are not directly matched by the underlying Java objects. A Java object can outlive its component, and the runtime can contain multiple Java objects related to the same live component. This is a source of confusion, and as we will see in Chapter 6, it poses **a risk for memory leaks**. 

组件及其生命周期是 Android 特有的术语，不同于底层 Java 对象。Java 对象可以比其组件存活的更久，带来内存泄漏的风险。


An Activity is a screen—almost always taking up the device’s full screen—shown to the user. It displays information, handles user input, and so on. It contains the UI components.

The state of an application’s topmost Activity has an impact on the application’s system priority—also known as process rank—which in turn affects both the chances of terminating an application (“Application termination” on page 7) and the scheduled execution time of the application threads (Chapter 3). 

Activity 是一个窗口，包含 UI 组件、展示信息、处理用户输入。


A Service can execute invisibly in the background without direct user interaction. It is typically used to offload execution from other components, when the operations can outlive their lifetime.  

Service 运行在后台，可以长期运行，不必直接和用户交互。

ContentProvider is most commonly used in collaboration with SQLite databases, which are always private to an application.  With the help of a ContentProvider, an application can publish that data to applications that execute in remote processes. 

ContentProvider 经常和 SQLite databases 捆绑使用，以开放其数据库给 Remote App。

BroadcastReceiver listens for intents sent from within the application, remote applications, or the platform. It filters incoming intents to determine which ones are sent to the BroadcastReceiver.  

BroadcastReceiver 只能用来监听 intents（App 内的，Remote App的，platform 的）。

### Process

An application in Android corresponds to a unique user in Linux and cannot access other applications’ resources. By default, applications and processes have a one-to-one relationship. What Android adds to each process is a **runtime execution environment**, either Dalvik or ART (Android Runtime API level 19).

![](/assets/img/android/efficient-android-threading/Figure-1-3-Applications-execute-in-different-processes-and-VMs.png)

一个应用对应到一个进程，Android 为每个进程添加一个 runtime execution environment。

Any component can be the entry point for the application, and once the first component is triggered to start, a Linux process is started — unless it is already running — leading to the following startup sequence: 


1. Start Linux process. 
2. Create runtime.
3. Create Application instance.
4. Create the entry point component for the application. 

当 App 还未运行时，启动其任意一个组件，会导致一系列的启动过程。



Setting up a new Linux process and the runtime is not an instantaneous operation.  the system tries to shorten the startup time for Android applications by starting a special process called Zygote on system boot. Zygote has the entire set of core libraries preloaded. New application processes are forked from the Zygote process without copying the core libraries, which are shared across all applications. 

创建一个新的 Linux 进程不是瞬间就能完成的，系统为了降低 App 启动时间，在系统启动过程中创建了一个特殊的进程  Zygote，App 的进程 fork 自 Zygote.



A process is created at the start of the application and finishes when the system wants to free up resources.  When the system is low on resources, it’s up to the runtime to decide which process should be killed. To make this decision, the system imposes a ranking on each process depending on the application’s visibility and the components that are currently executing.  

进程在 App 启动时创建，但即使所有的组件已经被销毁，进程仍然可能被保存在系统中，以便用户下次打开应用时快速启动。当系统内存资源紧张时，runtime 会按照 **process rank**（由 App visibility 和当前正在运行的组件决定） 决定销毁哪个进程以回收资源。

process rank：

1. Foreground 前台有可见组件；或者有前台 Service；或者有运行的 BroadcastReceiver。
2. Visible 有可见组件，但部分被遮挡。
3. Service  有后台 Service（未 bound 到可见组件）。
4. Background 没有可见组件。
5. Empty 没有任何运行的组件。

![](/assets/img/android/efficient-android-threading/process-ranks.png)

上图展示了两个进程的生命周期。Broadcast（BR）启动了进程 P1，P1 创建了 BroadcastReceiver 和 Application 实例，然后 Activiy 被启动，其运行过程中启动了进程 P2 的 Service。用户完成操作后就离开了 P1 Activiy，再然后 P2 Service 被其它进程或 runtime 停止。

### Structuring Applications for Performance 

operations can be partitioned and executed concurrently & asynchronously to optimize application performance.

操作可以被分割、并发、异步执行以最优化应用性能。


One approach is to split application execution into several processes, because those can run concurrently.  However, every process allocates memory for its own substantial resources. Furthermore, starting and communicating between processes is slow, and not an efficient way of achieving asynchronous execution. To achieve higher throughput and better performance, an application should utilize multiple threads within each process. 

一种途径是把 App 分割成多个进程以实现并发。但会耗费更多的内存，而且进程间通信较慢，也不是实现异步的高效方式。优先考虑的途径是在同一个进程内实现多线程。


Responsiveness is the way the user perceives the application during interaction: that the UI responds quickly to button clicks, smooth animations, etc. 

To make the application responsive, all Android components and system callbacks — unless denoted otherwise — run on the UI thread (a.k.a main thread), and should use background threads (a.k.a worker thread) when executing longer tasks. 

为了构建快速响应的 App，所有 Android 组件和系统回调运行在 UI thread（除非另有说明）， 长时间任务在后台线程中执行。


Long-running tasks typically include:

- Network communication
- Reading or writing to a file
- Creating, deleting, and updating elements in databases
- **Reading or writing to SharedPreferences（！！！）**
- Image processing
- Text parsing 

------

by weiyi.li @2017-02-18 17:05:52 @[li2.me](li2.me)