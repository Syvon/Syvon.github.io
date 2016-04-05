---
layout:     post
title:      "探究内容提供器"
subtitle:   "《第一行代码》复习笔记6"
date:       2016-03-31
author:     "WunWun"
header-img: "img/Android-normal.png"
tags:
    - Android
    - 学习笔记
---

> 这是《第一行代码》复习笔记的第七章.

## 简介

内容提供器（Content Provider）主要用于**在不同的应用程序之间实现数据共享的功能**，它提供了一套完整的机制，允许一个程序访问另一个程序中的数据，同时还能保证被防数据的安全性。

内容提供器的用法一般有两种，一种是使用现有的内容提供器来读取和操作相应程序中的数据，另一种是创建自己的内容提供器给我们程序的数据提供外部访问接口。

---

## 访问其他程序中的数据

当一个应用程序通过内容提供器对其数据提供了外部访问接口，任何其他的应用程序就都可以对这部分数据进行访问。

#### ContentResolver的基本用法

ContentResolver中的增删改查方法接收一个Uri参数，这个参数被称为内容URI。

**内容URI给内容提供器中的数据建立了唯一标识符，**它主要由两部分组成，**权限（authority）**和**路径（path）**。权限是用于对不同的应用程序做区分的，一般为了避免冲突，都会采用程序包名的方式来进行命名。比如某个程序的包名是com.example.app，那么该程序对应的权限就可以命名为com.example.app.provider。路径则是用于对同一应用程序中不同的表做区分的，通常都会添加到权限的后面。比如某个程序的数据库里存在两张表，table1和table2，这时就可以将路径分别命名为/table1和/table2，然后把权限和路径进行组合，内容URI就变成了com.example.app.provider/table1和com.example.app.provider/table2。不过，目前还很难辨认出这两个字符串就是两个内容URI，我们还需要在字符串的头部加上协议声明。因此，内容URI最标准的格式写法如下：

	content://com.example.app.provider/table1
	content://com.example.app.provider/table2

在得到了内容URI字符串之后，我们还需要将它解析成Uri对象才可以作为参数传入。只需要调用Uri.parse()方法，就可以将内容URI字符串解析成Uri对象。

	Uri uri=Uri.parse("content://com.example.app.provider/table1")

现在我们就可以使用这个Uri对象来操作table1表中的数据了。

查询数据：

	Cursor cursor = getContentResolver().query(uri, projection, selection, selectionArgs, sortOrder);

	if (cursor != null) {
		while (cursor.moveToNext()) {
			String column1 = cursor.getString(cursor.getColumnIndex("column1"));
			int column2 = cursor.getInt(cursor.getColumnIndex("column2"));
		}
		cursor.close();
	}

添加数据：

	ContentValues values = new ContentValues();
	values.put("column1","text");
	values.put("column2",1);
	getContentResolver().insert(uri, values);

更新数据：

	ContentValues values = new ContentValues();
	values.put("column1", "");
	getContentResolver().update(uri, values, "column1 = ? and column2 = ?", new String[] {"text", "1"});

删除数据：

	getContentResolver().delete(uri, "column2 = ?", new String[] { "1" });

#### 读取系统联系人

	Cursor cursor = null;
	try {
		// 查询联系人数据
		cursor = getContentResolver().query(ContactsContract.CommonDataKinds.Phone.CONTENT_URI, null, null, null, null);
		while (cursor.moveToNext()) {
			// 获取联系人姓名
			String displayName = cursor
					.getString(cursor.getColumnIndex(ContactsContract.CommonDataKinds.Phone.DISPLAY_NAME));
			// 获取联系人手机号
			String number = cursor.getString(cursor.getColumnIndex(ContactsContract.CommonDataKinds.Phone.NUMBER));
			contactsList.add(displayName + "\n" + number);
		}
	}catch(Exception e) {
		e.printStackTrace();
	}finally {
		if (cursor != null) {
			cursor.close();
		}
	}

声明权限

	<uses-permission android:name="android.permission.READ_CONTACTS" />

---

## 创建自己的内容提供器

#### 创建内容提供器的步骤

如果想要实现跨程序共享数据的功能，官方推荐的方式就是使用内容提供器，可以通过新建一个类去继承ContentProvider的方式来创建一个自己的内容提供器。ContentProvider类中有六个抽象方法，我们在使用子类继承它的时候，需要将这六个方法全部重写。

	public class MyProvider extends ContentProvider {
		@Override
		public boolean onCreate() {
			return false; 
		}	

		@Override
		public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder) {
			return null;
		}	

		@Override
		public Uri insert(Uri uri, ContentValues values) {
			return null;
		}	

		@Override
		public int update(Uri uri, ContentValues values, String selection, String[] selectionArgs) {
			return 0;
		}	

		@Override
		public int delete(Uri uri, String selection, String[] selectionArgs) {
			return 0;
		}	

		@Override
		public String getType(Uri uri) {
			return null;
		}
	}

