# qtchooserh����

qtchooser��1��qt�İ汾���ƹ���

yangang@ubuntu:/usr/lib/x86_64-linux-gnu/qtchooser$ qtchooser 
Usage:
  qtchooser { -l | -list-versions | -print-env }
  qtchooser -install [-f] [-local] <name> <path-to-qmake>
  qtchooser -run-tool=<tool name> [-qt=<Qt version>] [program arguments]
  <executable name> [-qt=<Qt version>] [program arguments]

Environment variables accepted:
 QTCHOOSER_RUNTOOL  name of the tool to be run (same as the -run-tool argument)
 QT_SELECT          version of Qt to be run (same as the -qt argument)

# QT_SELECT���������������Ҫ

1. QT_SELECT��default��ʱ�� 
������������Ӿ�����qmake�İ汾
```
yangang@ubuntu:/usr/lib/x86_64-linux-gnu/qt-default/qtchooser$ ll default.conf 
lrwxrwxrwx 1 root root 46 Apr 20 01:18 default.conf -> /usr/share/qtchooser/qt4-x86_64-linux-gnu.conf
``` 

2. QT_SELECT��"qt4"��ʱ�� 
������������Ӿ�����qmake�İ汾
```
yangang@ubuntu:/usr/lib/x86_64-linux-gnu/qtchooser$ ll qt4.conf 
lrwxrwxrwx 1 root root 46 Apr 20 01:28 qt4.conf -> /usr/share/qtchooser/qt4-x86_64-linux-gnu.conf
``` 


3. QT_SELECT��"qt5"��ʱ�� 
������������Ӿ�����qmake�İ汾
```
yangang@ubuntu:/usr/lib/x86_64-linux-gnu/qtchooser$ ll qt5.conf 
lrwxrwxrwx 1 root root 46 Apr 20 01:47 qt5.conf -> /usr/share/qtchooser/qt5-x86_64-linux-gnu.conf
``` 


�޸ķ�����
1. ͨ��export �޸�QT_SELECT 
2. ���û�������ָ�����ļ�������
