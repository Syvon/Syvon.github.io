---
layout:     post
title:      "Android中的广播机制"
subtitle:   "《第一行代码》复习笔记4"
date:       2016-03-28
author:     "WunWun"
header-img: "img/android-group.jpg"
tags:
    - Android
    - 学习笔记
---

> 这是《第一行代码》复习笔记的第五章.


## 广播机制简介

Android中的广播主要分为两种类型：**标准广播**和**有序广播**。

- 标准广播（Normal broadcasts）是一种**完全异步执行**的广播，在广播发出之后，所有的广播接收器几乎都会在同一时刻接收到这条广播消息，因此他们之间没有任何先后顺序可言。这种广播的效率会比较高，但同时也意味着它是**无法被截断的**。

![Android-Normal-broadcasts](/img/in-post/the-first-line-of-code/normal-broadcast.jpg)
<small class="img-hint">标准广播</small>

- 有序广播（Ordered broadcasts）是一种同步执行的广播，在广播发出之后，同一时刻只会有一个广播接收器能够收到这条广播消息，当这个广播接收器中的逻辑执行完毕之后，广播才会继续传递。所以此时的广播接收器是**有先后顺序的**，优先级高的广播接收器就可以先收到广播消息，并且前面的广播接收器还可以截断正在传递的广播，这样后面的广播接收器就无法收到广播消息了。

![Android-Oredred-broadcasts](/img/in-post/the-first-line-of-code/normal-broadcast.jpg)
<small class="img-hint">有序广播</small>

## 接收系统广播

Android内置了很多系统级别的广播，我们可以在应用程序中通过监听这些广播来得到各种系统的状态信息。比如手机开机完成后会发出一条广播，电池的电量发生变化会发出一条广播，时间或时区发生改变也会发出一条广播等等。如果想要接收到这些广播，就需要使用广播接收器。

广播接收器可以自由地对自己感兴趣的广播进行注册，这样当有相应的广播发出时，广播接收器就能够收到该广播，并在内部处理相应的逻辑。注册广播的方式一般有两种，在代码中注册（**动态注册**）和在AndroidManifest.xml中注册（**静态注册**）。

- 动态注册监听网络变化

要创建一个广播接收器，只需要新建一个类，让它继承自 BroadcastReceiver， 并重写父类的**onReceive()**方法就行了。这样当有广播到来时，onReceive()方法就会得到执行， 具体的逻辑就可以在这个方法中处理。

    public class MainActivity extends Activity {    

        private IntentFilter intentFilter;
        private NetworkChangeReceiver networkChangeReceiver;    

        @Override   
        protected void onCreate(Bundle savedInstanceState) { 
            super.onCreate(savedInstanceState); 
            setContentView(R.layout.activity_main); intentFilter = new IntentFilter(); 
            intentFilter.addAction("android.net.conn.CONNECTIVITY_CHANGE"); 
            networkChangeReceiver = new NetworkChangeReceiver();    
            registerReceiver(networkChangeReceiver, intentFilter);      

        }   

        @Override   
        protected void onDestroy() {    
        super.onDestroy();  
        unregisterReceiver(networkChangeReceiver);  
        }       

        class NetworkChangeReceiver extends BroadcastReceiver { 
        @Override   
        public void onReceive(Context context, Intent intent) { 
            Toast.makeText(context, "network changes", Toast.LENGTH_SHORT).show();  
        }   
        }   
    }

可以看到，我们在MainActivity中定义了一个内部类 NetworkChangeReceiver，这个类是继承自BroadcastReceiver的，并重写了父类的onReceive()方法。这样每当网络状态发生变化时，onReceive()方法就会得到执行。

