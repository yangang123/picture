经过几天对qt和qgroundcontrol的了解，掌握到qt版本的一些技巧。把大家学习qt会遇到的坑告诉大家。

1. qt安装一定要注意，qt的不同版本，在linux下安装默认工具是不同的，比如有5.3.2会默认选择gcc的工具链，
   qt5.7默认安装了qtcharts，但是qt5.11.3就不安装。更具不同的需要去选择不同的工具
2. 编译qgroundcontrol建议大家先用qmake进行编译，qtcreator的配置也是很闹心的
3. qmake -v 去查看qt版本，qtchooser可以很好的管理的不同版本的qmake,但是也是很麻烦的，
   建议新手直接用export PATH去配置环境变量，这样系统就是第一找到你的qmake路径，
4. qgroundcontrol中的.travis.yml中自动部署项目，有兴趣可以阅读，清楚的知道当前版本qgroundcontrol
  去那个版本的qmake去编译

# 编译qgroundcontrol3.2
``` 
# 下载qt5.7.1 同时设置环境变量

$ cd /opt
$ wget http://download.qt.io/archive/qt/5.7/5.7.1/qt-opensource-linux-x64-5.7.1.run
$ chmod +x qt-opensource-linux-x64-5.7.1.run
$ ./qt-opensource-linux-x64-5.7.1.run
$ export PATH=/opt/Qt5.7.1/5.7/gcc_64/bin:$PATH

# 下载qgroundcontrol仓库指定tag是3.2
$ git clone -b v3.2.0 https://github.com/mavlink/qgroundcontrol.git
$ cd qgroundcontrol

# 编译qgroundctrol,执行QGroundControl

$ mkdir build && cd build
$ qmake ../qgroundcontrol.pro 
$ make -j4
$ ./release/QGroundControl 

```

# 编译qgroundcontrol3.5

``` 
$ cd /opt
$ wget http://download.qt.io/archive/qt/5.11/5.11.3/qt-opensource-linux-x64-5.11.3.run
$ chmod +x qt-opensource-linux-x64-5.11.3.run
$ ./qt-opensource-linux-x64-5.11.3.run
$ export PATH=/opt/Qt5.11.3/5.11/gcc_64/bin:$PATH

# 下载qgroundcontrol仓库指定tag是3.5
$ git clone -b v3.5.0 https://github.com/mavlink/qgroundcontrol.git
$ cd qgroundcontrol

# 编译qgroundctrol,执行QGroundControl

$ mkdir build && cd build
$ qmake ../qgroundcontrol.pro

Project MESSAGE: Qt version 5.11.3
Project MESSAGE: Linux build
Project MESSAGE: QGroundControl v3.5.0
Project MESSAGE: Release flavor
Project MESSAGE: Using Default QtLocation headers
Project MESSAGE: Using MAVLink dialect 'ardupilotmega'.
Project MESSAGE: Skipping support for Zeroconf (unsupported platform)
Project MESSAGE: Including support for video streaming
Project MESSAGE: This project is using private headers and will therefore be tied to this specific Qt module build version.
Project MESSAGE: Running this project against other versions of the Qt modules may crash at any arbitrary point.
Project MESSAGE: This is not a bug, but a result of using Qt internals. You have been warned!

$ make -j4
$ ./release/QGroundControl 

```

## 可能错误的错误:
```
Project MESSAGE: Qt version 5.11.3
Project MESSAGE: Linux build
Project MESSAGE: QGroundControl v3.5.0
Project MESSAGE: Release flavor
Project MESSAGE: Using Default QtLocation headers
Project MESSAGE: Using MAVLink dialect 'ardupilotmega'.
Project MESSAGE: Skipping support for Zeroconf (unsupported platform)
Project MESSAGE: Including support for video streaming
Project ERROR: Unknown module(s) in QT: charts
``` 
通过log输出，我们看到qt charts模块没有安装导致的。我们需要重新安装q-5.11.3
