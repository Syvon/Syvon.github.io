---
layout:     post
title:      "Android中的服务"
subtitle:   "服务是个什么鬼"
date:       2016-04-04
author:     "WunWun"
header-img: "img/android-.jpg"
tags:
    - Android
    - 学习笔记
---

> 这是《第一行代码》复习笔记的第九章.

## 服务是什么

服务（Service）是Android中实现程序后台运行的解决方案，它非常适合用于去**执行那些不需要和用户交互而且还要求长期执行的任务**。服务的运行不依赖于任何用户界面，即使程序被切换到后台，或者用户打开了另外一个应用程序，服务仍然能够保持正常运行。

服务并不是运行在一个独立的进程当中，而是**依赖于创建服务时所在的应用程序进程**。当某个应用程序进程被杀掉时，所有依赖于该进程的服务也会停止运行。

实际上服务并不会自动开启线程，所有的代码都是默认运行在主线程当中的。也就是说，我们需要**在服务的内部手动创建子线程**，并在这里执行具体的任务，否则就有可能出现主线程被阻塞住的情况。

## Android多线程编程

#### 线程的基本用法

定义一个线程只需要新建一个类继承自Thread，然后重写父类的run()方法，并在里面编写耗时逻辑即可。

    class MyThread extends Thread {
          
        @Override
        public void run() {
            // 处理具体的逻辑
        }
    }

那么该如何启动这个线程呢？其实也很简单，只需要new出MyThread的实例，然后调用它的start()方法，这样run()方法中的代码就会在子线程当中运行了。

    new MyThread().start();

当然，使用继承的方式耦合性有点高，更多的时候我们都会选择使用实现 Runnable 接口的方式来定义一个线程。

    class MyThread implements Runnable {
          
        @Override
        public void run() {
            // 处理具体的逻辑
        }
    }
启动线程的方法也需要进行相应的改变。

    MyThread myThread = new MyThread();
    new Thread(myThread).start();

可以看到，Thread的构造函数接收一个Runnable参数，而我们new出的MyThread正是一个实现了Runnable接口的对象，所以可以直接将它传入到Thread的构造函数里。接着调用Thread的start()方法，run()方法中的代码就会在子线程当中运行了。

当然，如果你不想专门再定义一个类去实现Runnable接口，也可以使用匿名类的方式，这种写法更为常见，如下所示：

     new Thread(new Runnable() {    

           @Override
           public void run() {
                // 处理具体的逻辑
           }
    }).start();

#### 在子线程中更新UI

和许多其他的GUI库一样，Android的UI库也是线程不安全的。也就是说，如果想要更新应用程序里的UI元素，就必须在主线程中进行，否则就会出现异常。

Android提供了一套异步消息处理机制，完美地解决了在子线程中进行UI操作的问题。

    public class MainActivity extends Activity implements OnClickListener {
        public static final int UPDATE_TEXT = 1;
        private TextView text;
        private Button changeText;  

        private Handler handler = new Handler() {
            public void handleMessage(Message msg) {
                switch (msg.what) {
                case UPDATE_TEXT:
                    // 在这里可以进行 UI 操作
                    text.setText("Nice to meet you");
                    break;
                default:
                    break;
                }
            }   

        };  

        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
            text = (TextView) findViewById(R.id.text);
            changeText = (Button) findViewById(R.id.change_text);
            changeText.setOnClickListener(this);
        }   

        @Override
        public void onClick(View v) {
            switch (v.getId()) {
            case R.id.change_text:
                new Thread(new Runnable() {
                    @Override
                    public void run() {
                        Message message = new Message();
                        message.what = UPDATE_TEXT;
                        handler.sendMessage(message); // 将 Message 对象发送出去
                    }
                }).start();
                break;
            default:
                break;
            }
        }   

    }

先是定义了一个整型常量UPDATE_TEXT，用于表示更新TextView这个动作。然后新增一个Handler对象，并重写父类的handleMessage方法，在这里对具体的Message进行处理。如果发现Message的what字段的值等于UPDATE_TEXT，就将TextView先是的内容改成Nice to meet you。