然后观察 onCreate()方法，首先我们创建了一个IntentFilter的实例，并给它添加了一个值为 android.net.conn.CONNECTIVITY_CHANGE的action，为什么要添加这个值呢？因为 当网络状态发生变化时，系统发出的正是一条值为 android.net.conn.CONNECTIVITY_ CHANGE 的广播，**也就是说我们的广播接收器想要监听什么广播，就在这里添加相应的 action就行了**。接下来创建了一个 NetworkChangeReceiver 的实例，然后调用 registerReceiver() 方法进行注册，将 NetworkChangeReceiver 的实例和 IntentFilter 的实例都传了进去，这样 NetworkChangeReceiver 就会收到所有值为 android.net.conn.CONNECTIVITY_CHANGE 的广 播，也就实现了监听网络变化的功能。

最后要记得，**动态注册的广播接收器一定都要取消注册才行**，这里我们是在 onDestroy()方法中通过调用 unregisterReceiver()方法来实现的。

如果要精确地告诉用户是有网络还是没网络，可以进一步优化。

    public class MainActivity extends Activity {    
    ……  
        class NetworkChangeReceiver extends BroadcastReceiver {     

            @Override   
            public void onReceive(Context context, Intent intent) { 
                ConnectivityManager connectionManager = (ConnectivityManager)   
                    getSystemService(Context.CONNECTIVITY_SERVICE); 
                NetworkInfo networkInfo = connectionManager.getActiveNetworkInfo();         

                if (networkInfo != null && networkInfo.isAvailable()) { 
                    Toast.makeText(context, "network is available", 
                        Toast.LENGTH_SHORT).show();         

                } else {            

                    Toast.makeText(context, "network is unavailable", Toast.LENGTH_SHORT).show();   
                }   
            }   
        }   
    }

在 onReceive()方法中，首先通过getSystemService()方法得到了ConnectivityManager的实例，这是一个系统服务类，专门用于管理网络连接的。然后调用它getActiveNetworkInfo()方法可以得到NetworkInfo的实例，接着调用NetworkInfo的isAvailable()方法，就可以判断出当前是否有网络了。

Android系统为了保证应用程序的安全性做了规定，如果程序需要访问一些系统的关键性信息，必须在配置文件中声明权限才可以，否则程序将 会直接崩溃，比如这里查询系统的网络状态就是需要声明权限的。打开AndroidManifest.xml文件，在里面加入如下权限就可以查询系统网络状态了：

    <manifest xmlns:android="http://schemas.android.com/apk/res/android" 
        package="com.example.broadcasttest"
        android:versionCode="1" 
        android:versionName="1.0" >
        <uses-sdk 
            android:minSdkVersion="14" 
            android:targetSdkVersion="19" />
        <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
    ……
    </manifest>

- 静态注册实现开机启动

动态注册的广播接收器可以自由地控制注册与注销，在灵活性方面有很大的优势，但是它也存在着一个缺点，即必须要在程序启动之后才能接收到广播，因为注册的逻辑是写在 onCreate()方法中的。如果要让程序在未启动的情况下就能接收到广播，就需要使用静态注册的方式了。

这里我们准备让程序接收一条开机广播，当收到这条广播时就可以在 onReceive()方法里 执行相应的逻辑，从而实现开机启动的功能。新建一个 BootCompleteReceiver 继承自 BroadcastReceiver，代码如下所示：

    public class BootCompleteReceiver extends BroadcastReceiver {
        @Override
        public void onReceive(Context context, Intent intent) { 
            Toast.makeText(context, "Boot Complete", Toast.LENGTH_LONG).show();
        }   

    }

在 AndroidManifest.xml 中将这个广播接收器的类名注册进去.

    <manifest xmlns:android="http://schemas.android.com/apk/res/android" 
        package="com.example.broadcasttest"    
        android:versionCode="1" 
        android:versionName="1.0" >     
        ……      
        <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />      
        <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />        
        <application 
            android:allowBackup="true" 
            android:icon="@drawable/ic_launcher" 
            android:label="@string/app_name" 
            android:theme="@style/AppTheme" >       
            ……          
            <receiver android:name=".BootCompleteReceiver" >            
                <intent-filter>         
                    <action android:name="android.intent.action.BOOT_COMPLETED" />          
                </intent-filter>            
            </receiver>     
        </application>  
    </manifest>

