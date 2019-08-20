[TOC]

# 创建布局文件
>File->XML

<div align="center">
<img src="https://github.com/yangang123/yangang123.github.io/raw/master/3-android/resourcexml_create.png" height="720" width="500" > 
</div>

# 布局基础知识

## 添加button
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <Button
        android:id="@+id/button1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="打开"/>

    <Button
        android:id="@+id/button2"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="关闭"/>

</LinearLayout>
```

## button属性修改
>android:id             : button的id
>android:layout_width   : button的宽度，可以是wrap_content，或者match_parent
>android:layout_height  : button的宽度，可以是wrap_content，或者match_parent
>android:text           : button的字符
```xml
 <Button
        android:id="@+id/button1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="打开"/>
```


## 线性布局中添加水平属性
>android中最常用的布局方式的是LinearLayout, 可以设置android:orientation属性是horizontal
```xml
<LinearLayout 
    android:orientation="horizontal"> 
</LinearLayout>
```
<div align="center">
<img src="https://github.com/yangang123/yangang123.github.io/raw/master/3-android/resourceLinearLayout_horizontal.png" height="720" width="500" > 
</div>

## 线性布局中添加垂直属性
>android中最常用的布局方式的是LinearLayout, 可以设置android:orientation属性是vertical
```xml
<LinearLayout 
    android:orientation="vertical"> 
</LinearLayout>
```

<div align="center">
<img src="https://github.com/yangang123/yangang123.github.io/raw/master/3-android/resourceLinearLayout_vertical.png" height="720" width="500" > 
</div>