- onCreate()

初始化内容提供器的时候调用。通常会在这里完成对数据库的创建和升级等操作，返回true表示内容提供器初始化成功，返回false则表示失败。注意，只有当存在ContentResolver尝试访问我们程序中的数据时，内容提供器才会被初始化。

- query()

从内容提供器中查询数据。使用uri参数来确定查询哪张表，projection参数用于确定查询哪些列，selection和selectionArgs参数用于约束查询哪些行，sortOrder参数用于对结果进行排序，查询的结果存放在Cursor对象中返回。

- insert()

向内容提供器中添加一条数据。使用uri参数来确定要添加到的表，待添加的数据保存在values参数中。添加完成后，返回一个用于表示这条新记录的URI。

- update()

更新内容提供器中已有的数据。使用uri参数来确定更新哪一张表中的数据，新数据保存在values参数中，selection和selectionArgs参数用于约束更新哪些行，受影响的行数将作为返回值返回。

- delete()

从内容提供器中删除数据。使用uri参数来确定删除哪一张表中的数据，selection和selectionArgs参数用于约束删除哪些行，被删除的行数将作为返回值返回。

- getType()

根据传入的内容URI来返回相应的MIME类型。可以看到，几乎每一个方法都会带有Uri这个参数，这个参数也正是调用ContentResolver的增删改查方法时传递过来的。而现在，我们需要对传入的Uri参数进行解析，从中分析出调用方期望访问的表和数据。

##### 内容URI的格式

内容URI的格式主要有两种，**以路径结尾**就表示期望访问该表中所有的数据，**以id结尾**就表示期望访问该表中拥有相应id的数据。

content://com.example.app.provider/table1

表示调用方期望访问的是com.example.app这个应用的table1表中的数据。

content://com.example.app.provider/table1/1

表示调用方期望访问的是com.example.app这个应用的table1表中id为1的数据。

##### 通配符

我们还可以使用通配符的方式来分别匹配这两种格式的内容URI，规则如下。

1.*：表示匹配任意长度的任意字符

2.#：表示匹配任意长度的数字：

	content://com.example.app.provider/*    能够匹配任意表的内容URI

	content://com.example.app.provider/table1/#    能够匹配table1表中任意一行数据的内容

##### UriMatcher

借助UriMatcher这个类就可以轻松地实现匹配内容URI的功能。UriMatcher中提供了一个**addURI()**方法，这个方法接收三个参数，可以分别把**权限、路径和一个自定义代码**传进去。这样，**当调用UriMatcher的match()方法时，就可以将一个Uri对象传入，返回值是某个能够匹配这个Uri对象所对应的自定义代码，利用这个代码，我们就可以判断出调用方期望访问的是哪张表中的数据了**。

	public class MyProvider extends ContentProvider {

		public static final int TABLE1_DIR = 0;
		public static final int TABLE1_ITEM = 1;
		public static final int TABLE2_DIR = 2;
		public static final int TABLE2_ITEM = 3;
		private static UriMatcher uriMatcher;

		static {
			uriMatcher = new UriMatcher(UriMatcher.NO_MATCH);
			uriMatcher.addURI("com.example.app.provider", "table1", TABLE1_DIR);
			uriMatcher.addURI("com.example.app.provider ", "table1/#", TABLE1_ITEM);
			uriMatcher.addURI("com.example.app.provider ", "table2", TABLE2_ITEM);
			uriMatcher.addURI("com.example.app.provider ", "table2/#", TABLE2_ITEM);
		}……

		@Override
		public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder) {
			switch (uriMatcher.match(uri)) {
				case TABLE1_DIR:
				// 查询table1表中的所有数据
					break;
				case TABLE1_ITEM:
				// 查询table1表中的单条数据
					break;
				case TABLE2_DIR:
				// 查询table2表中的所有数据
					break;
				case TABLE2_ITEM:
				// 查询table2表中的单条数据
					break;
				default:
					break;
			}
		……
		}……
	}

