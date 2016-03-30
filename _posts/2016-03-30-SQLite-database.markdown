---
layout:     post
title:      "SQLite数据库的最佳实践"
subtitle:   "如何用好SQLite数据库"
date:       2016-03-30
author:     "WunWun"
header-img: "img/android-cute.jpg"
tags:
    - Android
    - SQLite
---


## 使用事务

SQLite数据库是支持事务的，事务的特性可以保证让某一系列的操作要么全部完成，要么一个都不完成。

    private MyDatabaseHelper dbHelper;
    dbHelper = new MyDatabaseHelper(this, "BookStore.db", null, 2); 
    SQLiteDatabase db = dbHelper.getWritableDatabase();
    db.beginTransaction(); // 开启事务
    try {
    db.delete("Book", null, null);
    if (true) {
    // 在这里手动抛出一个异常，让事务失败
    throw new NullPointerException();
    }
    ContentValues values = new ContentValues(); values.put("name", "Game of Thrones"); values.put("author", "George Martin"); values.put("pages", 720); values.put("price", 20.85); db.insert("Book", null, values);
    db.setTransactionSuccessful(); // 事务已经执行成功
    } catch (Exception e) {
    e.printStackTrace();
    } finally {
    db.endTransaction(); // 结束事务
    }

首先调用SQLiteDatabase的**beginTransaction()**方法来开启一个事务，然后在一个异常捕获的代码块中去执行具体的数据库操作，当所有的操作都完成之后，调用**setTransactionSuccessful()**表示事务已经执行成功了，最后在finally代码块中调用**endTransaction()**来结束事务。

## 升级数据库的最佳写法

> 每一个数据库版本都会对应一个版本号，当指定的数据库版本号大于当前数据库版本号的时候，就会进入到onUpgrade()方法中去执行更新操作。这里需要为每一个版本号赋予它各自改变的内容，然后在onUpgrade()方法中对当前数据库的版本号进行判断，再执行相应的改变就可以了。

第一版的程序要求非常简单，只需要创建一张Book表.

    public class MyDatabaseHelper extends SQLiteOpenHelper {
        public static final String CREATE_BOOK = "create table Book (" 
            + "id integer primary key autoincrement, "
            + "author text, " 
            + "price real, " 
            + "pages integer, " 
            + "name text)";

        public MyDatabaseHelper(Context context, String name, CursorFactory factory, int version) {
            super(context, name, factory, version);
        }

        @Override
        public void onCreate(SQLiteDatabase db) {
            db.execSQL(CREATE_BOOK);
        }

        @Override
        public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
        }
    }

不过，几星期之后又有了新需求，这次需要向数据库中再添加一张Category表。于是，修改MyDatabaseHelper中的代码，如下所示：

    public class MyDatabaseHelper extends SQLiteOpenHelper {

        public static final String CREATE_BOOK = "create table Book (" 
            + "id integer primary key autoincrement, "
            + "author text, " 
            + "price real, " 
            + "pages integer, " 
            + "name text)";

        public static final String CREATE_CATEGORY = "create table Category ("
            + "id integer primary key autoincrement, " 
            + "category_name text, " 
            + "category_code integer)";

        public MyDatabaseHelper(Context context, String name, CursorFactory factory, int version) {
            super(context, name, factory, version);
        }

        @Override
        public void onCreate(SQLiteDatabase db) {
            db.execSQL(CREATE_BOOK);
            db.execSQL(CREATE_CATEGORY);
        }

        @Override
        public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
            switch (oldVersion) {
            case 1:
                db.execSQL(CREATE_CATEGORY);
            default:
            }
        }
    }

可以看到，在onCreate()方法里我们新增了一条建表语句，然后又在onUpgrade()方法中添加了一个switch判断，如果用户当前数据库的版本号是1，就只会创建一张Category表。这样当用户是直接安装的第二版的程序时，就会将两张表一起创建。而当用户是使用第二版的程序覆盖安装第一版的程序时，就会进入到升级数据库的操作中，此时由于Book表已经存在了，因此只需要创建一张Category表即可。

没过多久，新的需求又来了，这次要给Book表和Category表之间建立关联，需要在Book表中添加一个category_id的字段。再次修改MyDatabaseHelper中的代码，如下所示：

    public class MyDatabaseHelper extends SQLiteOpenHelper {
        public static final String CREATE_BOOK = "create table Book (" + "id integer primary key autoincrement, "
            + "author text, " 
            + "price real, " 
            + "pages integer, " 
            + "name text, " 
            + "category_id integer)";
            
        public static final String CREATE_CATEGORY = "create table Category ("
                + "id integer primary key autoincrement, " 
                + "category_name text, " 
                + "category_code integer)";

        public MyDatabaseHelper(Context context, String name, CursorFactory factory, int version) {
            super(context, name, factory, version);
        }

        @Override
        public void onCreate(SQLiteDatabase db) {
            db.execSQL(CREATE_BOOK);
            db.execSQL(CREATE_CATEGORY);
        }

        @Override
        public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
            switch (oldVersion) {
            case 1:
                db.execSQL(CREATE_CATEGORY);
            case 2:
                db.execSQL("alter table Book add column category_id integer");
            default:
            }
        }
    }

首先我们在Book表的建表语句中添加了一个category_id列，这样当用户直接安装第三版的程序时，这个新增的列就已经自动添加成功了。然而，如果用户之前已经安装了某一版本的程序，现在需要覆盖安装，就会进入到升级数据库的操作中。在onUpgrade()方法里，我们添加了一个新的case，如果当前数据库的版本号是2，就会执行alter命令来为Book表新增一个category_id列。

这里请注意一个非常重要的细节，switch中每一个case的最后都是没有使用break的，为什么要这么做呢？这是为了保证在跨版本升级的时候，每一次的数据库修改都能被全部执行到。比如用户当前是从第二版程序升级到第三版程序的，那么case2中的逻辑就会执行。而如果用户是直接从第一版程序升级到第三版程序的，那么case1和case2中的逻辑都会执行。使用这种方式来维护数据库的升级，不管版本怎样更新，都可以保证数据库的表结构是最新的，而且表中的数据也完全不会丢失了。