# qtchooserh介绍

qtchooser是1个qt的版本控制工具

yangang@ubuntu:/usr/lib/x86_64-linux-gnu/qtchooser$ qtchooser 
Usage:
  qtchooser { -l | -list-versions | -print-env }
  qtchooser -install [-f] [-local] <name> <path-to-qmake>
  qtchooser -run-tool=<tool name> [-qt=<Qt version>] [program arguments]
  <executable name> [-qt=<Qt version>] [program arguments]

Environment variables accepted:
 QTCHOOSER_RUNTOOL  name of the tool to be run (same as the -run-tool argument)
 QT_SELECT          version of Qt to be run (same as the -qt argument)

# QT_SELECT这个环境变量很重要

1. QT_SELECT：default的时候 
下面这个软链接决定了qmake的版本
```
yangang@ubuntu:/usr/lib/x86_64-linux-gnu/qt-default/qtchooser$ ll default.conf 
lrwxrwxrwx 1 root root 46 Apr 20 01:18 default.conf -> /usr/share/qtchooser/qt4-x86_64-linux-gnu.conf
``` 

2. QT_SELECT："qt4"的时候 
下面这个软链接决定了qmake的版本
```
yangang@ubuntu:/usr/lib/x86_64-linux-gnu/qtchooser$ ll qt4.conf 
lrwxrwxrwx 1 root root 46 Apr 20 01:28 qt4.conf -> /usr/share/qtchooser/qt4-x86_64-linux-gnu.conf
``` 


3. QT_SELECT："qt5"的时候 
下面这个软链接决定了qmake的版本
```
yangang@ubuntu:/usr/lib/x86_64-linux-gnu/qtchooser$ ll qt5.conf 
lrwxrwxrwx 1 root root 46 Apr 20 01:47 qt5.conf -> /usr/share/qtchooser/qt5-x86_64-linux-gnu.conf
``` 


修改方法：
1. 通过export 修改QT_SELECT 
2. 设置环境变量指定的文件的内容