可以看到，MyProvider中新增了四个整型常量，其中TABLE1_DIR表示访问table1表中的所有数据，TABLE1_ITEM表示访问table1表中的单条数据，TABLE2_DIR表示访问table2表中的所有数据，TABLE2_ITEM表示访问table2表中的单条数据。接着在静态代码块里我们创建了UriMatcher的实例，并调用addURI()方法，将期望匹配的内容URI格式传递进去，注意这里传入的路径参数是可以使用通配符的。然后当query()方法被调用的时候，就会通过UriMatcher的match()方法对传入的Uri对象进行匹配，如果发现UriMatcher中某个内容URI格式成功匹配了该Uri对象，则会返回相应的自定义代码，然后我们就可以判断出调用方期望访问的到底是什么数据了。

##### getType()方法

getType()方法是所有的内容提供器都必须提供的一个方法，用于获取Uri对象所对应的MIME类型。一个内容URI所对应的MIME 字符串主要由三部分组分，Android 对这三个部分做了如下格式规定。

1. 必须以 vnd 开头。
2. 如果内容 URI 以路径结尾，则后接 android.cursor.dir/，如果内容 URI 以 id 结尾， 则后接 android.cursor.item/。
3. 最后接上 vnd.<authority>.<path>。

所以
	content://com.example.app.provider/table1 

这个内容URI，它所对应的MIME类型就可以写成：

	vnd.android.cursor.dir/vnd.com.example.app.provider.table1

	content://com.example.app.provider/table1/1 

这个内容URI，它所对应的MIME类型就可以写成：

	vnd.android.cursor.item/vnd.com.example.app.provider.table1

## 实现跨程序数据共享

	public class DatabaseProvider extends ContentProvider {

		public static final int BOOK_DIR = 0;
		public static final int BOOK_ITEM = 1;
		public static final int CATEGORY_DIR = 2;
		public static final int CATEGORY_ITEM = 3;
		public static final String AUTHORITY = "com.example.databasetest.provider";
		private static UriMatcher uriMatcher;
		private MyDatabaseHelper dbHelper;

		static {
			uriMatcher = new UriMatcher(UriMatcher.NO_MATCH);
			uriMatcher.addURI(AUTHORITY, "book", BOOK_DIR);
			uriMatcher.addURI(AUTHORITY, "book/#", BOOK_ITEM);
			uriMatcher.addURI(AUTHORITY, "category", CATEGORY_DIR);
			uriMatcher.addURI(AUTHORITY, "category/#", CATEGORY_ITEM);
		}	

		@Override
		public boolean onCreate() {
			dbHelper = new MyDatabaseHelper(getContext(), "BookStore.db", null, 2);
			return true;
		}	

		@Override
		public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder) {
			// 查询数据
			SQLiteDatabase db = dbHelper.getReadableDatabase();
			Cursor cursor = null;
			switch (uriMatcher.match(uri)) {
			case BOOK_DIR:
				cursor = db.query("Book", projection, selection, selectionArgs, null, null, sortOrder);
				break;
			case BOOK_ITEM:
				String bookId = uri.getPathSegments().get(1);
				cursor = db.query("Book", projection, "id = ?", new String[] { bookId }, null, null, sortOrder);
				break;
			case CATEGORY_DIR:
				cursor = db.query("Category", projection, selection, selectionArgs, null, null, sortOrder);
				break;
			case CATEGORY_ITEM:
				String categoryId = uri.getPathSegments().get(1);
				cursor = db.query("Category", projection, "id = ?", new String[] { categoryId }, null, null, sortOrder);
				break;
			default:
				break;
			}
			return cursor;
		}	

		@Override
		public Uri insert(Uri uri, ContentValues values) {
			// 添加数据
			SQLiteDatabase db = dbHelper.getWritableDatabase();
			Uri uriReturn = null;
			switch (uriMatcher.match(uri)) {
			case BOOK_DIR:
			case BOOK_ITEM:
				long newBookId = db.insert("Book", null, values);
				uriReturn = Uri.parse("content://" + AUTHORITY + "/book/" + newBookId);
				break;
			case CATEGORY_DIR:
			case CATEGORY_ITEM:
				long newCategoryId = db.insert("Category", null, values);
				uriReturn = Uri.parse("content://" + AUTHORITY + "/category/" + newCategoryId);
				break;
			default:
				break;
			}
			return uriReturn;
		}	

		@Override
		public int update(Uri uri, ContentValues values, String selection, String[] selectionArgs) {
			// 更新数据
			SQLiteDatabase db = dbHelper.getWritableDatabase();
			int updatedRows = 0;
			switch (uriMatcher.match(uri)) {
			case BOOK_DIR:
				updatedRows = db.update("Book", values, selection, selectionArgs);
				break;
			case BOOK_ITEM:
				String bookId = uri.getPathSegments().get(1);
				updatedRows = db.update("Book", values, "id = ?", new String[] { bookId });
				break;
			case CATEGORY_DIR:
				updatedRows = db.update("Category", values, selection, selectionArgs);
				break;
			case CATEGORY_ITEM:
				String categoryId = uri.getPathSegments().get(1);
				updatedRows = db.update("Category", values, "id = ?", new String[] { categoryId });
				break;
			default:
				break;
			}
			return updatedRows;
		}	

		@Override
		public int delete(Uri uri, String selection, String[] selectionArgs) {
			// 删除数据
			SQLiteDatabase db = dbHelper.getWritableDatabase();
			int deletedRows = 0;
			switch (uriMatcher.match(uri)) {
			case BOOK_DIR:
				deletedRows = db.delete("Book", selection, selectionArgs);
				break;
			case BOOK_ITEM:
				String bookId = uri.getPathSegments().get(1);
				deletedRows = db.delete("Book", "id = ?", new String[] { bookId });
				break;
			case CATEGORY_DIR:
				deletedRows = db.delete("Category", selection, selectionArgs);
				break;
			case CATEGORY_ITEM:
				String categoryId = uri.getPathSegments().get(1);
				deletedRows = db.delete("Category", "id = ?", new String[] { categoryId });
				break;
			default:
				break;
			}
			return deletedRows;
		}	

		@Override
		public String getType(Uri uri) {
			switch (uriMatcher.match(uri)) {
			case BOOK_DIR:
				return "vnd.android.cursor.dir/vnd.com.example.databasetest. provider.book";
			case BOOK_ITEM:
				return "vnd.android.cursor.item/vnd.com.example.databasetest. provider.book";
			case CATEGORY_DIR:
				return "vnd.android.cursor.dir/vnd.com.example.databasetest. provider.category";
			case CATEGORY_ITEM:
				return "vnd.android.cursor.item/vnd.com.example.databasetest. provider.category";
			}
			return null;
		}
	}

