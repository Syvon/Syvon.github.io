---
layout:     post
title:      "接收和发送短信"
subtitle:   ""
date:       2016-04-03
author:     "WunWun"
header-img: "img/android-evolution.jpg"
tags:
    - Android
    - 学习笔记
---

接收短信主要是利用我们前面学过的广播机制。当手机接收到一条短信的时候，系统会发出一条值为andorid.provider.Telephony.SMS_RECEIVED的广播，这条广播里携带着与短信相关的所有数据。每个应用程序都可以在广播接收器里对它进行监听，收到广播时在从中解析出短信的内容即可。

## 接收短信

首先创建一个广播接收器来接受系统发出的短信广播。在MainActivity中新建MessageReceiver内部类继承自BroadcastReceiver，并在onReceive()方法中编写获取短信数据的逻辑。

    class MessageReceiver extends BroadcastReceiver{  
     //定义广播接收器  
      
        @Override  
        public void onReceive(Context context, Intent intent) {  
            // TODO Auto-generated method stub  
              
            /* 
             * 首先从Intent参数中取出一个Bundle对象，然后使用pdu密钥来提取一个SMS pdus数组，其中 
             * 每一个pdu都表示一条短信消息。接着使用SmsMessage的createFromPdu()方法将每一个 
             * pdu字节数组转换为SmsMessage对象，调用这个对象的getOriginatingAddress()方法 
             * 就可以获取到发送短信的发送方号码，调用getMessageBody()方法就可以获取到短信的内容
             * */  
            Bundle bundle=intent.getExtras();  
            Object[] pdus=(Object[]) bundle.get("pdus");//提取短信消息  
            SmsMessage[] messages=new SmsMessage[pdus.length];  
            for(int i=0;i<messages.length;i++){  
                messages[i]=SmsMessage.createFromPdu((byte[]) pdus[i]);  
            }  
            String address=messages[0].getOriginatingAddress();//获取发送号码  
            String fullMessage="";  
            for(SmsMessage message: messages){  
                fullMessage+=message.getMessageBody();//获取短信内容  
            }  
            sender.setText(address);  
            content.setText(fullMessage);  
        }           
    }  

对MessageReceiver进行注册才能让它接收到短信广播。

    public class MainActivity extends Activity {  
      
        private TextView sender;  
        private TextView content;  
        private IntentFilter receiverFilter;//过滤器  
        private MessageReceiver messageReceiver;//广播接收器  
        @Override  
        protected void onCreate(Bundle savedInstanceState) {  
            super.onCreate(savedInstanceState);  
            setContentView(R.layout.activity_main);  
            sender=(TextView) findViewById(R.id.sender);  
            content=(TextView) findViewById(R.id.content);  
            receiverFilter=new IntentFilter();  
            receiverFilter.addAction("android.provider.Telephony.SMS_RECEIVED");  
            messageReceiver=new MessageReceiver();  
            registerReceiver(messageReceiver,receiverFilter);//注册广播  
        }           
          
        @Override  
        protected void onDestroy() {  
            // TODO Auto-generated method stub  
            super.onDestroy();  
            unregisterReceiver(messageReceiver);//解绑广播  
        }
    }

增加权限

    <!--增加权限  -->  
    <uses-permission android:name="android.permission.RECEIVE_SMS"/>

## 拦截短信

系统发出的短信广播是一条有序广播，因此我们可以截断它。

一是提高MessageReceiver的优先级，让它能够优先系统短信程序接收到短信广播。

    receiverFilter.setPriority(100);//设计优先级  

二是在onReceive()方法中调用abortBroadcast()方法，中止掉广播的继续传递。

    abortBroadcast();//截断广播  

## 发送短信

先调用SmsManager的getDefault()方法获取到SmsManager的实例，然后再调用它的**sendTextMessage()**方法就可以去发送短信了。sendTextMessage()方法接收5个参数，其中第一个参数用于指定接收人的手机号码，第三个参数用于指定短信的内容，其他的几个参数我们暂时用不到， 直接传入null就可以了。

    SmsManager smsManager=SmsManager.getDefault();  
    smsManager.sendTextMessage("1592865****",  
            null, "message text",   
            null, null); 

发送短信需要权限，因此修改AndroidManifest.xml中的代码.

    <!--增加发送短信权限  -->  
    <uses-permission android:name="android.permission.SEND_SMS"/> 