下面再来看一下ChangeText按钮的点击事件中的代码。可以看到，这次我们并没有在子线程里直接进行UI操作，而是创建了一个Message（android.os.Message）对象，并将它的what字段的值指定为UPDATE_TEXT，然后调用Handler的sendMessage()方法将这条Message发送出去。很快，Handler就会收到这条Message，并在handleMessage()方法中对它进行处理。注意此时handleMessage()方法中的代码就是在主线程当中运行的了，所以我们可以放心地在这里进行UI操作。接下来对Message携带的what字段值进行判断，如果等于UPDATE_TEXT，就将TextView显示的内容改成 Nice to meet you。

## 解析异步消息处理机制

Android中的异步消息处理主要由四个部分组成，Message，Handler，MessageQueue和Looper。

1.  Message
    Message 是在**线程之间传递的消息**，它可以在内部携带少量的信息，用于在不同线程之间交换数据。上一小节中我们使用到了 Message 的 what 字段，除此之外还可以使用 arg1 和 arg2 字段来携带一些整型数据，使用 obj 字段携带一个 Object 对象。

2.  Handler
    Handler 顾名思义也就是处理者的意思，它主要是**用于发送和处理消息**的。发送消息一般是使用 Handler 的 sendMessage() 方法，而发出的消息经过一系列地辗转处理后，最终会传递到 Handler 的 handleMessage() 方法中。

3.  MessageQueue
    MessageQueue 是消息队列的意思，它主要用于**存放所有通过 Handler发送的消息**。这部分消息会一直存在于消息队列中，等待被处理。每个线程中只会有一个 MessageQueue 对象。

4.  Looper
    Looper 是每个线程中的 **MessageQueue 的管家**，调用 Looper 的 loop() 方法后，就会进入到一个无限循环当中，然后每当发现 MessageQueue 中存在一条消息，就会将它取出，并传递到 Handler 的 handleMessage() 方法中。每个线程中也只会有一个 Looper 对象。

首先需要在主线程当中创建一个 Handler 对象，并重写 handleMessage() 方法。然后当子线程中需要进行 UI 操作时，就创建一个 Message 对象，并通过 handler 将这条消息发送出去。之后这条消息会被添加到 MessageQueue 的队列中等待被处理，而 Looper 则会一直尝试从 MessageQueue 中取出待处理消息，最后分发回 Handler 的 handleMessage() 方法中。由于 Handler 是在主线程中创建的，所以此时 handleMessage() 方法中的代码也会在主线程中运行，于是我们在这里就可以安心地进行 UI 操作了。整个异步消息处理机制的流程示意图如图 所示。

![Android-Message-handler](/img/in-post/the-first-line-of-code/android-message-handler.jpg)
<small class="img-hint">Android异步消息处理机制</small>

## 使用AsyncTask

为了更加方便我们在子线程中对UI进行操作，Android还提供了另外一些好用的工具，AsyncTask就是其中之一。借助AsyncTask，即使你对异步消息处理机制完全不了解，也可以十分简单地从子线程切换到主线程。当然，AsyncTask背后的实现原理也是基于异步消息处理机制的，只是Android帮我们做了很好的封装而已。

由于AsyncTask是一个抽象类，所以如果我们想使用它，就必须要创建一个子类去继承它。在继承时我们可以为 AsyncTask 类指定三个泛型参数。

- Params
    在执行 AsyncTask 时需要传入的参数，可用于在后台任务中使用。

- Progress
    后台任务执行时，如果需要在界面上**显示当前的进度**，则使用这里指定的泛型作为进度单位。

- Result
    当任务执行完毕后，如果需要**对结果进行返回**，则使用这里指定的泛型作为返回值类型。

一个最简单的自定义 AsyncTask 就可以写成如下方式：

    class DownloadTask extends AsyncTask<Void, Integer, Boolean> {
        ......
    }