首先在类的一开始，同样是定义了四个常量，分别用于表示访问Book表中的所有数据、访问Book表中的单条数据、访问Category表中的所有数据和访问Category表中的单条数据。然后在静态代码块里对UriMatcher进行了初始化操作，将期望匹配的几种URI格式添加了进去。

onCreate()方法创建了一个MyDatabaseHelper的实例，然后返回true表示内容提供器初始化成功，这时数据库就已经完成了创建或升级操作。

query()方法，在这个方法中先获取到了SQLiteDatabase的实例，然后根据传入的Uri参数判断出用户想要访问哪张表，再调用SQLiteDatabase的query()进行查询，并将Cursor对象返回就好了。注意当访问单条数据的时候有一个细节，这里调用了Uri对象的getPathSegments()方法，它会将内容URI权限之后的部分以“/”符号进行分割，并把分割后的结果放入到一个字符串列表中，那这个列表的第0个位置存放的就是路径，第1个位置存放的就是id了。得到了id之后，再通过selection和selectionArgs参数进行约束，就实现了查询单条数据的功能。

将内容提供器在AndroidManifest.xml文件中注册。

	<provider 
		android:name="com.example.databasetest.DatabaseProvider" 
		android:authorities="com.example.databasetest.provider" 
		Android:exported="true">
	</provider>

#### 测试访问刚才的程序

添加数据

	Uri uri = Uri.parse("content://com.example.databasetest.provider/book");
	ContentValues values = new ContentValues();
	values.put("name", "A Clash of Kings");
	values.put("author", "George Martin");
	values.put("pages", 1040);
	values.put("price", 22.85);
	Uri newUri = getContentResolver().insert(uri, values);
	newId = newUri.getPathSegments().get(1);

查询数据

	Uri uri = Uri.parse("content://com.example.databasetest.provider/book");
	Cursor cursor = getContentResolver().query(uri, null, null,null, null);
	if(cursor!=null) {
		while (cursor.moveToNext()) {
			String name = cursor.getString(cursor.getColumnIndex("name"));
			String author = cursor.getString(cursor.getColumnIndex("author"));
			int pages = cursor.getInt(cursor.getColumnIndex("pages"));
			double price = cursor.getDouble(cursor.getColumnIndex("price"));
		}
	}

更新数据

	Uri uri = Uri.parse("content://com.example.databasetest. provider/book/" + newId);
	ContentValues values = new ContentValues();
	values.put("name", "A Storm of Swords");
	values.put("pages", 1216);
	values.put("price", 24.05);
	getContentResolver().update(uri, values, null, null);

删除数据

	Uri uri = Uri.parse("content://com.example.databasetest. provider/book/" + newId);
	getContentResolver().delete(uri, null, null);
