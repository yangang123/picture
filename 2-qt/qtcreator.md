qt版本介绍

# 简介
qt支持的windows, linux， windows等很多版本

# qt版本

<div align="center">
<p>  安装 </p> 
<img src="https://github.com/yangang123/picture/raw/master/2-qt/resource/qt_version.png" height="720" width="1280" > 
</div>


## mingw版本
Mingw是Minimalist GNUfor Windows的缩写，mingw项目主要为了解决如果在windows上运行unix的程序的项目
mingw继承了gcc, gdb, make 这些常用的unix上开发项目的工具。qt-opensource-windows-x86-mingw491_opengl-5.4.0.exe
这个版本就是基于mingw作为编译工具进行开发。

```
qt-opensource-windows-x86-mingw491_opengl-5.4.0.exe
调用关系：qt -> mingw -> windows 

优点: 在mingw上编译的程序可以在很便捷的移植到unix，有很好的移植性
缺点: windows下性能可能和unix环境下还是有差距的，毕竟unix程序和window中多增加了一层API调用
```
qt版本介绍

# 简介
qt支持的windows, linux， windows等很多版本

# qt版本

<div align="center">
<p>  安装 </p> 
<img src="https://github.com/yangang123/picture/raw/master/2-qt/resource/qt_version.png" height="720" width="1280" > 
</div>


## mingw版本
Mingw是Minimalist GNUfor Windows的缩写，mingw项目主要为了解决如果在windows上运行unix的程序的项目
mingw继承了gcc, gdb, make 这些常用的unix上开发项目的工具。qt-opensource-windows-x86-mingw491_opengl-5.4.0.exe
这个版本就是基于mingw作为编译工具进行开发。

```
qt-opensource-windows-x86-mingw491_opengl-5.4.0.exe
调用关系：qt -> mingw -> windows 

优点: 在mingw上编译的程序可以在很便捷的移植到unix，有很好的移植性
缺点: windows下性能可能和unix环境下还是有差距的，毕竟unix程序和window中多增加了一层API调用
```

## msv2013 
qt-msv2013版本就是qt利用mvs2013的编译器，调试器开发

```
qt-opensource-windows-x86-msvc2013_64-5.4.0.exe
调用关系：qt -> windows 

优点: 由于qt可以直接调用windows的API,所以此版本下编译的应用程序运行效率可能比mingw版本的要好一点，毕竟少调用了一层API
缺点: 由于使用msv2013的编译器进行开发，在c++语法上是否和gcc的c++语法是否一致，要么会导致程序放到unix编译报错的问题
```

##  linux
qt-linux版本是在linux下开发的，linux下的版本很单一，现在一般的CPU都是64位的，下载的时候选择64位的，如果少数人的电脑是
32位，选择32位版本的软件
```
qt-opensource-linux-x64-5.4.0.run
```

## 总结

1. 如果仅仅是学习qt,建议大家使用mingw版本的，尤其对于崇拜unix的开发环境的
2. 如果追求软件性能的，建议大家选择msv各个版本的qt环境




## msv2013 
qt-msv2013版本就是qt利用mvs2013的编译器，调试器开发

```
qt-opensource-windows-x86-msvc2013_64-5.4.0.exe
调用关系：qt -> windows 

优点: 由于qt可以直接调用windows的API,所以此版本下编译的应用程序运行效率可能比mingw版本的要好一点，毕竟少调用了一层API
缺点: 由于使用msv2013的编译器进行开发，在c++语法上是否和gcc的c++语法是否一致，要么会导致程序放到unix编译报错的问题
```

##  linux
qt-linux版本是在linux下开发的，linux下的版本很单一，现在一般的CPU都是64位的，下载的时候选择64位的，如果少数人的电脑是
32位，选择32位版本的软件
```
qt-opensource-linux-x64-5.4.0.run
```

## 总结

1. 如果仅仅是学习qt,建议大家使用mingw版本的，尤其对于崇拜unix的开发环境的
2. 如果追求软件性能的，建议大家选择msv各个版本的qt环境



