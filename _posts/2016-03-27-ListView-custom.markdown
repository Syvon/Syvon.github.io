---
layout:     post
title:      "ListView控件"
subtitle:   "《第一行代码》复习笔记3"
date:       2016-03-27
author:     "WunWun"
header-img: "img/android-rain.jpg"
tags:
    - Android
    - 学习笔记
---

> 这是《第一行代码》复习笔记的第三章.


## ListView的简单用法

在布局中加入ListView控件

    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:orientation="vertical" 
        android:layout_width="match_parent"
        android:layout_height="match_parent">
        <ListView
            android:id="@+id/list_view"
            android:layout_width="match_parent"
            android:layout_height="match_parent">
        </ListView>
    </LinearLayout>

数据是无法直接传递给ListView的，需要借助适配器来完成。在ArrayAdapter的构造函数中依次传递当前上下文，ListView子项布局的id，以及要适配的数据。最后调用ListView的setAdapter()方法，将构建好的适配器对象传递进去。


    public class MainActivity extends Activity {
        private String[] data = {"Apple","Banana","Orange","Watermelon",
                "Pear","Grape","Pineapple","Strawberry","Cherry","Mango"};
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
            ArrayAdapter<String> adapter = new ArrayAdapter<String>(
                    MainActivity.this,android.R.layout.simple_list_item_1,data
            );
            ListView listView = (ListView)findViewById(R.id.list_view);
            listView.setAdapter(adapter);
        }
    }

![Android-ListView-Simple](/img/in-post/the-first-line-of-code/android-listview-simple.png)
<small class="img-hint">ListView控件</small>

## 定制ListView的界面

定义一个实体类，作为ListView适配器的适配类型

    public class Fruit {
        private String name;
        private  int imageId;
        public Fruit(String name, int imageId) {
            this.name = name;
            this.imageId = imageId;
        }
        public String getName() {
            return name;
        }
        public int getImageId() {
            return imageId;
        }
    }

为ListView的子项指定一个我们自定义的布局。定义了一个ImageView用于显示水果的图片，又定义了一个TextView用于显示水果的名称。

    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:orientation="vertical" android:layout_width="match_parent"
        android:layout_height="match_parent">
        <ImageView
            android:id="@+id/fruit_image"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content" />
        <TextureView
            android:id="@+id/fruit_name"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center"
            android:layout_marginLeft="10dip"/>
    </LinearLayout>

创建一个自定义的适配器，这个适配器继承自ArrayAdapter，并将泛型指定为Fruit类。

    public class FruitAdapter extends ArrayAdapter<Fruit> {
        private int resourceId;
        public FruitAdapter(Context context, int resource, List<Fruit> objects) {
            super(context, resource, objects);
            resourceId = resource;
        }
        @Override
        public View getView(int position, View convertView, ViewGroup parent) {
            Fruit fruit = getItem(position);
            View view = LayoutInflater.from(getContext()).inflate(resourceId,null);
            ImageView fruitImage = (ImageView)view.findViewById(R.id.fruit_image);
            TextView fruitName = (TextView)view.findViewById(R.id.fruit_name);
            fruitImage.setImageResource(fruit.getImageId());
            fruitName.setText(fruit.getName());
            return view;
        }
    }

    getView()方法在每个子项被滚动到屏幕内的时候会被调用。在getView()方法中，首先通过getItem()方法得到当前项的Fruit实例，然后使用LayoutInflater来为每个子项加载我们传入的布局。

修改MainActivity中的代码。

    public class MainActivity extends Activity {
        private List<Fruit> fruitList = new ArrayList<Fruit>();
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
            initFruits();
            FruitAdapter adapter = new FruitAdapter(
                    MainActivity.this,R.layout.fruit_item,fruitList
            );
            ListView listView = (ListView)findViewById(R.id.list_view);
            listView.setAdapter(adapter);
        }
        private void initFruits() {
            Fruit apple = new Fruit("Apple",R.drawable.apple);
            fruitList.add(apple);
            Fruit banana = new Fruit("Banana",R.drawable.banana);
            fruitList.add(banana);
            Fruit orange = new Fruit("Orange",R.drawable.orange);
            fruitList.add(orange);
            Fruit watermelon = new Fruit("Watermelon",R.drawable.watermelon);
            fruitList.add(watermelon);
            Fruit pear = new Fruit("Pear",R.drawable.pear);
            fruitList.add(pear);
            Fruit grape = new Fruit("Grape",R.drawable.grape);
            fruitList.add(grape);
            Fruit pineapple = new Fruit("pineapple",R.drawable.pineapple);
            fruitList.add(pineapple);
            Fruit strawberry = new Fruit("strawbberry",R.drawable.strawberry);
            fruitList.add(strawberry);
            Fruit cherry = new Fruit("cherry",R.drawable.cherry);
            fruitList.add(cherry);
            Fruit mango = new Fruit("mango",R.drawable.mango);
            fruitList.add(mango);
        }
    }

![Android-ListView-Custom](/img/in-post/the-first-line-of-code/android-listview-custom.png)
<small class="img-hint">自定义ListView控件</small>

