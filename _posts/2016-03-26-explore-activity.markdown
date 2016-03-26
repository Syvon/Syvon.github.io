---
layout:     post
title:      "探究Activity"
subtitle:   "《第一行代码》复习笔记2"
date:       2016-03-26
author:     "WunWun"
header-img: "img/android-light.jpg"
tags:
    - Android
    - 学习笔记
---

> 这是《第一行代码》复习笔记的第二章.


## 在活动中使用Toast

> Toast是Android视同提供的一种非常好的提醒方式，在程序中可以使用它将一些短小的信息通知给用户。

    Button button = (Button)findViewById(R.id.button);
    button.setOnClickListener(new OnClickListener(){
        @Override
        public void onClick(View v) {
            Toast.makeText(MainActivity.this,"Hello,world",Toast.LENGTH_LONG).show();
        }
    });

在活动中，可以通过**findViewById()**方法获取到在布局文件中定义的元素，它接受一个资源id的参数，返回一个View对象，我们将它向下转型成Button对象。得到按钮的实例之后，通过调用**setOnClickListener()**方法为按钮注册一个监听器，点击按钮时就会执行监听器中的onClick()方法。

**makeText()**方法第一个参数是Context，也就是Toast要求的上下文，由于活动本身就是一个Context对象，因此直接传入MainActivity.this即可


## 在活动中使用menu

在res目录下面，新建menu文件夹，在这个文件夹下新建菜单文件。

	<?xml version="1.0" encoding="utf-8"?>
	<menu xmlns:android="http://schemas.android.com/apk/res/android">
    	<item
        	android:id="@+id/add_item"
        	android:title="Add" />

    	<item
        	android:id="@+id/remove_item"
        	android:title="Remove" />
	</menu>

在Activity中，重写**onCreateOptionsMenu()**方法，将菜单显示出来。

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.main,menu);
        return true;
    }


在Activity中重写**onOptionsItemSelected()**方法，定义菜单显示事件。

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        switch (item.getItemId()) {
            case R.id.add_item:
                Toast.makeText(this,"You clicked Add",Toast.LENGTH_LONG).show();
                break;
            case R.id.remove_item:
                Toast.makeText(this,"You clicked Remove",Toast.LENGTH_LONG).show();
                break;
            default:
        }
        return true;
    }


## Intent

> Intent是Android程序中各组件之间进行交互的一种重要方式，它不仅可以指明当前组件想要执行的动作，还可以在不同组件之间传递数据。Intent一般可被用于启动活动，启动服务，以及发送广播场景。

Intent用法大致分为两种：显式Intent和隐式Intent。

- 显式Intent

	Intent intent = new Intent(FirstActivity.this,SecondActivity.class);
	startActivity(intent);

- 隐式Intent

> 隐式Intent相对比较含蓄，它并不明确指出我们想要启动哪一个活动，而是指定了一系列更为抽象的action和category等信息，然后交由系统去分析这个Intent，并帮我们找出合适的活动去启动。

在AndroidManifest.xml中，通过在<activity>标签下配置<intent-filter>的内容，可以指定当前活动能够相应的action和category。

    <activity android:name=".MainActivity">
        <intent-filter>
            <action android:name="com.example.activitytest.ACTION_START" />
            <category android:name="android.intent.category.DEFAULT" />
        </intent-filter>
    </activity>

在<action>标签中，我们指明了当前活动可以响应com.example.activitytest.ACTION_START这个action，而<category>标签则包含了一些附加信息，更精确的指明了当前活动能够响应的Intent中还可能带有的category。只有<action>和<category>中的内容同时能够匹配上Intent中指定的action和category时，这个活动才能响应该Intent。

在activity中，我们可以这样启动Intent。

	Intent intent = new Intent("com.example.activitytest.ACTION_START");
	//intent.addCategory("com.example.activitytest.MY_CATEGORY");
	startActivity(intent);