终于，<application>标签内出现了一个新的标签<receiver>，所有静态注册的广播接收器 都是在这里进行注册的。它的用法其实和<activity>标签非常相似，首先通过 android:name来指定具体注册哪一个广播接收器，然后在<intent-filter>标签里加入想要接收的广播就行了，由于 Android 系统启动完成后会发出一条值为 android.intent.action.BOOT_COMPLETED 的广播，因此我们在这里添加了相应的 action。

另外，监听系统开机广播也是需要声明权限的，我们使用<uses-permission>标签又加入了一条 android.permission.RECEIVE_BOOT_COMPLETED 权限。

## 发送自定义广播

- 发送标准广播

    Intent intent = new Intent("com.example.broadcasttest.MY_BROADCAST");
    sendBroadcast(intent);

首先构建出了一个Intent对象，并把要发送的广播的值传入，然后调用了Context的**sendBroadcast()**方法将广 播发送出去，这样所有监听com.example.broadcasttest.MY_BROADCAST这条广播的广播接收器就会收到消息。此时发出去的广播就是一条标准广播。

- 发送有序广播 

    Intent intent = new Intent("com.example.broadcasttest.MY_BROADCAST");
    sendOredresBroadcast(intent);

**sendOrderedBroadcast()**方法接收两个参数，第一个参数仍然是 Intent，第二个参数是一个与权限相关的字符串。

这个时候的广播接收器是有先后顺序的，而且前面的广播接收器还可以将广播截断，以阻止其继续传播。

那么该如何设定广播接收器的先后顺序呢？当然是在注册的时候进行设定的了，修改AndroidManifest.xml 中的代码，如下所示：

    <receiver android:name=".MyBroadcastReceiver">
        <intent-filter android:priority="100" >
            <action android:name="com.example.broadcasttest.MY_BROADCAST"/>
        </intent-filter>
    </receiver>

**abortBroadcast()**方法，可以将广播截断，后面的广播接收器将无法再接收到这条广播。

## 使用本地广播

Android引入了一套本地广播机制，使用这个机制发出的广播只能够在应用程序的内部进行传递，并且广播接收器也只能接收来自本应用 程序发出的广播。

    public class MainActivity extends Activity { 
        private IntentFilter intentFilter; 
        private LocalReceiver localReceiver;
        private LocalBroadcastManager localBroadcastManager;
        @Override
        protected void onCreate(Bundle savedInstanceState) { 
            super.onCreate(savedInstanceState); 
            setContentView(R.layout.activity_main);
            localBroadcastManager = LocalBroadcastManager.getInstance(this);            

            // 获取实例
            Button button = (Button) findViewById(R.id.button);
            button.setOnClickListener(new OnClickListener() {            
            @Override
            public void onClick(View v) {       
                Intent intent = new Intent("com.example.broadcasttest. LOCAL_BROADCAST");
                localBroadcastManager.sendBroadcast(intent); // 发送本地广播
            }           

            });
            intentFilter = new IntentFilter(); intentFilter.addAction("com.example.broadcasttest.LOCAL_BROADCAST"); localReceiver = new LocalReceiver(); localBroadcastManager.registerReceiver(localReceiver, intentFilter);// 注册本地广播监听器
            }   
        @Override   
        protected void onDestroy() {    
            super.onDestroy();  
            localBroadcastManager.unregisterReceiver(localReceiver);    
        }       

        class LocalReceiver extends BroadcastReceiver {     

            @Override
            public void onReceive(Context context, Intent intent) { 
                Toast.makeText(context, "received local broadcast",
                Toast.LENGTH_SHORT).show();
            }
        }
    }