## 提升ListView的运行效率

目前我们ListView的运行效率是很低的，因为在 FruitAdapter的getView()方法中每次都将布局重新加载了一遍，当ListView快速滚动的时候这就会成为性能的瓶颈。仔细观察，getView()方法中还有一个**convertView 参数，这个参数用于将之前加载好的布局进行缓存，以便之后可以进行重用**。

    public class FruitAdapter extends ArrayAdapter<Fruit> {
    ……
    　　@Override
    　　public View getView(int position, View convertView, ViewGroup parent) {
    　　　　Fruit fruit = getItem(position);
    　　　　View view;
          if (convertView == null) {
          　　view = LayoutInflater.from(getContext()).inflate(resourceId, null);
          } else {
          　　view = convertView;
          }
    　　　　ImageView fruitImage = (ImageView) view.findViewById(R.id.fruit_image);
    　　　　TextView fruitName = (TextView) view.findViewById(R.id.fruit_name);
    　　　　fruitImage.setImageResource(fruit.getImageId());
    　　　　fruitName.setText(fruit.getName());
    　　　　return view;
    　　}
    }

现在我们在 getView()方法中进行了判断，如果 convertView 为空，则使用LayoutInflater 去加载布局，如果不为空则直接对 convertView进行重用。这样就大大提高了ListView的运行效率，在快速滚动的时候也可以表现出更好的性能。

不过，目前我们的这份代码还是可以继续优化的，虽然现在已经不会再重复去加载布局，但是每次在getView()方法中还是会调用View的findViewById()方法来获取一次控件的实例。我们可以借助一个**ViewHolder**来对这部分性能进行优化，修改 FruitAdapter 中的代码，如下所示：

    public class FruitAdapter extends ArrayAdapter<Fruit> {
    ……
    　　@Override
    　　public View getView(int position, View convertView, ViewGroup parent) {
    　　　　Fruit fruit = getItem(position);
    　　　　View view;
    　　　　ViewHolder viewHolder;
    　　　　if (convertView == null) {
    　　　　　　view = LayoutInflater.from(getContext()).inflate(resourceId, null);
    　　　　　　viewHolder = new ViewHolder();
    　　　　　　viewHolder.fruitImage = (ImageView) view.findViewById(R.id.fruit_image);
    　　　　　　viewHolder.fruitName = (TextView) view.findViewById(R.id.fruit_name);
    　　　　　　view.setTag(viewHolder); //  将ViewHolder 存储在View 中
    　　　　} else {
    　　　　　　view = convertView;
    　　　　　　viewHolder = (ViewHolder) view.getTag(); //  重新获取ViewHolder
    　　　　}
    　　　　viewHolder.fruitImage.setImageResource(fruit.getImageId());
    　　　　viewHolder.fruitName.setText(fruit.getName());
    　　　　return view;
    　　}
    　　class ViewHolder {
    　　　　ImageView fruitImage;
    　　　　TextView fruitName;
    　　}
    }

我们**新增了一个内部类ViewHolder，用于对控件的实例进行缓存**。当convertView为空的时候，创建一个ViewHolder对象，并将控件的实例都存放在 ViewHolder 里，然后调用 View的 setTag()方法，将 ViewHolder 对象存储在 View 中。当 convertView 不为空的时候则调用View的 getTag()方法，把 ViewHolder 重新取出。这样所有控件的实例都缓存在了 ViewHolder里，就没有必要每次都通过 findViewById()方法来获取控件实例了。

##ListView的点击事件

ListView 的滚动毕竟只是满足了我们视觉上的效果，可是如果ListView中的子项不能点击的话，这个控件就没有什么实际的用途了。因此，我们来学习一下ListView如何才能响应用户的点击事件。

    public class MainActivity extends Activity {
    　　private List<Fruit> fruitList = new ArrayList<Fruit>();
    　　@Override
    　　protected void onCreate(Bundle savedInstanceState) {
    　　　　super.onCreate(savedInstanceState);
    　　　　setContentView(R.layout.activity_main);
    　　　　initFruits();
    　　　　FruitAdapter adapter = new FruitAdapter(MainActivity.this,R.layout.fruit_item, fruitList);
    　　　　ListView listView = (ListView) findViewById(R.id.list_view);
    　　　　listView.setAdapter(adapter);
    　　　　listView.setOnItemClickListener(new OnItemClickListener() {
    　　　　@Override
    　　　　　　public void onItemClick(AdapterView<?> parent, View view,
    　　　　　　int position, long id) {
    　　　　　　　　Fruit fruit = fruitList.get(position);
    　　　　　　　　Toast.makeText(MainActivity.this, fruit.getName(),
    　　　　　　　　Toast.LENGTH_SHORT).show();
    　　　　}
    　　});
        }
    ……
    }

我们使用了**setOnItemClickListener()**方法来为 ListView 注册了一个监听器，当用户点击了 ListView中的任何一个子项时就会回调 onItemClick()方法，在这个方法中可以通过 position 参数判断出用户点击的是哪一个子项，然后获取到相应的水果，并通过 Toast将水果的名字显示出来。