**每个Intent中只能指定一个action，但却能指定多个category**

- 隐式Intent的更多用法

	Intent intent = new Intent(Intent.ACTION_VIEW);
	intent.setData(Uri.parse("http://www.baidu.com"));
	startActivity(intent);

setData()部分接受一个Uri对象，主要用于指定当前Intent正在操作的数据。

与此对应，我们可以在<intent-filter>标签中再配置一个<data>标签，用于更精确地指定当前活动能够响应什么类型的数据。

    <activity android:name=".MainActivity">
        <intent-filter>
            <action android:name="com.example.activitytest.ACTION_START" />
            <category android:name="android.intent.category.DEFAULT" />
            <data android:scheme="http" />
        </intent-filter>
    </activity>

- 向下一个活动传递数据

Intent提供了一系列方法的重载，可以把我们想要传递的数据暂存在Intent中。启动了另一个活动后，只需要把这些数据再从Intent中取出就可以了。

    button.setOnClickListener(new OnClickListener(){
        @Override
        public void onClick(View v) {
            String data = "Hello SecondActivity";
            Intent intent = new Intent(FirstActivity.this,SecondActivity.class);
            intent.putExtra("extra_data",data);
            startActivity(intent);
        }
    });

在SecondActivity中将传递的数据取出。

	Intent intent = getIntent();
	String data = intent.getStringExtra("extra_data");

- 返回数据给上一个活动

Activity还有一个startActivityForResult()方法用于启动活动，但这个方法期望在活动销毁的时候能够返回一个结果给上一个活动。

> startActivityForResult()方法接受两个参数，第一个参数还是Intent，第二个参数是请求码，用于在之后的回调中判断数据的来源。

    button.setOnClickListener(new OnClickListener(){
        @Override
        public void onClick(View v) {
            Intent intent = new Intent(FirstActivity.this,SecondActivity.class);
            startActivityForResult(intent，1);
        }
    });

向上一个活动返回数据。我们还是构建了一个Intent，只不过这个Intent仅仅是用于传递数据而已，没有指定任何的“意图”。setResult()专门用于向上一个活动返回数据。第一个参数用于向上一个活动返回处理结果，一般使用RESULT_OK或RESULT_CANCELLED。

    button2.setOnClickListener(new OnClickListener(){
        @Override
        public void onClick(View v) {
            Intent intent = new Intent();
            intent.putExtra("data_return",“Hello FirstActivity”);
            setResult(RESULT_OK,intent);
        }
    });

由于我们使用startActivityForResult()方法来启动SecondActivity，在SecondActivity被销毁之后会回调上一个活动的onActivityResult()方法。

	@Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        switch (requestCode) {
        case 1:
        	if (resultCode == RESULT_OK) {
        		String returnedData = data.getStringExtra("data_return");
        	}
        	break;
        default;
    	}
    }

## 活动的生命周期

1. 返回栈
	
	Android使用任务（Task）来管理活动，一个任务就是一组存放在栈里的活动的集合，这个栈也被称作返回栈（Back Stack）。每当我们启动一个新活动，它会在返回栈中入栈，并处于栈顶的位置。每当我们按下Back键或调用finish()方法去销毁一个活动时，处于栈顶的活动会出栈。系统总是会显示处于栈顶的活动给用户。

![Android-Activity-Task](/img/in-post/the-first-line-of-code/android-activity-task.png)
<small class="img-hint">返回栈</small>

2. 活动状态

	- 运行状态

	当一个活动位于栈顶位置，这时活动就处于运行状态。

	- 暂停状态

	当一个活动不再处于栈顶位置，但仍然可见时，这时活动就进入了暂停状态。

	- 停止状态

	当一个活动不再处于栈顶位置，并且完全不可见的时候，就进入了停止状态。

	- 销毁状态

	当一个活动从返回栈中移除后就变成了销毁状态。