AsyncTask 的第一个泛型参数指定为 Void，表示在执行 AsyncTask 的时候不需要传入参数给后台任务。第二个泛型参数指定为 Integer，表示使用整型数据来作为进度显示单位。第三个泛型参数指定为 Boolean，则表示使用布尔型数据来反馈执行结果。

当然，目前我们自定义的 DownloadTask 还是一个空任务，并不能进行任何实际的操作。我们还需要去重写 AsyncTask 中的几个方法才能完成对任务的定制。经常需要去重写的方法有以下四个。

- onPreExecute()
    这个方法会**在后台任务开始执行之前调用**，用于进行一些界面上的初始化操作，比如显示一个进度条对话框等。

- doInBackground(Params...)
    这个方法中的所有代码都会在子线程中运行，我们应该在这里去**处理所有的耗时任务**。任务一旦完成就可以通过 return 语句来将任务的执行结果返回，如果 AsyncTask 的第三个泛型参数指定的是 Void，就可以不返回任务执行结果。注意，在这个方法中是不可以进行 UI 操作的，如果需要更新 UI 元素，比如说反馈当前任务的执行进度，可以调用 publishProgress(Progress...)方法来完成。

- onProgressUpdate(Progress...)
    当在后台任务中调用 publishProgress(Progress...) 方法后，这个方法就会很快被调用，方法中携带的参数就是在后台任务中传递过来的。在这个方法中可以对 UI 进行操作，利用参数中的数值就可以对界面元素进行相应地更新。

- onPostExecute(Result)
    当后台任务执行完毕并通过 return 语句进行返回时，这个方法就很快会被调用。返回的数据会作为参数传递到此方法中，可以利用返回的数据来进行一些 UI 操作，比如说提醒任务执行的结果，以及关闭掉进度条对话框等。

因此，一个比较完整的自定义 AsyncTask 就可以写成如下方式：

    class DownloadTask extends AsyncTask<Void, Integer, Boolean>
    {   

        @Override
        protected void onPreExecute() {
            progressDialog.show(); // 显示进度对话框
        }   

        @Override
        protected Boolean doInBackground(Void... params) {
            try {
                while (true) {
                    int downloadPercent = doDownload(); // 这是一个虚构的方法
                    publishProgress(downloadPercent);
                    if (downloadPercent >= 100) {
                        break;
                    }
                }
            } catch (Exception e) {
                return false;
            }
            return true;
        }   

        @Override
        protected void onProgressUpdate(Integer... values) {
            // 在这里更新下载进度
            progressDialog.setMessage("Downloaded " + values[0] + "%");
        }   

        @Override
        protected void onPostExecute(Boolean result) {
            progressDialog.dismiss(); // 关闭进度对话框
            // 在这里提示下载结果
            if (result) {
                Toast.makeText(context, "Download succeeded", Toast.LENGTH_SHORT).show();
            } else {
                Toast.makeText(context, "Download failed", Toast.LENGTH_SHORT).show();
            }
        }
        
    }

简单来说，使用 AsyncTask 的诀窍就是，**在 donInBackground() 方法中去执行具体的耗时任务，在 onProgressUpdate() 方法中进行 UI 操作，在 onPostExecute() 方法中执行一些任务的收尾工作**。

 如果想要启动这个任务，只需编写以下代码即可：
    new DownloadTask().execute();

## 服务的基本用法

#### 定义一个服务

    public class MyService extends Service {    

        @Override
        public IBinder onBind(Intent intent) {
            return null;
        }   

        @Override
        public void onCreate() {
            super.onCreate();
        }   

        @Override
        public int onStartCommand(Intent intent, int flags, int startId) {
            return super.onStartCommand(intent, flags, startId);
        }   

        @Override
        public void onDestroy() {
            super.onDestroy();
        }   

    }
onBind() 方法是 Service 中唯一一个抽象方法，所以必须要在子类里实现。

onCreate() 方法会在服务创建的时候调用，onStartCommand() 方法会在每次服务启动的时候调用，onDestroy() 方法会在服务销毁的时候调用。

#### 启动和停止服务

public class MainActivity extends Activity implements OnClickListener {

