---
layout: post
title: Android Socket Programming
description: 
category: Android
tags: [android-http]
---


文章链接：[http://li2.me/2017/02/android-socket-programming.html](http://li2.me/2017/02/android-socket-programming.html)


### [Source Code Link > GitHub](https://github.com/li2/Learning_Android_Open_Source/tree/master/Socket)

### ClientSocket

An Android App which implements the client-side socket.

`NetworkConnection` is a class designed to manage the socket connection:

```java
// @author: weiyi.li, http://li2.me
public class NetworkConnection {
    /*
     * A boolean flag to record connection status. When connection status changed,
     * et: open/close socket, network disconnect,
     * the flag will be updated and the callback will be invoked to notify UI.
     */
    private boolean mIsConnected;
    
    /** A callback for UI to update */
    public interface ConnectionListener {
        void onConnected(final boolean isConnected, final String disconnectReason);
        void onDataReceived(final String data);
        void onDataSent(final String data, final boolean succeeded);
    }

    /** Constructor */
    public NetworkConnection(Context context, ConnectionListener listener) {}

    /** Connect to server */
    public void connect(final String serverIp, final int serverPort) {}

    /** Disconnect to server */
    public void disconnect(final String disconnectReason) {}
    
    /** Send data to server */
    public void sendData(final String data) {}
    
    public boolean isConnected() {}
    
    /** Receive data from server through a thread when connected */
    private void startReceiveThread() {
        ...
            public void run() {
                try {
                    while (isConnected() && !mReceiveThreadInterrupted) {
                        mDataInputStream.readUTF();
                    }
                } catch (IOException e) {
                    // consider as disconnection to server
                }
            }
        ...
    }
}

```

Through this class, UI don't need to care about the details of connection (socket, input/output stream), just register the callback and use the public method to connect/disconnect/send data to server. commit #2f452ea

```java
// @author: weiyi.li, http://li2.me
public class MainActivity extends AppCompatActivity {
    private NetworkConnection mConnection;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        // Create connection
        mConnection = new NetworkConnection(this, mConnectionListener);
    }

    @Override
    protected void onDestroy() {
        // Destroy connection
        if (mConnection != null)
        {
            mConnection.disconnect("MainActivity.onDestroy");
        }
    }

    @OnClick(R.id.connectBtn)
    void connectToServer() {
        if (!mConnection.isConnected()) {
            mConnection.connect(ip, port);
        } else {
            mConnection.disconnect("User intent to disconnect");
        }
    }

    @OnClick(R.id.sendBtn)
    void sendToServer(String data) {
        if (mConnection.isConnected()) {
            mConnection.sendData(data);
        }
    }

    private NetworkConnection.ConnectionListener mConnectionListener =
        new NetworkConnection.ConnectionListener() {
            @Override
            public void onConnected(boolean isConnected, String disconnectReason) {
                Log.d(TAG, "onConnected " + isConnected);
            }

            @Override
            public void onDataReceived(String data) {
                Log.d(TAG, "onDataReceived " + data);
            }

            @Override
            public void onDataSent(String data, boolean succeeded) {}
        };
}
```


### ServerSocket

A Java Project which implements the server-side socket. When server is running, it waits new client connection, receive data from client, send response to this client, and broadcast the data to other clients. To run the server:

```sh
$ ./runServer.sh
```

To support multiple clients socket connection, create a thread pool (through ExecutorService), commit #aac37a0. 


### References

- [The Java™ Tutorials > Custom Networking > Lesson: All About Sockets](https://docs.oracle.com/javase/tutorial/networking/sockets/)

    > Socket classes are used to represent the connection between a client program and a server program. The java.net package provides two classes--Socket and ServerSocket--that implement the client side of the connection and the server side of the connection, respectively.
    
- [IBM Developerworks > Linux socket 编程，第一部分](http://www.ibm.com/developerworks/cn/education/linux/l-sock/l-sock.html)

    > 这个入门级的教程展示如何开始使用套接字编程。重点集中于 C 和 Python，本教程指导您完成一个回显（echo）服务器和客户机（它们通过 TCP/IP 来连接）的创建过程。它描述了基础的网络、层和协议概念，同时提供了丰富的示例源代码。
    
- [IBM Developerworks > 使用 Android 实现联网](https://www.ibm.com/developerworks/cn/opensource/os-android-networking/)

    > 「清单 3. Daytime 客户机」所示的代码片段展示了另一种与远程服务器交互的方式。使用了较低级的 Socket 类在端口 13 打开与远程服务器的基于流的 socket 连接。Daytime Server 接受传入的 socket 连接并以文本的形式将日期和时间发送给调用 socket。一旦发送完数据，服务器将关闭 socket。

- [IBM Developerworks > 深入分析 Java I/O 的工作机制](https://www.ibm.com/developerworks/cn/java/j-lo-javaio/)

    > 在上面的这段程序中，是将 Server 端的监听连接请求的事件和处理请求的事件放在一个线程中，但是在实际应用中，我们通常会把它们放在两个线程中，一个线程专门负责监听客户端的连接请求，而且是阻塞方式执行的；另外一个线程专门来处理请求，这个专门处理请求的线程才会真正采用 NIO 的方式。
    
- Server can only receive the first message from client. commit #65796b8

- [StackOverflow > Multithreading Socket communication Client/Server](http://stackoverflow.com/questions/12588476/multithreading-socket-communication-client-server)

    > Create a thread pool through {@link ExecutorService} to support multiple client socket connection.
    
- [StackOverflow > Java socket API: How to tell if a connection has been closed?](http://stackoverflow.com/a/10241044/2722270)

    > We have to close stream and socket when disconnect. The right way to detect disconnect is to catch the IOException. commit #dee5bce
    
- [StackOverflow > How to terminate a thread blocking on socket IO operation instantly?](http://stackoverflow.com/a/4426050/2722270)

    > Socket should be closed firstly, to make any threads blocked in socket
stream operations to be unblocked, otherwise, ReceiveThread.join() will block the current thread, and the
disconnect operation will also be blocked.

- [StackOverflow > Stop/Interrupt threads blocked on waiting input from socket](http://stackoverflow.com/a/1024501/2722270)


### Demo Figures

![Server](/assets/img/android/DemoServerSocket.png)

![Client1](/assets/img/android/DemoClientSocket1.png)

![book-cover](/assets/img/android/DemoClientSocket2.png)

------

weiyi.li, 2017-01-14, http://li2.me