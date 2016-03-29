---
layout:     post
title:      "Android的数据存储技术"
subtitle:   "《第一行代码》复习笔记5"
date:       2016-03-29
author:     "WunWun"
header-img: "img/android-backpack.jpg"
tags:
    - Android
    - 学习笔记
---

> 这是《第一行代码》复习笔记的第六章.

## Catagory

1. [文件存储](#文件存储)
    1. [将数据存储到文件中](#将数据存储到文件中)
    2. [从文件中读取数据](#从文件中读取数据)

2. [SharedPreference存储](#SharedPreference存储)
    1. [将数据存储到SharedPreferences中](#将数据存储到SharedPreferences中)
    2. [从SharedPreferences中读取数据](#从SharedPreferences中读取数据)

3. [数据库存储](#数据库存储)
    1. [创建数据库](#创建数据库)
    2. [升级数据库](#升级数据库)
    3. [添加数据](#添加数据)
    4. [更新数据](#更新数据)
    5. [删除数据](#删除数据)
    6. [查询数据](#查询数据)
    7. [使用SQL操作数据库](#使用SQL操作数据库)

---

## 文件存储

文件存储是Android中最基本的一种数据存储方式，它不对存储的内容进行任何的格式化处理，所有数据都是原封不动地保存到文件当中的，因而它比较适合用于存储一些简单的文本数据或二进制数据。

#### 将数据存储到文件中

Context 类中提供了一个**openFileOutput()**方法，可以用于将数据存储到指定的文件中。这个方法接收两个参数，第一个参数是文件名，在文件创建的时候使用的就是这个名称，注意这里指定的文件名不可以包含路径，因为所有的文件都是默认存储到/data/data/<package name>/files/目录下的。第二个参数是文件的操作模式 ，主要有两种模式可选 ，MODE_PRIVATE和 MODE_APPEND。其中 MODE_PRIVATE是默认的操作模式，表示当指定同样文件名的时候，所写入的内容将会覆盖原文件中的内容，而 MODE_APPEND则表示如果该文件已存在就往文件里面追加内容，不存在就创建新文件。

openFileOutput()方法返回的是一个FileOutputStream对象，得到了这个对象之后就可以使用Java流的方式将数据写入到文件中了。以下是一段简单的代码示例，展示了如何将一 段文本内容保存到文件中：

    public void save() {
        String data = "Data to save";
        FileOutputStream out = null;
        BufferedWriter writer = null;
        try {
            out = openFileOutput("data", Context.MODE_PRIVATE);
            writer = new BufferedWriter(new OutputStreamWriter(out));
            writer.write(data);
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                if (writer != null) {
                    writer.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

#### 从文件中读取数据

Context类中还提供了一个**openFileInput()**方法，用于从文件中读取数据。它只接收一个参数，即要读取的文件名，然后系统会自动到/data/data/<package name>/files/目录下去加载这个文件，并返回一个FileInputStream对象，得到了这个对象之后再通过Java流的方式就可以将数据读取出来了。

    public String load() {
        FileInputStream in = null;
        BufferedReader reader = null;
        StringBuilder content = new StringBuilder();

        try {
            in = openFileInput("data");
            reader = new BufferedReader(new InputStreamReader(in));
            String line = "";
            while ((line = reader.readLine()) != null) {
                content.append(line);
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (reader != null) {
                try {
                    reader.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        return content.toString();
    }

在这段代码中，首先通过**openFileInput()**方法获取到了一个**FileInputStream**对象，然后借助它又构建出了一个**InputStreamReader**对象，接着再使用InputStreamReader构建出一个**BufferedReader**对象，这样我们就可以通过BufferedReader进行一行行地读取，把文件中所有的文本内容全部读取出来并存放在一个StringBuilder对象中，最后将读取到的内容返回就可以了。

---

## SharedPreferences存储

> SharedPreferences 是使用键值对的方式来存储数据的。

#### 将数据存储到SharedPreferences中

要想使用SharedPreferences来存储数据，首先需要获取到SharedPreferences对象。Android中主要提供了三种方法用于得到SharedPreferences对象。

1.  Context 类中的 getSharedPreferences()方法

此方法接收两个参数，第一个参数用于指定SharedPreferences文件的名称，如果指
定的文件不存在则会创建一个。第二个参数用于指定操作模式，主要有两种模式可以选择，MODE_PRIVATE和MODE_MULTI_PROCESS。MODE_PRIVATE仍然是默认的操作模式，和直接传入0效果是相同的，表示只有当前的应用程序才可以对这个SharedPreferences文件进行读写。MODE_MULTI_PROCESS则一般是用于会有多个进程中对同一个SharedPreferences文件进行读写的情况。

2. Activity 类中的 getPreferences()方法

这个方法和Context中的getSharedPreferences()方法很相似，不过它只接收一个操作模式参数，因为使用这个方法时会自动将当前活动的类名作为SharedPreferences的文件名。

3. PreferenceManager 类中的 getDefaultSharedPreferences()方法

这是一个静态方法，它接收一个Context参数，并自动使用当前应用程序的包名作为前缀来命名SharedPreferences文件。

得到了SharedPreferences对象之后，就可以开始向SharedPreferences文件中存储数据了，主要可以分为三步实现。

1. 调用SharedPreferences对象的edit()方法来获取一个SharedPreferences.Editor对象。

2. 向SharedPreferences.Editor对象中添加数据，比如添加一个布尔型数据就使用putBoolean方法，添加一个字符串则使用putString()方法，以此类推。

3. 调用commit()方法将添加的数据提交，从而完成数据存储操作。

代码如下所示：

    SharedPreferences.Editor editor = getSharedPreferences("data", MODE_PRIVATE).edit();
    editor.putString("name", "Tom"); editor.putInt("age", 28); editor.putBoolean("married", false);
    editor.commit();

#### 从SharedPreferences中读取数据

SharedPreferences对象中提供了一系列的get方法用于对存储的数据进行读取，每种get方法都对应了SharedPreferences.Editor中的一种put方法，比如读取一个布尔型数据就使用getBoolean()方法，读取一个字符串就使用getString()方法。这些get方法都接收两个参数，第一个参数是键，传入存储数据时使用的键就可以得到相应的值了，第二个参数是默认值，即表示当传入的键找不到对应的值时，会以什么样的默认值进行返回。

    SharedPreferences pref = getSharedPreferences("data", MODE_PRIVATE);
    String name = pref.getString("name", "");
    int age = pref.getInt("age", 0);
    boolean married = pref.getBoolean("married", false);

---

## SQLite数据库存储

Android为了让我们能够更加方便地管理数据库，专门提供了一个SQLiteOpenHelper帮助类，借助这个类就可以非常简单地对数据库进行创建和升级。

SQLiteOpenHelper是一个**抽象类**，这意味着如果我们想要使用它的话，就需要创建一个自己的帮助类去继承它。SQLiteOpenHelper中有两个抽象方法，分别是**onCreate()**和**onUpgrade()**，我们必须在自己的帮助类里面重写这两个方法，然后分别在这两个方法中去实现创建、升级数据库的逻辑。

SQLiteOpenHelper中还有两个非常重要的实例方法，**getReadableDatabase()**和**getWritableDatabase()**。这两个方法都可以创建或打开一个现有的数据库（如果数据库已存在则直接打开，否则创建一个新的数据库），并返回一个可对数据库进行读写操作的对象。不同的是，当数据库不可写入的时候（如磁盘空间已满）getReadableDatabase()方法返回的对象将以只读的方式去打开数据库，而getWritableDatabase()方法则将出现异常。

#### 创建数据库

![Android-SQLiteOpenHelper-Constructor](/img/in-post/the-first-line-of-code/SQLiteOpenHelper-Constructor.jpg)
<small class="img-hint">SQLiteOpenHelper构造函数</small>

我们希望创建一个名为BookStore.db的数据库，然后在这个数据库中新建一张Book表，表中有id（主键）、作者、价格、页数和书名等列。创建数据库表当然还是需要用建表语句的。

    create table Book (
        id integer primary key autoincrement, 
        author text,
        price real, 
        pages integer, 
        name text)

然后需要在代码中去执行这条SQL语句，才能完成创建表的操作。

    public class MyDatabaseHelper extends SQLiteOpenHelper {
        public static final String CREATE_BOOK = "create table book ("
                + "id integer primary key autoincrement, "
                + "author text, "
                + "price real, "
                + "pages integer, "
                + "name text)";
        private Context mContext;
        public MyDatabaseHelper(Context context, String name, CursorFactory factory, int version) {
            super(context, name, factory, version);
            mContext = context;
        }
        @Override
        public void onCreate(SQLiteDatabase db) {
            db.execSQL(CREATE_BOOK);
            Toast.makeText(mContext, "Create succeeded", Toast.LENGTH_SHORT).show();
        }
        @Override
        public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
        }
    }

可以看到，我们把建表语句定义成了一个字符串常量，然后在onCreate()方法中又调用了SQLiteDatabase的**execSQL()**方法去执行这条建表语句。

然后在MainActivity中创建数据库。

    private MyDatabaseHelper dbHelper;
    dbHelper=new MyDatabaseHelper(this,"BookStore.db",null,1);
    dbHelper.getWritableDatabase();

#### 升级数据库

我们想再添加一张Category表用于记录书籍的分类。

    create table Category (
    id integer primary key autoincrement,
    category_name text,
    category_code integer)

接下来我们将这条建表语句添加到MyDatabaseHelper中，

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
        private Context mContext;
        public MyDatabaseHelper(Context context, String name, CursorFactory factory, int version) {
            super(context, name, factory, version);
            mContext = context;
        }
        @Override
        public void onCreate(SQLiteDatabase db) {
            db.execSQL(CREATE_BOOK);
            db.execSQL(CREATE_CATEGORY);
            Toast.makeText(mContext, "Create succeeded", Toast.LENGTH_SHORT).
                    show();
        }
        @Override
        public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
            db.execSQL("drop table if exists Book");
            db.execSQL("drop table if exists Category");
            onCreate(db);
        }
    }

在MainActivity中修改代码

    private MyDatabaseHelper dbHelper;
    dbHelper = new MyDatabaseHelper(this, "BookStore.db", null, 2); 
    dbHelper.getWritableDatabase();

这里将数据库版本号指定为2（只要传入一个比**上一个版本大的数**，就可以让onUpgrade()方法得到执行），表示我们对数据库进行升级了。

#### 添加数据

    private MyDatabaseHelper dbHelper;
    dbHelper = new MyDatabaseHelper(this, "BookStore.db", null, 2); 

    SQLiteDatabase db = dbHelper.getWritableDatabase(); 
    ContentValues values = new ContentValues();
    // 开始组装第一条数据
    values.put("name", "The Da Vinci Code"); 
    values.put("author", "Dan Brown"); 
    values.put("pages", 454); 
    values.put("price", 16.96);
    db.insert("Book", null, values); // 插入第一条数据
    values.clear();
    // 开始组装第二条数据
    values.put("name", "The Lost Symbol"); 
    values.put("author", "Dan Brown"); 
    values.put("pages", 510); 
    values.put("price", 19.95);
    db.insert("Book", null, values); // 插入第二条数据

#### 更新数据

    private MyDatabaseHelper dbHelper;
    dbHelper = new MyDatabaseHelper(this, "BookStore.db", null, 2); 

    SQLiteDatabase db = dbHelper.getWritableDatabase();
    ContentValues values = new ContentValues();
    values.put("price", 10.99);
    db.update("Book", values, "name = ?", new String[] { "The DaVinci Code" });

这里构建了一个ContentValues对象，并且只给它指定了一组数据，说明我们只是想把价格这一列的数据更新成10.99。然后调用了SQLiteDatabase的update()方法去执行具体的更新操作，可以看到，这里使用了第三、第四个参数来指定具体更新哪几行。**第三个参数对应的是SQL语句的where部分，表示去更新所有name等于?的行，而?是一个占位符，可以通过第四个参数提供的一个字符串数组为第三个参数中的每个占位符指定相应的内容**。因此上述代码想表达的意图就是，将名字是The Da Vinci Code的这本书的价格改成10.99。

#### 删除数据

    private MyDatabaseHelper dbHelper;
    dbHelper = new MyDatabaseHelper(this, "BookStore.db", null, 2); 
    SQLiteDatabase db = dbHelper.getWritableDatabase();
    db.delete("Book", "pages > ?", new String[] { "500" })

SQLiteDatabase中提供了一个delete()方法专门用于删除数据，这个方法接收三个参数，第一个参数仍然是表名，第二、第三个参数又是用于去约束删除某一行或某几行的数据，不指定的话默认就是删除所有行。

#### 查询数据

![Android-SQLiteOpenHelper-Query](/img/in-post/the-first-line-of-code/SQLiteOpenHelper-Query.jpg)
<small class="img-hint">SQLiteOpenHelper中的query()方法</small>

    private MyDatabaseHelper dbHelper;
    dbHelper = new MyDatabaseHelper(this, "BookStore.db", null, 2); 
    SQLiteDatabase db = dbHelper.getWritableDatabase();

    // 查询Book表中所有的数据
    Cursor cursor = db.query("Book", null, null, null, null, null, null);
    if(cursor.moveToFirst())
    {
        do {
            // 遍历Cursor对象，取出数据并打印
            String name = cursor.getString(cursor.getColumnIndex("name"));
            String author = cursor.getString(cursor.getColumnIndex("author"));
            int pages = cursor.getInt(cursor.getColumnIndex("pages"));
            double price = cursor.getDouble(cursor.getColumnIndex("price"));
        } while (cursor.moveToNext());
    }
    cursor.close();

我们首先调用了SQLiteDatabase的query()方法去查询数据。这里的query()方法非常简单，只是使用了第一个参数指明去查询Book表，后面的参数全部为null。这就表示希望查询这张表中的所有数据，虽然这张表中目前只剩下一条数据了。查询完之后就得到了一个Cursor对象，接着我们调用它的**moveToFirst()**方法将数据的指针移动到第一行的位置，然后进入了一个循环当中，去遍历查询到的每一行数据。在这个循环中可以通过Cursor的**getColumnIndex()**方法获取到某一列在表中对应的位置索引，然后将这个索引传入到相应的取值方法中，就可以得到从数据库中读取到的数据了。最后别忘了调用**close()**方法来关闭Cursor。

#### 使用SQL操作数据库

添加数据：

    db.execSQL("insert into Book (name, author, pages, price) values(?, ?, ?, ?)", new String[] { "The Da Vinci Code", "Dan Brown", "454", "16.96" }); 
    db.execSQL("insert into Book (name, author, pages, price) values(?, ?, ?, ?)", new String[] { "The Lost Symbol", "Dan Brown", "510", "19.95" });

更新数据：

    db.execSQL("update Book set price = ? where name = ?", new String[] { "10.99", "The Da Vinci Code" });

删除数据：

    db.execSQL("delete from Book where pages > ?", new String[] { "500" });

查询数据：

    db.rawQuery("select * from Book", null);