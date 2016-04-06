---
layout:     post
title:      "Android中的通知"
subtitle:   ""
date:       2016-04-02
author:     "WunWun"
header-img: "img/Android-Water.jpg"
tags:
    - Android
    - 学习笔记
---

> 这是《第一行代码》复习笔记的第八章.

## 通知的基本用法

> 当某个应用程序希望向用户发出一些提示信息，而该应用程序又不在前台运行时，就可以借助通知来实现。发出一条通知后，手机最上方的状态栏会显示一个通知的图标，下拉状态栏后就可以看到通知的详细内容。

首先需要一个**NotificationManager**来对通知进行管理，可以调用Context的getSystemService()方法获取到。

	NotificationManager manager = (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE); 

接下来要创建一个**Notification**对象，这个对象用于存储通知所需的各种信息。

	Notification notification=new Notification(R.drawable.ic_launcher, 
                    "this is ticker text",
                    System.currentTimeMillis()); 

创建好了Notification对象后，我们还需要对通知的布局进行设定，这里只需要调用Notification的**setLatestEventInfo()**方法就可以给通知设置一个标准布局。

	 notification.setLatestEventInfo(this, 
	 	"this is content title", 
        "this is content text", null); 

以上工作都做完了后，只需要调用NotificationManager的**notify()**方法就可以让通知显示出来了。

	manager.notify(1,notification); 

## PendingIntent,实现通知的点击效果

PendingIntent和Intent有些类似，它们都可以用于启动活动，启动服务以及发送广播等。不同的是Intent更倾向于立刻执行某个动作，而PendingIntent更倾向于**在某个合适的时机去执行某个动作**。所以也可以把PendingIntent简单的理解为延迟执行的Intent。

PendingIntent的用法比较简单，它主要提供了几个静态方法用于获取PendingIntent的实例，可以根据需求来选择是使用getActivity()方法，getBroadcast()方法，还是getService()方法。

    NotificationManager manager=(NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);  
    Notification notification=new Notification(R.drawable.ic_launcher,  
            "this is ticker text",System.currentTimeMillis());  
      
    ` Intent intent=new Intent(this,NotificationActivity.class);  `
    PendingIntent pi=PendingIntent.getActivity(this, 0,  
            intent, PendingIntent.FLAG_CANCEL_CURRENT);  
      
    notification.setLatestEventInfo(this, "this is content title",  
            "this is content text", pi);  
    manager.notify(1,notification);  

注意看，你会发现系统状态上面的图标还没有消失，这是为什么了？这是因为，如果我们没有在代码中对该通知进行取消，它会一直在系统的状态栏上显示。解决的方法也比较简单的，调用**NotificationManager的cancel()方法**就可以取消通知了。修改NotificationActivity中的代码.

        NotificationManager manager=(NotificationManager)   
                getSystemService(Context.NOTIFICATION_SERVICE);  
        manager.cancel(1);  //想要取消哪一条通知，就在cancel()方法中传入该通知的id就行了。 

## 通知的高级技巧

- sound属性

它可以在通知发出的时候播出一段音频，这样就能够更好地告知用户有通知到来。

	Uri soundUri=Uri.parse(new File("/system/media/audio/ringtongs/basic_tone.ogg"));
	notification.sound=soundUri;

- vibrate属性

vibrate这个属性。它是一个长整形的数组，由于设置手机静止和震动的时长，以毫秒为单位。下标0的值表示手机静止的时长，下标为1的值表示手机震动的时长，下标为2的值又表示手机静止的时长，以此类推。

	long[] vibrates={0,1000,1000,1000}; //手机在通知到来的时候立刻震动1秒，然后静止1秒，再震动1秒。
	notification.vibrate=vibrates;

控制手机震动还需要声明权限

	<uses-permission android.name="android.permission.VIBRATE"/>

- 在通知到来的时候控制手机LED灯

手机基本上都会前置一个LED灯，当有未接电话或未读短信，而此时手机又处于锁屏状态时LED灯就会不停地闪烁，提醒用户去查看。我们可以使用ledARGB,ledOnMS,ledOffMS以及flags这几个属性来实现这种效果。ledARGB用于控制LED灯的颜色，一般有红绿蓝三种颜色可选。ledOnMS用于指定LED灯亮起的时长，以毫秒为单位。ledOffMS由于指定LED灯暗去的时长，也是以毫秒为单位。flags可用于指定通知的一些行为，其中就包括显示LED灯这一选项。所以，等通知到来时，如果想要实现LED灯以绿色的灯一闪一闪的效果，就可以写成：

	notification.ledARGB=Color.GREEN;
	notification.ledOnMS=1000;
	notification.ledOffMS=1000;
	notification.flags=Notification.FLAG_SHOW_LIGHTS;

- 采用默认设置

当然，如果你不想进行那么多繁琐的设置，也可以直接使用通知的默认效果，它会根据当前手机的环境来决定播放什么铃声，以及如何震动，写法如下：
	notification.defaults=Notification.DEFAULT_ALL;