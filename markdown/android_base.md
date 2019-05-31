[TOC]

# ���������ļ�
>File->XML

<div align="center">
<img src="https://github.com/yangang123/picture/raw/master/Android/xml_create.png" height="720" width="500" > 
</div>

# ���ֻ���֪ʶ

## ���button
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
        android:text="��"/>

    <Button
        android:id="@+id/button2"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="�ر�"/>

</LinearLayout>
```

## button�����޸�
>android:id             : button��id
>android:layout_width   : button�Ŀ�ȣ�������wrap_content������match_parent
>android:layout_height  : button�Ŀ�ȣ�������wrap_content������match_parent
>android:text           : button���ַ�
```xml
 <Button
        android:id="@+id/button1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="��"/>
```


## ���Բ��������ˮƽ����
>android����õĲ��ַ�ʽ����LinearLayout, ��������android:orientation������horizontal
```xml
<LinearLayout 
    android:orientation="horizontal"> 
</LinearLayout>
```
<div align="center">
<img src="https://github.com/yangang123/picture/raw/master/Android/LinearLayout_horizontal.png" height="720" width="500" > 
</div>

## ���Բ�������Ӵ�ֱ����
>android����õĲ��ַ�ʽ����LinearLayout, ��������android:orientation������vertical
```xml
<LinearLayout 
    android:orientation="vertical"> 
</LinearLayout>
```

<div align="center">
<img src="https://github.com/yangang123/picture/raw/master/Android/LinearLayout_vertical.png" height="720" width="500" > 
</div>
