# 资源
> linux_build_px4_posix.md
> 闫刚 px4仿真架构

# 编译
> 切换到1.8.0版本，编译posix_sitl_test，生成px4可执行文件
```
1. git check tag_v1.8.0
2. make posix_sitl_test
3. build/posix_sitl_test  
```

# 分析px4可执行文件中编译了那些模块
通过px4的elf分析代码中有那些模块，同时知道了main的入口，readelf -s xxx表示读取可执行文件的代码段

```c
yangang@yangang-ubuntu:~/work/px4_proj/Firmware/build/posix_sitl_test$ readelf -s px4 |grep -n "main.cpp"
1465:    44: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS replay_main.cpp
1474:    53: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS main.cpp
1573:   152: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS land_detector_main.cpp
1584:   163: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS navigator_main.cpp
1585:   164: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS ekf2_main.cpp
1588:   167: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS wind_estimator_main.cpp
1594:   173: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS mc_att_control_main.cpp
1640:   219: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS MS5525_main.cpp
1644:   223: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS SDP3X_main.cpp
1770:   349: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS controllib_test_main.cpp
1782:   361: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS mc_pos_control_main.cpp
1786:   365: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS uORB_tests_main.cpp
1836:   415: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS hello_main.cpp
1838:   417: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS hrt_test_main.cpp
1840:   419: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS muorb_test_main.cpp
1842:   421: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS vcdevtest_main.cpp
1886:   465: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS mavlink_main.cpp
2007:   586: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS vtol_att_control_main.cpp
2098:   677: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS main.cpp
```

# 启动px4，看到了需要我们制定启动脚
>"startup_config"是需要指定的启动脚本文件
```shell
yangang@yangang-ubuntu:~/work/px4_proj/Firmware/build/posix_sitl_test$ ./px4 
ERROR [Unknown] Error expected 1 or 2 position arguments, got 0
./px4 [-d] [data_directory] startup_config [-h]
   -d            - Optional flag to run the app in daemon mode and does not listen for user input.
                   This is needed if px4 is intended to be run as a upstart job on linux
<data_directory> - directory where ROMFS and posix-configs are located (if not given, CWD is used)
<startup_config> - config file for starting/stopping px4 modules
   -h            - help/usage information
Restoring terminal
```
# 我们先用px4的shell/iris的脚本, 发现已经正常跑了起来
```shell
yangang@yangang-ubuntu:~/work/px4_proj/Firmware/build/posix_sitl_test$ ./px4  ../../posix-configs/SITL/init/shell/iris
commands file: ../../posix-configs/SITL/init/shell/iris
58 WARNING: setRealtimeSched failed (not run as root?)

______  __   __    ___ 
| ___ \ \ \ / /   /   |
| |_/ /  \ V /   / /| |
|  __/   /   \  / /_| |
| |     / /^\ \ \___  |
\_|     \/   \/     |_/

px4 starting.

pxh> 
```
#  启动qgroundstation
```
yangang@yangang-ubuntu:~/work/github_proj/fight$ ./QGroundControl.AppImage 
```
# 在px4中启动mavlink,连接飞控, 发现飞机已经连接上了
```shell
pxh> mavlink start -r 800000 -u 14556 -m config
INFO  [mavlink] mode: Config, data rate: 800000 B/s on udp port 14556 remote port 14550
pxh> mavlink boot_complete
INFO  [mavlink] MAVLink only on localhost (set param MAV_BROADCAST = 1 to enable network)
pxh> INFO  [mavlink] partner IP: 127.0.0.1
```
# 查看px4占用的线程有那些
```shell
yangang@yangang-ubuntu:/proc$ ps -ef |grep "px4"
yangang  30631 22252  0 18:32 pts/1    00:00:01 ./px4 ../../posix-configs/SITL/init/shell/iris
yangang@yangang-ubuntu:/proc/30631$ ls
yangang@yangang-ubuntu:/proc/30631$ cd task/
yangang@yangang-ubuntu:/proc/30631/task$ ls
30631  30632  30634  30636  30638  30640  31297  31298
yangang@yangang-ubuntu:/proc/30631/task$ ps -T -p 30631
  PID  SPID TTY          TIME CMD
30631 30631 pts/1    00:00:00 px4
30631 30632 pts/1    00:00:00 DFWorker
30631 30634 pts/1    00:00:00 hpwork
30631 30636 pts/1    00:00:00 lpwork
30631 30638 pts/1    00:00:00 wkr_hrt
30631 30640 pts/1    00:00:00 dataman
30631 31297 pts/1    00:00:01 mavlink_if0
30631 31298 pts/1    00:00:00 mavlink_rcv_if0
```
# posix px4启动
```shell
mavlink start -r 800000 -u 14556 -m config
```