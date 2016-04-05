---
layout:     post
title:      "实现跨程序共享"
subtitle:   ""
date:       2016-04-01
author:     "WunWun"
header-img: "img/android-proud.jpg"
tags:
    - Android
    - 学习笔记
---

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

insert()方法，先获取到了SQLiteDatabase的实例，然后根据传入的Uri参数判断出用户想要往哪张表里添加数据，再调用SQLiteDatabase的insert()方法进行添加就可以了。注意insert()方法要求返回一个能够表示这条新增数据的URI，所以我们还需要调用Uri.parse()方法来将一个内容URI解析成Uri对象，当然这个内容URI是以新增数据的id结尾的。

update()方法了，先获取SQLiteDatabase的实例，然后根据传入的Uri参数判断出用户想要更新哪张表里的数据，再调用SQLiteDatabase的update()方法进行更新就好了，受影响的行数将作为返回值返回。

delete()方法，先获取到SQLiteDatabase的实例，然后根据传入的Uri参数判断出用户想要删除哪张表里的数据，再调用SQLiteDatabase的delete()方法进行删除就好了，被删除的行数将作为返回值返回。

将内容提供器在AndroidManifest.xml文件中注册。

	<provider 
		android:name="com.example.databasetest.DatabaseProvider" 
		android:authorities="com.example.databasetest.provider" 
		Android:exported="true">
	</provider>

## 访问程序共享的数据

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