如果想知道到底短信发送成功了没有，这个时候就可以利用sendTextMessage()方法的第四个参数来对短信的发送状态进行监控。

    public class MainActivity extends Activity {  
      
        private TextView sender;  
        private TextView content;  
        private IntentFilter receiverFilter;//过滤器  
        private MessageReceiver messageReceiver;//广播接收器  
        private EditText to;//接收人号码  
        private EditText msgInput;//短信内容  
        private Button send;//发送按钮  
          
        private IntentFilter sendFilter;//发送短信过滤器  
        private SendStatusReceiver sendStatusReceiver;//发送短信广播接收器  
        @Override  
        protected void onCreate(Bundle savedInstanceState) {  
            super.onCreate(savedInstanceState);  
            setContentView(R.layout.activity_main);  
            sender=(TextView) findViewById(R.id.sender);  
            content=(TextView) findViewById(R.id.content);  
            receiverFilter=new IntentFilter();  
            receiverFilter.addAction("android.provider.Telephony.SMS_RECEIVED");  
            receiverFilter.setPriority(100);//设计优先级  
            messageReceiver=new MessageReceiver();  
            registerReceiver(messageReceiver,receiverFilter);//注册广播  
              
              
            to=(EditText) findViewById(R.id.to);  
            msgInput=(EditText) findViewById(R.id.msg_input);  
            send=(Button) findViewById(R.id.send);  
              
            sendFilter=new IntentFilter();  
            sendFilter.addAction("SENT_SMS_ACTION");  
            sendStatusReceiver=new SendStatusReceiver();  
            registerReceiver(sendStatusReceiver,sendFilter);  
              
            send.setOnClickListener(new OnClickListener(){  
      
                @Override  
                public void onClick(View v) {  
                    // TODO Auto-generated method stub  
                    /* 
                     * 当send按钮被点击时，会先调用SmsManager的getDefault()方法 
                     * 获取到SmsManager的实例，然后再调用它的sendTextMessage()方法就可以 
                     * 去发送短信了。sendTextMessage()方法接收5个参数，其中第一个参数用于指定 
                     * 接收人的手机号码，第三个参数用于指定短信的内容，其他的几个参数我们暂时用不到， 
                     * 直接传入null就可以了。 
                     * */  
                    SmsManager smsManager=SmsManager.getDefault();  
                    Intent sendIntent=new Intent("SENT_SMS_ACTION");  
                    PendingIntent pi=PendingIntent.getBroadcast(MainActivity.this,  
                            0, sendIntent, 0);  
                    smsManager.sendTextMessage(to.getText().toString(),  
                            null, msgInput.getText().toString(),   
                            pi, null);  
                    /* 
                     * 在send按钮的点击事件里面我们调用了PendingIntent.getBroadcast()方法 
                     * 获取到一个PendingIntent对象，并将它作为第四个参数传递到sendTextMessage() 
                     * 方法中。然后又注册一个新的广播接收器SendStatusReceiver，这个广播接收器就是专门 
                     * 用于监听短信发送状态的，当getResultCode()==RESULT_OK就会提示发送成功，否则 
                     * 提示发送失败。 
                     * */  
                }  
                  
            });  
              
        }  
        
        @Override  
        protected void onDestroy() {  
            // TODO Auto-generated method stub  
            super.onDestroy();  
            unregisterReceiver(messageReceiver);//解绑广播  
            unregisterReceiver(sendStatusReceiver);//解绑广播  
        }  
      
      
           
        class MessageReceiver extends BroadcastReceiver{  
         //定义广播接收器  
          
            @Override  
            public void onReceive(Context context, Intent intent) {  
                // TODO Auto-generated method stub  
                  
                /* 
                 * 首先从Intent参数中取出一个Bundle对象，然后使用pdu密钥来提取一个SMS pdus数组，其中 
                 * 每一个pdu都表示一条短信消息。接着使用SmsMessage的createFromPdu()方法将每一个 
                 * pdu字节数组转换为SmsMessage对象，调用这个对象的getOriginatingAddress()方法 
                 * 就可以获取到发送短信的发送方号码，调用getMessageBody()方法就可以获取到短信的内容，然后 
                 * 将获取到的发送方号码和短信内容显示在TextView上。 
                 * */  
                Bundle bundle=intent.getExtras();  
                Object[] pdus=(Object[]) bundle.get("pdus");//提取短信消息  
                SmsMessage[] messages=new SmsMessage[pdus.length];  
                for(int i=0;i<messages.length;i++){  
                    messages[i]=SmsMessage.createFromPdu((byte[]) pdus[i]);  
                }  
                String address=messages[0].getOriginatingAddress();//获取发送号码  
                String fullMessage="";  
                for(SmsMessage message: messages){  
                    fullMessage+=message.getMessageBody();//获取短信内容  
                }  
                sender.setText(address);  
                content.setText(fullMessage);  
                abortBroadcast();//截断广播  
            }  
              
        }  
          
          
        class SendStatusReceiver extends BroadcastReceiver{  
      
            @Override  
            public void onReceive(Context context, Intent intent) {  
                // TODO Auto-generated method stub  
                if(getResultCode()==RESULT_OK){  
                    //短信发送成功  
                    Toast.makeText(context, "send succeed",   
                            Toast.LENGTH_LONG).show();;  
                }else{  
                    //短信发送失败  
                    Toast.makeText(context, "send failed",  
                            Toast.LENGTH_LONG).show();;  
                }  
            }  
              
        }  
    }  