---
layout:     post
title:      "服务的最佳实践"
subtitle:   "后台执行的定时任务"
date:       2016-04-05
author:     "WunWun"
header-img: "img/android-best-practice-service.jpg"
tags:
    - Android
    - 学习笔记
    - Service
---

## 关于定时的基础知识

Android 中的定时任务一般有两种实现方式，一种是使用 Java API 里提供的 Timer 类，一种是使用 **Android 的 Alarm 机制**。这两种方式在多数情况下都能实现类似的效果，但 Timer有一个明显的短板，它并不太适用于那些需要长期在后台运行的定时任务。我们都知道，为了能让电池更加耐用，每种手机都会有自己的休眠策略， Android 手机就会在长时间不操作的情况下自动让 CPU 进入到睡眠状态，这就有可能导致 Timer 中的定时任务无法正常运行。而 Alarm 机制则不存在这种情况，它具有唤醒 CPU功能，即可以保证每次需要执行定时任务的时候 CPU 都能正常工作。

获取一个 AlarmManager 的实例

    AlarmManager manager = (AlarmManager) getSystemService(Context.ALARM_SERVICE);

接下来调用 AlarmManager 的 set()方法就可以设置一个定时任务了， 比如说想要设定一个任务在 10 秒钟后执行，就可以写成：

    long triggerAtTime = SystemClock.elapsedRealtime() + 10 * 1000;
    manager.set(AlarmManager.ELAPSED_REALTIME_WAKEUP, triggerAtTime, pendingIntent);

set()方法中需要传入三个参数。第一个参数是一个整型参数，用于指定 AlarmManager 的工作类型，有四种值可选，分别是 ELAPSED_REALTIME、 ELAPSED_REALTIME_WAKEUP、RTC 和 RTC_WAKEUP。

- ELAPSED_REALTIME 表示让定时任务的触发时间从系统开机开始算起，但不会唤醒 CPU。

- ELAPSED_REALTIME_WAKEUP 同样表示让定时任务的触发时间从系统开机开始算起，但会唤醒 CPU。 

- RTC 表示让定时任务的触发时间从 1970 年 1月 1 日 0 点开始算起，但不会唤醒 CPU。

- RTC_WAKEUP 同样表示让定时任务的触发时间从1970 年 1 月 1 日 0 点开始算起，但会唤醒 CPU。

- 使用 SystemClock.elapsedRealtime()方法可以获取到系统开机至今所经历时间的毫秒数。

- 使用 System.currentTimeMillis()方法可以获取到 1970 年 1 月 1 日 0 点至今所经历时间的毫秒数。

第二个参数就好理解多了，就是定时任务触发的时间，以毫秒为单位。如果第一个参数使用的是 ELAPSED_REALTIME 或 ELAPSED_REALTIME_WAKEUP，则这里传入开机至今的时间再加上延迟执行的时间。如果第一个参数使用的是 RTC 或RTC_WAKEUP，则这里传入 1970 年 1 月 1 日 0 点至今的时间再加上延迟执行的时间。

第三个参数是一个 PendingIntent。这里我们一般会调用 getBroadcast()方法来获取一个能够执行广播的 PendingIntent。这样当定时任务被触发的时候，广播接收器的 onReceive()方法就可以得到执行。

## 进入实战

下面我们就来创建一个可以长期在后台执行定时任务的服务。 创建一个 ServiceBestPractice 项目，然后新增一个 LongRunningService类，代码如下所示：

    public class LongRunningService extends Service {
        @Override
        public IBinder onBind(Intent intent) {
        return null;
        }
        @Override
        public int onStartCommand(Intent intent, int flags, int startId) {
            new Thread(new Runnable() {
                @Override
                public void run() {
                Log.d("LongRunningService", "executed at " + new Date().toString());
            }}).start();    

            AlarmManager manager = (AlarmManager) getSystemService(ALARM_SERVICE);
            int anHour = 60 * 60 * 1000; // 这是一小时的毫秒数
            long triggerAtTime = SystemClock.elapsedRealtime() + anHour;
            Intent i = new Intent(this, AlarmReceiver.class);
            PendingIntent pi = PendingIntent.getBroadcast(this, 0, i, 0);
            manager.set(AlarmManager.ELAPSED_REALTIME_WAKEUP, triggerAtTime, pi);
            return super.onStartCommand(intent, flags, startId);
        }
    }

下一步就是要新建一个 AlarmReceiver 类，并让它继承自 BroadcastReceiver，代码如下所示：

    public class AlarmReceiver extends BroadcastReceiver {
        @Override
        public void onReceive(Context context, Intent intent) {
            Intent i = new Intent(context, LongRunningService.class);
            context.startService(i);
        }
    }

可以看出，我们在这里构建出了一个 Intent 对象，然后去启动LongRunningService 这个服务。那么这里为什么要这样写呢？其实在不知不觉中， 这就已经将一个长期在后台定时运行的服务完成了。 因为一旦启动 LongRunningService，就会在onStartCommand()方法里设定一个定时任务，这样一小时后 AlarmReceiver 的 onReceive()方法就将得到执行，然后我们在这里再次启动 LongRunningService，这样就形成了一个永久的循环，保证 LongRunningService 可以每隔一小时就会启动一次，一个长期在后台定时运行的服务自然也就完成了。

接下来，我们需要在打开程序的时候启动一次 LongRunningService，之后 LongRunningService 就可以一直运行了。

    public class MainActivity extends Activity {
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            Intent intent = new Intent(this, LongRunningService.class);
            startService(intent);
        }
    }

## setExact()方法

另外需要注意的是，从 Android 4.4 版本开始， Alarm 任务的触发时间将会变得不准确，有可能会延迟一段时间后任务才能得到执行。这并不是个 bug，而是系统在耗电性方面进行的优化。系统会自动检测目前有多少 Alarm 任务存在，然后将触发时间将近的几个任务放在一起执行，这就可以大幅度地减少 CPU 被唤醒的次数，从而有效延长电池的使用时间。当然，如果你要求 Alarm 任务的执行时间必须准备无误， Android 仍然提供了解决方案。使用 AlarmManager 的 setExact()方法来替代 set()方法，就可以保证任务准时执行了。