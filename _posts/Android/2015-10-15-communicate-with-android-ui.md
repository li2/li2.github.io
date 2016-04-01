---
layout: post
title: Android Message Handler 消息处理机制总结笔记
category: Android
tags: [android-handler]
---

官方文档 [Communicating with the UI Thread](http://developer.android.com/training/multiple-threads/communicate-ui.html)

## 一次性线程和无限循环线程

普通线程是一次性的，执行结束后也就退出了（这种说法可能不严谨，但为了下文描述方便）。
但某些情况下需要无限循环、不退出的线程。比如处理用户交互的线程，它等待并执行用户的点击、滑动等等操作事件，也执行由系统触发的广播等事件，称之为主线程，也叫UI线程。

关于这种无限循环线程需要说的是：

- 每个事件及其所包含的信息被封装为一个`Message`（下文中提到的“消息”和“事件”是一回事儿）；
- 线程不能同时处理所有事件，所以需要有个收件箱`MessageQueue`（下文中提到的线程，如果没有特别说明是一次性线程，那么都指looper thread）；
- 之所以不同于一次性线程，而能无限循环，是`Looper`的功劳，所以这种线程也被称为message looper thread；
- 事件由`Handler`发送和处理。


## 创建一个looper thread

Android提供的类`HandlerThread`，是一个looper thread类。通过源码理解它是怎么办到的：
[HandlerThread源码链接](http://grepcode.com/file/repository.grepcode.com/java/ext/com.google.android/android/5.1.1_r1/android/os/HandlerThread.java)

```java
23 public class HandlerThread extends Thread {
51     @Override
52     public void run() {
            // 初始化当前线程为looper thread，prepare()方法保证为每个线程只新建一个Looper object；
            // 在新建Looper object时，新建了一个MessageQueue object；
            // 所以，HandlerThread对象、Looper对象、MessageQueue对象，三者之间建立了彼此惟一的关联。
54         Looper.prepare();
55         synchronized (this) {
                // 该静态方法返回与当前线程关联的Looper object，如果当前线程未关联Looper则返回空；
                // 稍后讲到的Handler也是通过该方法建立和当前线程的惟一关联；
                // 至于为什么是当前线程，好像是和ThreadLocal<Looper>这个静态变量有关，不甚明了，存疑 TODO.
56             mLooper = Looper.myLooper();
57             notifyAll();
58         }
            // 你可以覆写这个函数，以在循环开始前做些必要的工作，比如实例化一个Handler，稍后讲它。
60         onLooperPrepared();
            // 调用loop()方法后，Looper对象就开始循环地从MessageQueue对象中取消息、处理消息、没消息时等待消息；
            // 这就使得它成为了一个looper thread；
            // 而如何处理消息，涉及Handler，稍后讲它。
61         Looper.loop();
63     }
```
所以，创建一个HandlerThread实例，就创建了一个looper thread：

<!-- more -->

```java
// 在合适的地方新建并启动线程，比如在Activity.onCreate()方法。
HandlerThread mHandlerThread = new HandlerThread("Gallery Downloading Thread");
mHandlerThread.start();

// 必须调用getLooper()，而且必须在start()之后调用，该方法获取与线程关联的Looper对象，以确保线程就绪；
// 因为该方法会被阻塞直至looper对象初始化好了。
mHandlerThread.getLooper();

// 在合适的地方调用quit()方法，比如Activity.onDestroy()方法，以退出线程的looper循环，不再处理消息队列中的任何消息；
// 此处存疑 TODO：线程退出了吗？（好像并没有，该如何处理？）
mHandlerThread.quit();
```
如此，一个looper thread就在运转了，但它发现没有消息，于是它等待。问题是，
该如何传递消息给它；它拿到消息后又是怎么处理的呢？答案在Handler。


## Handler负责消息的发送和处理

[Handler源码链接](http://grepcode.com/file/repository.grepcode.com/java/ext/com.google.android/android/5.1.1_r1/android/os/Handler.java)

关于Handler需要说的是：

- Handler使你能够发送并处理`Message`和`Runnable`对象；
- 当你创建了一个Handler实例，它就被绑定给创建它的线程，只能绑定给这1个线程，这也就意味着只关联了1个`MessageQueue`，此后，就可以发送messages和runnables给所关联的message queue，而当这些消息出队时，就在handler绑定的线程中得到执行；
- Handler主要有2种用途：(1) 安排messages和runnables在将来的某个时间点执行；(2) 让另一个线程做些事情。
以上内容截取自源码文件的注释（翻译后）。

### 使用默认构造器创建一个Handler

```java
Handler mHandler = new Handler();
```
通过源码来理解为什么handler被绑定到了创建它的线程:

```java
188    public Handler(Callback callback, boolean async) {
            // 通过该静态方法获得与当前线程绑定的Looper，而Looper是和当前线程惟一关联的（参看上文描述）；
            // 这样就建立了handler和当前线程及其MessageQueue的惟一关联；
            // 当然也可以通过另一个方法关联到指定的Looper，此处暂且不表 TODO.
198        mLooper = Looper.myLooper();
            // 如果当前线程没有looper，就没有queue，就没法接收消息，所以抛出异常。
199        if (mLooper == null) {
200            throw new RuntimeException(
201                "Can't create handler inside thread that has not called Looper.prepare()");
202        }
203        mQueue = mLooper.mQueue;
204        mCallback = callback;
206    }

// 默认的构造器就调用上面的方法，绑定handler到当前线程，也就是创建它的线程。
113    public Handler() {
114        this(null, false);
115    }
```
现在有了handler，又该如何发送消息呢？在发送之前，我们先新建一个消息。

### 使用默认构造器创建一个Message

```java
Message msg = new Message();
msg.what // 自定义的消息识别码，整形，以便接收者识别消息；
msg.arg1 // 如果两个int数据就可以满足你的要求，就用arg1和arg2；
msg.arg2
msg.obj // 如果复杂，就传递Object对象，或者Bundle数据；
msg.setData(Bundle data)
```

### 使用Handler发送消息
Handler可以发送处理`Message`和`Runnable`两种对象，提供了若干方法：

```java
boolean post(Runnable r);
boolean postAtTime(Runnable r, Object token, long uptimeMillis);
boolean postDelayed(Runnable r, long delayMillis);

boolean sendEmptyMessage(int what);
boolean sendMessage(Message msg);
boolean sendMessageAtTime(Message msg, long uptimeMillis);
boolean sendMessageDelayed(Message msg, long delayMillis);
```
这些方法最终都调用了`sendMessageAtTime()`，然后把消息放入队列。Runnable对象在内部被转成了Message对象的`callback`字段，稍后讲它。

### Message绑定给了发送它的Handler
需要特别关注message的成员变量`Handler target`，消息入队时，该字段被设置为发送它的handler，这就确保了message对象和handler对象的惟一关联。这样在message出队时，就知道该交给哪个handler处理；除此之外它还有别的用处，稍后讲它。

### 高效发送消息的做法

上文发送消息的做法，每次都需要新建一个message实例。如果频繁的话，Android推荐这样做：

```java
mHandler
    .obtainMessage(int what, int arg1, int arg2, Object obj)
    .sendToTarget();
```
`Handler.obtainMessage(...)` 方法从公共循环池里获取消息（如果没有的话，它会创建新的实例），并传入消息的各个字段。这样可以避免创建新的Message实例，提高效率。
`Message.sendToTarget()`方法只执行了一条语句`target.sendMessage(this)`，这就是上文提到的`target`字段的别的用处。这样就把消息放入了队列。

### 使用Handler处理消息
这是消息循环的核心源码，一个死循环：

```java
// Looper.loop()
109    public static void loop() {
110        final Looper me = myLooper();
114        final MessageQueue queue = me.mQueue;
115
121        for (;;) {
122            Message msg = queue.next(); // might block 没有消息则阻塞。
123            if (msg == null) {
                    // 收到空消息就退出。
124                // No message indicates that the message queue is quitting.
125                return;
126            }
                // 这就是前面提到的target字段的作用：知道把出队的消息交给谁。
135            msg.target.dispatchMessage(msg);
153        }
154    }
```
下面就开始dispatchMessage：

```java
// Handler.dispatchMessage()
93     public void dispatchMessage(Message msg) {
            // 上文提到的，发送消息时，Runnable对象在内部被转成了Message对象的`callback`字段；
            // 若callback不为空，那么消息就是一个runnable对象，那么就执行它；
94         if (msg.callback != null) {
95             handleCallback(msg);
96         } else {
                // 如果不是runnable，那么就是Message对象了：
                // 上文只讲了使用默认构造器创建handler，也可以Handler(Callback callback);
                // 这样就为每个handler设置了各自的callback，优先执行它；
97             if (mCallback != null) {
98                 if (mCallback.handleMessage(msg)) {
99                     return;
100                }
101            }
                // 如果handler对象没有自己的callback，那么就执行这个方法；
                // 这是一个空方法，当通过默认构造器新建一个handler时需要覆写它。
102            handleMessage(msg);
103        }
104    }
```
理论知识已经具备了，那么接下来，在1个典型的应用场景中使用它们解决我们的问题。

## 一个demo使用后台线程完成下载任务

这段代码截取自《Android权威编程指南》第26, 27章，作者构建了一个这样的App：下载Flicker上最新的100张缩略图，并填充到GridView内。首先通过AsyncTask线程获取包含缩略图url信息的XM文件；然后下载那些url指向的缩略图。这里需要思考的是，何时下载这些缩略图，是一次下载完？还是下载一部分？由谁触发下载？下载后怎样填充到GridView内？
解决了这些问题，就基本掌握了创建后台线程，并和主线程通信的方法。

```java
public class PhotoGalleryFragment extends Fragment {
    ThumbnailDownloader<ImageView> mThumbnailDownloader;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        mThumbnailDownloader = new ThumbnailDownloader<ImageView>(new Handler());
        mThumbnailDownloader.setOnThumbnailDownloadListener(new OnThumbnailDownloadListener<ImageView>() {
            public void onThumbnailDownloaded(ImageView imageView, Bitmap bitmap) {
                imageView.setImageBitmap(bitmap);
            }
        });
        mThumbnailDownloader.start();
        mThumbnailDownloader.getLooper();
    }

    @Override
    public void onDestroy() {
        mThumbnailDownloader.quit();
    }

    private class GalleryItemAdapter extends ArrayAdapter<GalleryItem> {
        @Override
        public View getView(int position, View convertView, ViewGroup parent) {
            ImageView imageView;
            String url;
            // 对于需要显示在GridView视图中的成百上千张图片，我们不可能一次下载完然后显示；
            // 只能在需要显示时下载，而GridView的adapter知道什么时候显示哪些视图；
            // 所以我们在此处安排后台线程的下载任务。
            mThumbnailDownloader.queueThumbnail(imageView, url);
            return convertView;
        }
    }
}


public class ThumbnailDownloader<Token> extends HandlerThread {
    private static final String TAG = "ThumbnailDownloader";
    private static final int MESSAGE_DOWNLOAD = 0;

    Handler mHandler;
    Map<Token, String> requestMap = Collections.synchronizedMap(new HashMap<Token, String>());

    Handler mResponseHandler;
    OnThumbnailDownloadListener<Token> mOnThumbnailDownloadListener;

    public interface OnThumbnailDownloadListener<Token> {
        void onThumbnailDownloaded(Token token, Bitmap thumbnail);
    }

    public void setOnThumbnailDownloadListener(OnThumbnailDownloadListener<Token> l) {
        mOnThumbnailDownloadListener = l;
    }

    public ThumbnailDownloader(Handler responseHandler) {
        super(TAG);
        // 后台线程能在主线程上完成任务的一种方式是，让主线程将其自身的Handler传给后台线程；
        // mResponseHandler始终和主线程保持关联，由它发送的消息都将在主线程中得到处理。
        mResponseHandler = responseHandler;
        // 我们也可以传递主线程的context，通过下述方式获取主线程的handler:
        // mResponseHandler = new Handler(mContext.getMainLooper());
    }

    public void queueThumbnail(Token token, String url) {
        // requestMap是一个同步HashMap。 使用Token作为key，可存储或获取与特定Token关联的URL.
        requestMap.put(token, url);
        // mHandler是和后台线程关联的，我们开放这个方法给主线程，主线程调用这个方法来安排后台线程的任务。
        // 我们把下载信息封装成message后放入后台线程的收件箱。
        mHandler.obtainMessage(MESSAGE_DOWNLOAD, token).sendToTarget();
    }

    @Override
    protected void onLooperPrepared() {
        // onLooperPrepared()方法发生在Looper.loop()之前，此时消息还没有开始循环，
        // 所以是我们实现mHandler的好地方，在此处下载。
        mHandler = new Handler() {
            @Override
            public void handleMessage(Message msg) {
                if (msg.what == MESSAGE_DOWNLOAD) {
                    Token token = (Token)msg.obj;
                    handleRequest(token);
                }
            }
        };
    }

    private void handleRequest(final Token token) {
        // 下载，并根据获取的数据构建Bitmap对象；
        final String url = requestMap.get(token);
        final Bitmap bitmap;
        // 下载完成后，我们在后台线程使用与主线程关联的handler，安排要在主线程上完成的任务。
        // 除了post，我们也可以sendMessage给主线程，那么主线程的handler需要覆写自己的handleMessage()方法。
        mResponseHandler.post(new Runnable() {
            @Override
            public void run() {
                if (mOnThumbnailDownloadListener != null) {
                    mOnThumbnailDownloadListener.onThumbnailDownloaded(token, bitmap);
                }
            }
        });
    }
}
```


## 后记和参考

强烈建议阅读《Android权威编程指南》第26, 27这两章代码。
[本书的官方主页链接 可以从这里获取本书的源码](https://www.bignerdranch.com/we-write/android-programming/)
但我在学习这两个章节的时候，有些概念和类的描述理解很吃力，比如“消息循环由一个线程和一个looper组成。 Looper对象管理着线程的消息队列”，又比如“一个Handler仅与一个Looper相关联，一个Message 也仅与一个目标Handler”。
因为作者着重讲解app的实现思路，对涉及到的类只给出了结论，没有说明为什么。所以初学时很费解，实际编程时也是只知其然不知其所以然。后来读了些博文（下面给出了链接），又阅读了源码，总算厘清了这些类，而这篇文章就是我理解后的一个产物。

下面是3个作者的4篇博文，总结的都很棒，侧重于源码分析。尤其是第2篇，作者最后画了一张图，通过传送带来解释涉及到的类和概念：“在现实生活的生产生活中，存在着各种各样的传送带，传送带上面洒满了各种货物，传送带在发动机滚轮的带动下一直在向前滚动，不断有新的货物放置在传送带的一端，货物在传送带的带动下送到另一端进行收集处理。”

- [Android中Handler的使用 - iSpring的CSDN博客](http://blog.csdn.net/iispring/article/details/47115879)
- [深入源码解析Android中的Handler,Message,MessageQueue,Looper - iSpring的CSDN博客](http://blog.csdn.net/iispring/article/details/47180325)
- [android的消息处理机制（图+源码分析）——Looper,Handler,Message - CodingMyWorld的cnblog](http://www.cnblogs.com/codingmyworld/archive/2011/09/14/2174255.html)
- [Android异步消息处理机制完全解析，带你从源码的角度彻底理解 - 郭霖的CSDN专栏](http://blog.csdn.net/guolin_blog/article/details/9991569)

下面就是源码链接了，对于理解android的消息处理机制非常有帮助。

- [Handler源码链接](http://grepcode.com/file/repository.grepcode.com/java/ext/com.google.android/android/5.1.1_r1/android/os/Handler.java)
- [HandlerThread源码链接](http://grepcode.com/file/repository.grepcode.com/java/ext/com.google.android/android/5.1.1_r1/android/os/HandlerThread.java)
- [Message源码链接](http://grepcode.com/file/repository.grepcode.com/java/ext/com.google.android/android/5.1.1_r1/android/os/Message.java)
- [Looper源码链接](http://grepcode.com/file/repository.grepcode.com/java/ext/com.google.android/android/5.1.1_r1/android/os/Looper.java)