3. 活动的七个方法

	- onCreate()

	它会在活动第一次被创建的时候调用。一般在这个方法中完成活动的初始化操作，比如加载布局，绑定事件等

	- onStart()

	这个方法在活动由不可见变为可见的时候调用。

	- onResume()

	这个方法在活动准备好和用户进行交互的时候调用。此时的活动一定位于返回栈的栈顶，并且处于运行状态。

	- onPause()

	这个方法在系统准备去启动或者恢复另一个活动的时候调用。通常在这个方法中将一些消耗CPU的资源释放掉，以及保存一些关键数据。但这个方法的执行速度一定要快，不然影响新的栈顶活动的使用。

	- onStop()

	这个方法在活动完全不可见的时候调用。

	- onDestroy()

	这个方法在活动被销毁之前调用。

	- onRestart()

	这个方法在活动由停止状态变为运行状态之前调用，也就是活动被重新启动了。

4. 活动的生存期

	- 完整生存期

	活动在onCreate()方法和onDestroy()方法之间所经历的，就是完整生存期。一般情况下，一个活动会在onCreate()方法中完成各种初始化操作，而在onDestroy()方法中完成释放内存的操作。

	- 可见生存期

	活动在onStart()方法和onStop()方法之间所经历的，就是可见生存期。在可见生存期内，活动对于用户总是可见的，即便有可能无法交互。我们可以通过这两个方法，合理的管理那些对用户可见的资源。

	- 前台生存期

	活动在onResume()方法和onPause()方法之间所经历的，就是前台生存期。在前台生存期内，活动总是处于运行状态的。

![Android-Activity-Life](/img/in-post/the-first-line-of-code/Android-Activity-Life.png)
<small class="img-hint">Android活动生命周期图</small>

5. 在活动被回收时保存临时数据

> Activity中提供了一个onSaveInstancState()回调方法，这个方法会保证一定在活动被回收之前调用，因此可以通过这个方法保存临时数据。

`
    @Override
    protected void onSaveInstanceState(Bundle outState) {
        super.onSaveInstanceState(outState);
        String tempData = "something you just typed";
        outState.putString("data_key",tempData);
    }	
`

onCreate()方法中有一个Bundle类型的参数。这个参数一般情况下都是null，但是当活动被系统回收之前有通过onSaveInstancState()保存数据的话，这个参数就会带有之前保存的全部数据。

`
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        if (savedInstanceState != null) {
            String tempData = savedInstanceState.getString("data_key");
        }
    }
`

## 活动的启动模式

![Android-LaunchMode](/img/in-post/the-first-line-of-code/activity-launchmode.jpg)
<small class="img-hint">活动的启动模式</small>

- standard

在standard模式（即默认）下，每当启动一个新的活动，他就会在返回栈中入栈，并处于栈顶的位置。系统不会在乎这个活动是否已经在返回栈中存在，每次启动都会创建该活动的一个实例。

![Android-Launch-Standard](/img/in-post/the-first-line-of-code/android-launch-standard.png)
<small class="img-hint">standard模式</small>

- singleTop

在启动活动的时候，如果发现返回栈的栈顶已经是该活动，则认为可以直接使用它，不会再创建新的活动实例。

![Android-Launch-singleTop](/img/in-post/the-first-line-of-code/android-launch-singleTop.png)
<small class="img-hint">singleTop模式</small>

- singleTask

每次启动该活动时，系统会在返回栈中检查是否存在该活动的实例，如果发现已经存在则直接使用该实例，并把这个活动之上的所有活动统统出栈，如果没有发现就会创建一个新的活动实例。

![Android-Launch-singleTask](/img/in-post/the-first-line-of-code/android-launch-singleTask.png)
<small class="img-hint">singleTask模式</small>

- singleInstance

系统会启用一个新的返回栈来管理这个活动。

![Android-Launch-singleInstance](/img/in-post/the-first-line-of-code/android-launch-singleInstance.png)
<small class="img-hint">singleInstance模式</small>
