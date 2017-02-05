---
layout: post
title: Android Socket Programming
description: 
category: Android
tags: [android-http]
---


文章链接：[http://li2.me/2017/02/android-socket-programming.html](http://li2.me/2017/02/android-socket-programming.html)


### [Source Code Link](https://github.com/li2/Learning_Android_Open_Source/tree/master/Socket)

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


### Issues：

- Server can only receive the first message from client. commit #65796b8
- Java socket API: How to tell if a connection has been closed? We have to close stream and socket when disconnect. The right way to detect disconnect is to catch the IOException. commit #dee5bce


![Server](/assets/img/android/DemoServerSocket.png)

![Client1](/assets/img/android/DemoClientSocket1.png)

![book-cover](/assets/img/android/DemoClientSocket2.png)

------

weiyi.li, 2017-01-14, http://li2.me