    private Button startService;
    private Button stopService;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        startService = (Button) findViewById(R.id.start_service);
        stopService = (Button) findViewById(R.id.stop_service);
        startService.setOnClickListener(this);
        stopService.setOnClickListener(this);
    }

    @Override
    public void onClick(View v) {
        switch (v.getId()) {
        case R.id.start_service:
            Intent startIntent = new Intent(this, MyService.class);
            startService(startIntent); // 启动服务
            break;
        case R.id.stop_service:
            Intent stopIntent = new Intent(this, MyService.class);
            stopService(stopIntent); // 停止服务
            break;
        default:
            break;
        }
    }

}

在Start Service按钮的点击事件里，我们构建出了一个Intent对象，并调用startService()方法来启动MyService这个服务。在Stop Service按钮的点击事件里，我们同样构建出了一个Intent对象，并调用stopService()方法来停止MyService这个服务。**startService()和stopService()方法都是定义在Context类中的，所以我们在Activity里可以直接调用这两个方法**。注意，这里完全是由Activity来决定服务何时停止的，如果没有点击Stop Service按钮，服务就会一直处于运行状态。那服务有没有什么办法让自己停止下来呢？当然可以，只需要在MyService的任何一个位置调用**stopSelf()方法**就能让这个服务停止下来了。

onCreate()方法是在服务第一次创建的时候调用的，而onStartCommand()方法则再每次启动服务的时候都会调用，由于刚才我们是第一次点击Start Service按钮，服务此时还未创建过，所以两个方法都会执行，之后如果你再连续多点击几次Start Service按钮，你就会发现只有onStartCommand()方法可以得到执行了。

#### 活动和服务进行通信

我们学会了启动和停止服务的方法，不过，虽然服务是在活动里启动的，但在启动服务之后，活动与服务基本就没有什么关系了。确实如此，我们在活动里调用了 startService() 方法来启动 MyService 这个服务，然后服务会一直处于运行状态，但具体运行的是什么逻辑，活动就控制不了了。

那么有没有什么办法能让活动和服务的关系更紧密一些呢？例如在活动中指挥服务去干什么，服务就去干什么。当然可以，这就需要借助我们刚刚忽略的onBind()方法了。

下面我们希望在MyService里提供一个下载功能，然后在活动中可以决定何时开始下载，以及随时查看下载进度。实现这个功能的思路是创建一个专门的 Binder 对象来对下载功能进行管理。

    public class MyService extends Service {    

        private DownloadBinder mBinder = new DownloadBinder();  

        class DownloadBinder extends Binder {   

            public void startDownload() {
                Log.d("MyService", "startDownload executed");
            }   

            public int getProgress() {
                Log.d("MyService", "getProgress executed");
                return 0;
            }   

        }   

        @Override
        public IBinder onBind(Intent intent) {
            return mBinder;
        }
            ......
    }

这里我们新建了一个 DownloadBinder 类，并让它继承自 Binder，然后在它的内部提供了开始下载以及查看下载进度的方法。

接着，在 MyService 中创建了 DownloadBinder 的实例，然后在onBind() 方法里返回了这个实例，这样 MyService 中的工作就全部完成了。

    public class MainActivity extends Activity implements OnClickListener { 

        private Button bindService;
        private Button unbindService;   

        private MyService.DownloadBinder downloadBinder;    

        private ServiceConnection connection = new ServiceConnection() {    

            @Override
            public void onServiceDisconnected(ComponentName name) {
            }   

            @Override
            public void onServiceConnected(ComponentName name, IBinder service) {
                downloadBinder = (MyService.DownloadBinder) service;
                downloadBinder.startDownload();
                downloadBinder.getProgress();
            }
        };  

        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
            ......
            bindService = (Button) findViewById(R.id.bind_service);
            unbindService = (Button) findViewById(R.id.unbind_service);
            bindService.setOnClickListener(this);
            unbindService.setOnClickListener(this);
        }   

        @Override
        public void onClick(View v) {
            switch (v.getId()) {
            ......
            case R.id.bind_service:
                Intent bindIntent = new Intent(this, MyService.class);
                bindService(bindIntent, connection, BIND_AUTO_CREATE); // 绑定服务
                break;
            case R.id.unbind_service:
                unbindService(connection); // 解绑服务
                break;
            default:
                break;
            }
        }   

    }

