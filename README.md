NotePad期中作业（基于原应用的扩展）
================================

本实验将基于NotePad应用做功能扩展

新增功能
-------
* NotesList中显示的条目下方新增时间
* 便签查询（标题查询）
* UI美化
* 更换背景颜色
* 便签导出到本地
* 便签按照某种方式排序（提供三种，创建时间、修改时间、颜色）

新增功能的解析以及截图实例
-------------------------
* NotesList中增加时间显示

在 noteslist_item.xml 增加时间显示的条目，将时间显示在标题下方。

### 相关源码
```xml
<!-- 时间戳TextView -->
    <TextView
        android:id="@+id/text_time"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textAppearance="?android:attr/textAppearanceSmall"
        android:paddingLeft="5dip"
        android:textColor="@color/colorBlack" />
```

使用 COLUMN_NAME_MODIFICATION_TIME 用来存储时间，在 PROJECTION 中定义，并将其导入数据库
### 相关源码(PROJECTION中添加时间，viewIDs添加时间)
```java
/**
     * The columns needed by the cursor adapter
     */
    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1

            //扩展 显示时间 颜色
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,
            NotePad.Notes.COLUMN_NAME_BACK_COLOR
    };
```
```java
private String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE ,  NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE } ;
    private int[] viewIDs = { android.R.id.text1 , R.id.text_time };
```

在NotePadProvider中的insert方法和NoteEditor中的updateNote方法中加入时间的转化，前者是创建的时间，后者是修改的时间

