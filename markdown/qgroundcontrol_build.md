qgroundcontrol3.2.0 

# 简介
qgroundcontrol3.2.0是px4开发地面站, 小编闲来无事，自己基于源码编译1个qground地面站。

# 安装编译

## 编译环境
ubuntu : 16.04
qt: 5.9.0
qgroundcontorl: 3.2.0

## 1. qt安装 
```
$ cd ~/
$ wget http://download.qt.io/official_releases/qt/5.9/5.9.0/qt-opensource-linux-x64-5.9.0.run
$ sudo chmod +x qt-opensource-linux-x64-5.9.0.run
$ ./qt-opensource-linux-x64-5.9.0.run
```

注意选择"Desktop_gcc_64-bit",否则会出现下面的问题

<div align="center">
<p>  安装 </p> 
<img src="https://github.com/yangang123/picture/raw/master/qgroundcontrol/qt_5.9.0_install.png" height="720" width="1280" > 
</div>

<div align="center">
<p>  出现没有kit的错误 </p> 
<img src="https://github.com/yangang123/picture/raw/master/qgroundcontrol/no_qmake_Install.PNG" height="720" width="1280" > 
</div>

卸载qt方式
```
yangang@ubuntu:~/work/tools/qt5.9.0$ ls
5.9             dist  Examples             Licenses         MaintenanceTool.dat  network.xml
components.xml  Docs  InstallationLog.txt  MaintenanceTool  MaintenanceTool.ini  Tools
yangang@ubuntu:~/work/tools/qt5.9.0$ ./MaintenanceTool
```
<!-- ![图片](./qgroundcontrol/qt_5.9.0_install.png)
![图片](./qgroundcontrol/no_qmake_Install.png) -->

## 2. 下载qgroundcontrol源码
版本切换，注意要是用git submodules进行同步子模块
```
$ git clone https://github.com/mavlink/qgroundcontrol.git
$ git checkout v3.2.0 
$ sudo git submodule  init
$ sudo git submodule  update  
```

## 3. 编译qgroundcontrol环境

<!-- ![图片](./qgroundcontrol/build.png) -->

<div align="center">
<p>  出现没有kit的错误 </p> 
<img src="https://github.com/yangang123/picture/raw/master/qgroundcontrol/build.png" height="720" width="1280" > 
</div>

1. 错误
   
Project ERROR: sdl development package not found

解决方法
```
yangang@ubuntu:~/work/tools/game-tools$ sudo apt-cache search "libSDL2*"
libsdl1.2-dev - Simple DirectMedia Layer development files
```
2. 错误2 
error: gstreamer-video-1.0 development package not found

解决方法：

切换qgroundcontrol到3.2.0, 就不会出现了

3. 编译完成

<div align="center">
<p>  编译完成 </p> 
<img src="https://github.com/yangang123/picture/raw/master/qgroundcontrol/build_end.png" height="720" width="1280" > 
</div>


4. 提醒
   
如果还出现其他问题，可以通过 https://github.com/mavlink/qgroundcontrol.git的
issue中找到一些解决方法

## 运行qgroundcontrol

<div align="center">
<p>  运行 </p> 
<img src="https://github.com/yangang123/picture/raw/master/qgroundcontrol/run.png" height="720" width="1280" > 
</div>

# 总结
今天一直在调试qt版本和qgroundcontrol版本对应的问题，终于找到了以上2个版本是匹配的。