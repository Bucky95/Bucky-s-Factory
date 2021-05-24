**1.时间戳的添加**

首先对noteslist_item.xml文件进行修改
noteslist_item.xml布局文件是笔记每个条目的布局，新增一个TextView,用于显示时间戳，修改后的代码如下:

```xml
<RelativeLayout android:layout_height="match_parent"
    android:layout_width="match_parent"
    xmlns:android="http://schemas.android.com/apk/res/android">
    <TextView xmlns:android="http://schemas.android.com/apk/res/android"
        android:id="@android:id/text1"
        android:layout_width="match_parent"
        android:layout_height="?android:attr/listPreferredItemHeight"
        android:textAppearance="?android:attr/textAppearanceLarge"
        android:gravity="center_vertical"
        android:paddingLeft="5dip"
        android:singleLine="true"
    />
    <TextView
        android:id="@+id/text2"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:paddingLeft="5dip"
        android:singleLine="true"
        android:gravity="center_vertical"
    />
</RelativeLayout>
```



接着找到NotePadProvider和NoteEditor，修改NotePadProvider中的insert方法，以及在NoteEditor中的updateNote方法，我们需要修改这个方法中的时间戳格式
NotePadProvider中的insert方法增加如下代码：

```java
Long now = Long.valueOf(System.currentTimeMillis());
//修改 需要将毫秒数转换为时间的形式yy.MM.dd HH:mm:ss
Date date = new Date(now);
SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yy-MM-dd HH:mm:ss");
String dateFormat = simpleDateFormat.format(date);//转换为yy.MM.dd HH:mm:ss形式的时间
// If the values map doesn't contain the creation date, sets the value to the current time.
if (values.containsKey(NotePad.Notes.COLUMN_NAME_CREATE_DATE) == false) {
    values.put(NotePad.Notes.COLUMN_NAME_CREATE_DATE, dateFormat);
}

// If the values map doesn't contain the modification date, sets the value to the current
// time.
if (values.containsKey(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE) == false) {
    values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, dateFormat);
}
```

NoteEditor中的updateNote方法修改后如下：

```java
long now = System.currentTimeMillis();
Date date = new Date(now);
SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yy-MM-dd HH:mm:ss");
String dateFormat = simpleDateFormat.format(date);
values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, dateFormat);
```

接着在在NotesList中，列的投影PROJECTION为:

```java
private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
    };
```

只投影出了ID和每条笔记的标题，如果我们要加入时间戳，必须要将修改时间的列也投影出来.
所以我们将PROJECTION添加上修改时间，修改后的代码如下：

```java
private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,//添加修改时间
    };
```

PROJECTION只是定义了需要被取出来的数据列，而之后用Cursor进行数据库查询，再之后用Adapter进行装填。
在看完源码之后，Cursor不用变化，我们需要将显示列dataColumns和他们的viewIDs加入修改时间这一属性
原来的代码:

```java
String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE};
int[] viewIDs = { android.R.id.text1 };
```

加入修改时间后:

```java
String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE, NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE }//加入修改时间;
int[] viewIDs = { android.R.id.text1, R.id.text2 }//加入修改时间;
```

运行结果如下：

![image](https://github.com/Bucky95/Bucky-s-factory/tree-save/master/NotePad-master/images/image-20210517172152350.png)