这里我们首先创建了一个 ServiceConnection 的匿名类，在里面重写了onServiceConnected() 方法和 onServiceDisconnected() 方法，这两个方法分别会在活动与服务成功绑定以及解除绑定的时候调用。在 onServiceConnected() 方法中，我们又通过向下转型得到了 DownloadBinder 的实例，有了这个实例，活动和服务之间的关系就变得非常紧密了。现在我们可以在活动中根据具体的场景来调用 DownloadBinder 中的任何 public 方法，即实现了指挥服务干什么，服务就去干什么的功能。这里仍然只是做了个简单的测试，在 onServiceConnected() 方法中调用了 DownloadBinder 的 startDownload() 和 getProgress() 方法。

当然，现在活动和服务其实还没进行绑定呢，这个功能是在 Bind Service 按钮的点击事件里完成的。可以看到，这里我们仍然是构建出了一个 Intent 对象，然后调用 bindService() 方法将 MainActivity 和 MyService 进行绑定。bindService() 方法接收三个参数，第一个参数就是刚刚构建出的 Intent 对象，第二个参数是前面创建出的 ServiceConnection 的实例，第三个参数是一个标志位，这里传入 BIND_AUTO_CREATE 表示在活动和服务进行绑定后自动创建服务。这会使得 MyService 中的 onCreate() 方法得到执行，但 onStartCommand() 方法不会执行。

## 服务的生命周期

一旦在项目的任何位置调用了 Context 的 startService()方法，相应的服务就会启动起来，并回调 onStartCommand()方法。如果这个服务之前还没有创建过， onCreate()方法会先于onStartCommand()方法执行。服务启动了之后会一直保持运行状态，直到 stopService()或stopSelf()方法被调用。注意虽然每调用一次 startService()方法， onStartCommand()就会执行一次，但实际上**每个服务都只会存在一个实例**。所以不管你调用了多少次 startService()方法，只需调用一次 stopService()或 stopSelf()方法，服务就会停止下来了。

此外，还可以调用 Context 的 bindService()来获取一个服务的持久连接，这时就会回调服务中的 onBind()方法。类似地，如果这个服务之前还没有创建过， onCreate()方法会先于onBind()方法执行。之后，调用方可以获取到 onBind()方法里返回的 IBinder 对象的实例，这样就能自由地和服务进行通信了。只要调用方和服务之间的连接没有断开，服务就会一直保持运行状态。

当调用了 startService()方法后，又去调用 stopService()方法，这时服务中的 onDestroy()方法就会执行，表示服务已经销毁了。类似地，当调用了 bindService()方法后，又去调用unbindService()方法， onDestroy()方法也会执行，这两种情况都很好理解。但是需要注意，我们是完全有可能对一个服务既调用了 startService()方法，又调用了 bindService()方法的，这种情况下该如何才能让服务销毁掉呢？**根据 Android 系统的机制，一个服务只要被启动或者被绑定了之后，就会一直处于运行状态，必须要让以上两种条件同时不满足，服务才能被销毁。所以，这种情况下要同时调用 stopService()和 unbindService()方法， onDestroy()方法才会执行。**

## 服务的更多技巧

#### 使用前台服务

