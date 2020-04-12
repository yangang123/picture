# 资源
> 闫刚 px4 sitl软件在还仿真

# 编译
> 切换到v1.11.0-beta1版本，编译posix_sitl_test，生成px4可执行文件
```
1. git check tag_vv1.11.0-beta1
2. make px4_sitl jmavsim
```

# 分析px4可执行文件中编译了那些模块
通过px4的elf分析代码中有那些模块，同时知道了main的入口，readelf -s xxx表示读取可执行文件的代码段

```
yangang@ubuntu:~/work/Firmware$ readelf -s build/px4_sitl_default/bin/px4 | grep -n "main.cpp"
1057:    11: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS main.cpp
1076:    30: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS cdevtest_main.cpp
1077:    31: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS controllib_test_main.cpp
1082:    36: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS wqueue_main.cpp
1094:    48: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS uORB_tests_main.cpp
1233:   187: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS ekf2_main.cpp
1255:   209: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS land_detector_main.cpp
1294:   248: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS mavlink_main.cpp
1361:   315: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS mc_att_control_main.cpp
1364:   318: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS mc_pos_control_main.cpp
1375:   329: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS navigator_main.cpp
1400:   354: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS replay_main.cpp
1455:   409: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS vtol_att_control_main.cpp
1567:   521: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS hrt_test_main.cpp
1568:   522: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS listener_main.cpp
1587:   541: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS work_queue_main.cpp
1591:   545: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS main.cpp
1600:   554: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS hello_main.cpp
1603:   557: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS main.cpp

```

## 安装jmavsim的依赖
```
sudo add-apt-repository ppa:openjdk-r/ppa 
sudo apt-get update 
sudo apt-get install openjdk-8-jdk
sudo apt-get install ant openjdk-8-jdk openjdk-8-jre

## 切换到vv1.11.0-beta1最近版本
git checkout v1.11.0-beta1
```
## jmavsim启动失败！！！！！！
$ make px4_sitl jmavsim 
```
com.jogamp.opengl.GLException: X11GLXDrawableFactory - Could not initialize shared resources for X11GraphicsDevice[type .x11, connection :0, unitID 0, handle 0x0, owner false, ResourceToolkitLock[obj 0x117719aa, isOwner false, <451c54bf, 767aa9a5>[count 0, qsz 0, owner <NULL>]]]
	at jogamp.opengl.x11.glx.X11GLXDrawableFactory$SharedResourceImplementation.createSharedResource(X11GLXDrawableFactory.java:326)
	at jogamp.opengl.SharedResourceRunner.run(SharedResourceRunner.java:297)
	at java.lang.Thread.run(Thread.java:748)
Caused by: java.lang.NullPointerException
	at jogamp.opengl.GLContextImpl.makeCurrent(GLContextImpl.java:688)
	at jogamp.opengl.GLContextImpl.makeCurrent(GLContextImpl.java:580)
	at jogamp.opengl.x11.glx.X11GLXDrawableFactory$SharedResourceImplementation.createSharedResource(X11GLXDrawableFactory.java:297)
	... 2 more
com.jogamp.opengl.GLException: J3D-Renderer-1: createImpl ARB n/a but required, profile > GL2 requested (OpenGL >= 3.1). Requested: GLProfile[GL3bc/GL3bc.hw], current: 3.0 (Compat profile, compat[ES2], FBO, hardware) - 3.0 Mesa 18.0.5
	at jogamp.opengl.x11.glx.X11GLXContext.createImpl(X11GLXContext.java:418)
	at jogamp.opengl.GLContextImpl.makeCurrentWithinLock(GLContextImpl.java:759)
	at jogamp.opengl.GLContextImpl.makeCurrent(GLContextImpl.java:642)
	at jogamp.opengl.GLContextImpl.makeCurrent(GLContextImpl.java:580)
	at javax.media.j3d.JoglPipeline$QueryCanvas.doQuery(JoglPipeline.java:8615)
	at javax.media.j3d.JoglPipeline$QueryCanvas.access$100(JoglPipeline.java:8566)
	at javax.media.j3d.JoglPipeline.createQueryContext(JoglPipeline.java:6562)
	at javax.media.j3d.Canvas3D.createQueryContext(Canvas3D.java:4609)
	at javax.media.j3d.Canvas3D.createQueryContext(Canvas3D.java:3606)
	at javax.media.j3d.Renderer.doWork(Renderer.java:461)
```
修改方法:
在.bashrc中添加以下内容
```
export SVGA_VGPU10=0
```

# 启动sitl_run的命令分析如下

>$  ./Tools/sitl_run.sh /home/yangang/work/Firmware/build/px4_sitl_default/bin/px4  none  jmavsim  none  /home/yangang/work/Firmware /home/yangang/work/Firmware/build/px4_sitl_default

```
/Tools/sitl_run.sh /home/yangang/work/Firmware/build/px4_sitl_default/bin/px4  none  jmavsim  none  /home/yangang/work/Firmware /home/yangang/work/Firmware/build/px4_sitl_default
SITL ARGS
sitl_bin: /home/yangang/work/Firmware/build/px4_sitl_default/bin/px4
debugger: none
program: jmavsim
model: none
src_path: /home/yangang/work/Firmware
build_path: /home/yangang/work/Firmware/build/px4_sitl_default
empty model, setting iris as default
SITL COMMAND: "/home/yangang/work/Firmware/build/px4_sitl_default/bin/px4" "/home/yangang/work/Firmware"/ROMFS/px4fmu_common -s etc/init.d-posix/rcS -t "/home/yangang/work/Firmware"/test_data
INFO  [px4] Creating symlink /home/yangang/work/Firmware/ROMFS/px4fmu_common -> /home/yangang/work/Firmware/build/px4_sitl_default/tmp/rootfs/etc

______  __   __    ___ 
| ___ \ \ \ / /   /   |
| |_/ /  \ V /   / /| |
|  __/   /   \  / /_| |
| |     / /^\ \ \___  |
\_|     \/   \/     |_/

px4 starting.

INFO  [px4] Calling startup script: /bin/sh etc/init.d-posix/rcS 0
INFO  [param] selected parameter default file eeprom/parameters_10016
[param] Loaded: eeprom/parameters_10016
INFO  [dataman] Unknown restart, data manager file './dataman' size is 11798680 bytes
INFO  [simulator] Waiting for simulator to connect on TCP port 4560
Buildfile: /home/yangang/work/Firmware/Tools/jMAVSim/build.xml

make_dirs:

compile:

create_run_jar:

copy_res:

BUILD SUCCESSFUL
Total time: 0 seconds
Options parsed, starting Sim.
Starting GUI...
3D [dev] 1.6.0-pre12-daily-experimental daily

Exception in thread "AWT-EventQueue-0" java.lang.NullPointerException
	at me.drton.jmavsim.Visualizer3D.getVectorToTargetObject(Visualizer3D.java:695)
	at me.drton.jmavsim.Visualizer3D.setDynZoomDistance(Visualizer3D.java:623)
	at me.drton.jmavsim.Visualizer3D$KeyboardHandler.keyReleased(Visualizer3D.java:1160)
	at java.awt.Component.pro

建议： 直接使用sitl_run，否则会出现一些问题
```

# 启动飞机
```
px4> commander takeoff

```