## 截图展示
![PatrickStaR](https://github.com/WanglePaiDaXing/NotePad/blob/master/image/%E6%8D%95%E8%8E%B7.PNG)  

* 查询功能
实现查询大纲以及实时查询
### 相关源码（创建一个搜索类NoteSearch,对文本变化进行监听）
```java
package com.example.android.notepad;

import android.app.ListActivity;
import android.content.ContentUris;
import android.content.Intent;
import android.database.Cursor;
import android.net.Uri;
import android.os.Bundle;
import android.view.View;
import android.widget.ListView;
import android.widget.SearchView;
import android.widget.SimpleCursorAdapter;

public class NoteSearch extends ListActivity implements SearchView.OnQueryTextListener {
    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID,//0
            NotePad.Notes.COLUMN_NAME_TITLE,//1
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,//2
            NotePad.Notes.COLUMN_NAME_BACK_COLOR,
    };
    @Override
    protected  void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.note_search_list);
        Intent intent = getIntent();
        if(intent.getData() == null) {
            intent.setData(NotePad.Notes.CONTENT_URI);
        }
        SearchView searchView = (SearchView)findViewById(R.id.search_view);
        //为查询文本框注册监听器
        searchView.setOnQueryTextListener(this);
    }
    @Override
    public  boolean onQueryTextSubmit(String query) {
        return false;
    }
    @Override
    public boolean onQueryTextChange(String newText) {
        String selection = NotePad.Notes.COLUMN_NAME_TITLE + " Like ? ";
        String[] selectionArgs = {"%" + newText + "%"};
        Cursor cursor = managedQuery(
                getIntent().getData(),//Use the default content URI for the provider
                PROJECTION,//Return the note ID and title for each note.and modifcation data
                selection,//条件左边
                selectionArgs,//条件右边
                NotePad.Notes.DEFAULT_SORT_ORDER//Use the default sort order
        );
        String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE , NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE };
        int[] viewIDs = { android.R.id.text1 , R.id.text_time };
        MyCursorAdapter adapter = new MyCursorAdapter(
                this,
                R.layout.noteslist_item,
                cursor,
                dataColumns,
                viewIDs
        );
        setListAdapter(adapter);
        return true;
    }
    @Override
    protected void onListItemClick(ListView listView, View v, int position, long id) {
        //Constructs a new URI from the incoming URI and the row ID
        Uri uri = ContentUris.withAppendedId(getIntent().getData() , id);
        //Gets the action from the incoming Intent
        String action = getIntent().getAction();
        //Handles requests for note data
        if (Intent.ACTION_PICK.equals(action) || Intent.ACTION_GET_CONTENT.equals(action)) {
            //Sets the result to return to the component that called this Activity.The result contains the new URI
            setResult(RESULT_OK , new Intent().setData(uri));
        } else {
            //Sends out an Intent to start an Activity that can handle ACTION_EDIT. The Intent's data is the note ID URI. The effect is to call NoteEdit.
            startActivity(new Intent(Intent.ACTION_EDIT, uri));
        }
    }
}

```
在AndroidManifest.xml中注册NoteSearch
## 截图展示
##### 搜索主界面，会显示所有便签
![PatrickStaR](https://github.com/WanglePaiDaXing/NotePad/blob/master/image/%E6%90%9C%E7%B4%A2.PNG)  
##### 实时搜索，输入啥搜索啥
![PatrickStaR](https://github.com/WanglePaiDaXing/NotePad/blob/master/image/%E6%90%9C%E7%B4%A2%E7%BB%93%E6%9E%9C.PNG)  

* UI美化
### 相关源码
先将NotesList换成白色背景
```xml
android:theme="@android:style/Theme.Holo.Light"
```
添加颜色字段，存储每一条NoteList的颜色信息
```java
public static final String COLUMN_NAME_BACK_COLOR = "color";
```
这样在每次读取时都会显示上一次保存的信息

## 截图展示
#### 变色
![PatrickStaR](https://github.com/WanglePaiDaXing/NotePad/blob/master/image/%E5%8F%98%E8%89%B2.PNG)  
#### 变色结果
![PatrickStaR](https://github.com/WanglePaiDaXing/NotePad/blob/master/image/%E5%8F%98%E8%89%B2%E7%BB%93%E6%9E%9C.PNG)  

* 导出到本地
### 相关源码
需要进行权限设置
```xml
 <!-- 在SD卡中创建与删除文件权限 -->
    <uses-permission android:name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS"/>
    <!-- 向SD卡写入数据权限 -->
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
```
## 截图展示
#### 导出
![PatrickStaR](https://github.com/WanglePaiDaXing/NotePad/blob/master/image/%E5%AF%BC%E5%87%BA.PNG)  
#### 导出结果
![PatrickStaR](https://github.com/WanglePaiDaXing/NotePad/blob/master/image/%E5%AF%BC%E5%87%BA%E7%BB%93%E6%9E%9C.PNG)  

* 便签的排序（颜色排序、创建时间排序、修改时间排序）
### 相关源码
菜单文件list_options_menu.xml进行修改添加
```xml
<item
    android:id="@+id/menu_sort"
    android:title="@string/menu_sort"
    android:icon="@android:drawable/ic_menu_sort_by_size"
    android:showAsAction="always" >
    <menu>
        <item
            android:id="@+id/menu_sort1"
            android:title="@string/menu_sort1"/>
        <item
            android:id="@+id/menu_sort2"
            android:title="@string/menu_sort2"/>
        <item
            android:id="@+id/menu_sort3"
            android:title="@string/menu_sort3"/>
        </menu>
    </item>
```
菜单的选择项也要台南佳，NoteList中
```java
//创建时间排序
    case R.id.menu_sort1:
        cursor = managedQuery(
                getIntent().getData(),            
                PROJECTION,                      
                null,                          
                null,                          
                NotePad.Notes._ID 
                );
        adapter = new MyCursorAdapter(
                this,
                R.layout.noteslist_item,
                cursor,
                dataColumns,
                viewIDs
        );
        setListAdapter(adapter);
        return true;
 //修改时间排序
    case R.id.menu_sort2:
        cursor = managedQuery(
                getIntent().getData(),          
                PROJECTION,                      
                null,                            
                null,                       
                NotePad.Notes.DEFAULT_SORT_ORDER 
        );
        adapter = new MyCursorAdapter(
                this,
                R.layout.noteslist_item,
                cursor,
                dataColumns,
                viewIDs
        );
        setListAdapter(adapter);
        return true;
    //颜色排序
    case R.id.menu_sort3:
        cursor = managedQuery(
                getIntent().getData(),
                PROJECTION,      
                null,       
                null,       
                NotePad.Notes.COLUMN_NAME_BACK_COLOR
                );
        adapter = new MyCursorAdapter(
                this,
                R.layout.noteslist_item,
                cursor,
                dataColumns,
                viewIDs
                );
        setListAdapter(adapter);
        return true;
```
## 截图展示
#### 创建时间排序
![PatrickStaR](https://github.com/WanglePaiDaXing/NotePad/blob/master/image/%E6%97%B6%E9%97%B4%E6%8E%92%E5%BA%8F.PNG)  
#### 修改时间排序
![PatrickStaR](https://github.com/WanglePaiDaXing/NotePad/blob/master/image/%E4%BF%AE%E6%94%B9%E6%97%B6%E9%97%B4%E6%8E%92%E5%BA%8F.PNG)  
#### 颜色排序
![PatrickStaR](https://github.com/WanglePaiDaXing/NotePad/blob/master/image/%E9%A2%9C%E8%89%B2%E6%8E%92%E5%BA%8F.PNG)  
* 其他小功能
#### 长按菜单
![PatrickStaR](https://github.com/WanglePaiDaXing/NotePad/blob/master/image/%E9%95%BF%E6%8C%89%E8%8F%9C%E5%8D%95.PNG)
