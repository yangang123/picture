
linux下的 qtcreatro是qt的开发工具IDE

## qt5.7和qt5.11工具安装

## qt5.7.1安装
我们看出qt5.7.1默认安装了很多的插件
<div align="center">
<img src="https://github.com/yangang123/yangang123.github.io/raw/master/4-px4/resource/qt5.7.1_install.png" height="720" width="1280" > 
</div>

## qt5.11.3 安装第一步选择路径
1. 出现默认路径
<div align="center">
<img src="https://github.com/yangang123/yangang123.github.io/raw/master/4-px4/resource/qt5.11.3_install_1.png" height="720" width="1280" > 
</div>
  
2. 选择路径
<div align="center">
<img src="https://github.com/yangang123/yangang123.github.io/raw/master/4-px4/resource/qt5.11.3_install_2.png" height="720" width="1280" > 
</div>

## qt5.11.3 安装选择模块
1. qt5.11.3默认的安装基本没有模块，
<div align="center">
<img src="https://github.com/yangang123/yangang123.github.io/raw/master/4-px4/resource/qt5.11.3_install_modules.png" height="720" width="1280" > 
</div>

2. qt5.11.3我们选择“Destop gcc 64-bit”和"Qt Charts"
<div align="center">
<img src="https://github.com/yangang123/yangang123.github.io/raw/master/4-px4/resource/qt5.11.3_install_modules_1.png" height="720" width="1280" > 
</div>

## 查看qtcreator的kit内容
<div align="center">
<img src="https://github.com/yangang123/yangang123.github.io/raw/master/4-px4/resource/qt5.11.3_kit.png" height="720" width="1280" > 
</div>

## 4-px4/resource编译完成
<div align="center">
<img src="https://github.com/yangang123/yangang123.github.io/raw/master/4-px4/resource/qt5.11.3_qgc_build_end.png" height="720" width="1280" > 
</div>

# 查看Qt Chart模块安装状态


## 查看qt的环境变量
```
yangang@yangang-ubuntu:~/work/github_proj/picture/4-px4/resource$ qmake -query
QT_SYSROOT:
QT_INSTALL_PREFIX:/opt/Qt5.11.3/5.11.3/gcc_64
QT_INSTALL_ARCHDATA:/opt/Qt5.11.3/5.11.3/gcc_64
QT_INSTALL_DATA:/opt/Qt5.11.3/5.11.3/gcc_64
QT_INSTALL_DOCS:/opt/Qt5.11.3/Docs/Qt-5.11.3
QT_INSTALL_HEADERS:/opt/Qt5.11.3/5.11.3/gcc_64/include
QT_INSTALL_LIBS:/opt/Qt5.11.3/5.11.3/gcc_64/lib
QT_INSTALL_LIBEXECS:/opt/Qt5.11.3/5.11.3/gcc_64/libexec
QT_INSTALL_BINS:/opt/Qt5.11.3/5.11.3/gcc_64/bin
QT_INSTALL_TESTS:/opt/Qt5.11.3/5.11.3/gcc_64/tests
QT_INSTALL_PLUGINS:/opt/Qt5.11.3/5.11.3/gcc_64/plugins
QT_INSTALL_IMPORTS:/opt/Qt5.11.3/5.11.3/gcc_64/imports
QT_INSTALL_QML:/opt/Qt5.11.3/5.11.3/gcc_64/qml
QT_INSTALL_TRANSLATIONS:/opt/Qt5.11.3/5.11.3/gcc_64/translations
QT_INSTALL_CONFIGURATION:/opt/Qt5.11.3/5.11.3/gcc_64
QT_INSTALL_EXAMPLES:/opt/Qt5.11.3/Examples/Qt-5.11.3
QT_INSTALL_DEMOS:/opt/Qt5.11.3/Examples/Qt-5.11.3
QT_HOST_PREFIX:/opt/Qt5.11.3/5.11.3/gcc_64
QT_HOST_DATA:/opt/Qt5.11.3/5.11.3/gcc_64
QT_HOST_BINS:/opt/Qt5.11.3/5.11.3/gcc_64/bin
QT_HOST_LIBS:/opt/Qt5.11.3/5.11.3/gcc_64/lib
QMAKE_SPEC:linux-g++
QMAKE_XSPEC:linux-g++
QMAKE_VERSION:3.1
QT_VERSION:5.11.3
```
## 查看qt lib中的Charts模块是否安装
qt的模块化也是非常强的，各个模块都是的动态库，我们通过qt的库也可以了解到qt的整个架构模块化很好的体现
```
yangang@yangang-ubuntu:/opt/Qt5.11.3/5.11.3/gcc_64/lib$ ll libQt5Charts.*
-rw-rw-r-- 1 root root     728 4月  26 14:10 libQt5Charts.la
-rw-rw-r-- 1 root root    1140 4月  26 14:10 libQt5Charts.prl
lrwxrwxrwx 1 root root      22 4月  26 14:10 libQt5Charts.so -> libQt5Charts.so.5.11.3*
lrwxrwxrwx 1 root root      22 4月  26 14:10 libQt5Charts.so.5 -> libQt5Charts.so.5.11.3*
lrwxrwxrwx 1 root root      22 4月  26 14:10 libQt5Charts.so.5.11 -> libQt5Charts.so.5.11.3*
-rwxrwxr-x 1 root root 2260640 4月  26 14:10 libQt5Charts.so.5.11.3*
```