服务的系统优先级还是比较低的，当系统出现内存不足的情况时，就有可能会回收掉正在后台运行的服务。如果你希望服务可以一直保持运行状态，而不会由于系统内存不足的原因导致被回收，就可以考虑使用前台服务。**前台服务和普通服务最大的区别就在于，它会一直有一个正在运行的图标在系统的状态栏显示，下拉状态栏后可以看到更加详细的信息，非常类似于通知的效果**。当然有时候你也可能不仅仅是为了防止服务被回收掉才使用前台服务的，有些项目由于特殊的需求必须使用前台服务，比如说墨迹天气，它的服务在后台更新天气数据的同时，还会在系统状态栏一直显示当前的天气信息。

    public class MyService extends Service {    

        @Override
        public void onCreate() {
            super.onCreate();   

            Intent notificationIntent = new Intent(this, MainActivity.class);
            PendingIntent pendingIntent = PendingIntent.getActivity(this, 0,
                    notificationIntent, 0);
            Notification notification = new Notification.Builder(this)
                    .setAutoCancel(true)
                    .setContentTitle("This is a title")
                    .setContentText("This is content")
                    .setContentIntent(pendingIntent)
                    .setSmallIcon(R.mipmap.ic_launcher)
                    .setWhen(System.currentTimeMillis())
                    .build();
            startForeground(1, notification);   

            Log.d("MyService","onCreate");
        }
    }

startForeground() 这个方法接收两个参数，第一个参数是通知的 id，类似于 notify() 方法的第一个参数，第二个参数则是构建出的 Notification 对象。调用startForeground() 方法后就会让 MyService 变成一个前台服务，并在系统状态栏显示出来。

#### 使用IntentService

服务中的代码都是默认运行在主线程当中的，如果直接在服务里去处理一些耗时的逻辑，就很容易出现**ANR（Application Not Responding）**的情况。

这个时候就需要用到 Android 多线程编程的技术了，我们应该在服务的每个具体方法里开启一个子线程，然后在这里取处理那些耗时的逻辑，因此，一个比较标准的服务就可以写成如下形式：

    public class MyService extends Service {  
      
        @Override  
        public IBinder onBind(Intent intent) {  
            return null;  
        }  
      
        @Override  
        public int onStartCommand(Intent intent, int flags, int startId) {  
            new Thread(new Runnable() {    
                @Override    
                public void run() {    
                    // 处理具体的逻辑  
                }    
            }).start();  
            return super.onStartCommand(intent, flags, startId);  
        }  
    }

但是，这种服务一旦启动之后，就会一直处于运行状态，必须调用stopService() 或者stopSelf() 方法才能让服务停止下来。所以，如果想要实现让一个服务在执行完毕后自动停止的功能，就可以这样写：

    public class MyService extends Service {  
      
        @Override  
        public IBinder onBind(Intent intent) {  
            return null;  
        }  
      
        @Override  
        public int onStartCommand(Intent intent, int flags, int startId) {  
            new Thread(new Runnable() {    
                @Override    
                public void run() {    
                    // 处理具体的逻辑  
                        stopSelf();  
                   }    
            }).start();  
            return super.onStartCommand(intent, flags, startId);  
        }  
    }

虽然这种写法并不复杂，但是总会有一些程序员忘记开启线程，或者忘记调用 stopSelf()方法。为了可以简单地创建一个异步的、会自动停止的服务，Android 专门提供了一个 **IntentService** 类，这个类就很好地解决了前面所提到的两种尴尬，下面我们就来看一下它的用法。

    public class MyIntentService extends IntentService {  
      
        public MyIntentService() {  
            super("MyIntentService"); // 调用父类的有参构造函数  
        }  
      
        @Override  
        protected void onHandleIntent(Intent intent) {  
                 // 打印当前线程的 id  
                Log.d("MyIntentService", "Thread id is " + Thread.currentThread().getId());  
        }  
      
        @Override  
        public void onDestroy() {  
            super.onDestroy();  
            Log.d("MyIntentService", "onDestroy executed");  
        }  
      
    }

这里首先是要提供一个无参的构造函数，并且必须在其内部调用父类的有参构造函数。然后要**在子类中去实现onHandleIntent() 这个抽象方法，在这个方法中可以去处理一些具体的逻辑，而且不用担心 ANR 的问题，因为这个方法已经是在子线程中运行的了**。这里为了证实一下，我们在 onHandleIntent() 方法中打印了当前线程的 id。另外根据 IntentService 的特性，这个服务在运行结束后应该是会自动停止的，所以我们又重写了 onDestroy() 方法，在这里也打印了一行日志，以证实服务是不是停止掉了。