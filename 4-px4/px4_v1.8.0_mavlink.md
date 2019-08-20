
@[TOC]
# 资源
> px4_v1.8.0_mavlink.md
> 标题： 闫刚 px4_v1.8.0_mavlink原理

# 1. 软件硬件平台
```
HW arch: PX4FMU_V5
HW type: V500
HW version: 0x00000000
HW revision: 0x00000000
FW git-hash: 400dff2fe41124bd4d7c048ff2c642e07528c7a0
FW version: 0.0.0 0 (0)
OS: NuttX
OS version: Release 7.22.0 (118882559)
OS git-hash: 8957a027f4214416646f384bb47b838ec3686643
```
# 2. 通过调试信息看到，px4fmuv5启动了4路mavlink
```
INFO  [mavlink] mode: Config, data rate: 800000 B/s on /dev/ttyACM0 @ 57600B
INFO  [mavlink] mode: Normal, data rate: 2000 B/s on /dev/ttyS3 @ 57600B
INFO  [mavlink] mode: Normal, data rate: 1200 B/s on /dev/ttyS1 @ 57600B
INFO  [mavlink] mode: OSD, data rate: 1000 B/s on /dev/ttyS2 @ 57600B
```

# 3. 分析rc.mavlink 
## 3.1 我们通过“/dev/ttyACM0”可以找到usb的启动位置
```
if ver hwcmp AEROFC_V1
then
    # don't start mavlink ttyACM0 on aerofc_v1
else
    # Start MAVLink
    mavlink start -r 800000 -d /dev/ttyACM0 -m config -x
    echo a
fi
```

## 3.2 我们通过“PX4FMU_V5”可以找到第2路的启动位置
```
if ver hwcmp PX4FMU_V5
    then
        mavlink start -r 2000 -b 57600 -d /dev/ttyS3
        echo b
    fi
```

## 3.3 查找第3路mavlink位置
> 通过'SYS_COMPANION'这个参数是157600，我们可以找到，第3路的启动位置
```
if param compare SYS_COMPANION 157600
then
	mavlink start -d ${MAVLINK_COMPANION_DEVICE} -b 57600 -m osd -r 1000
	echo d
fi
```

## 3.4  rcS和rc.mavlink是共用的环境变量
```
set MAVLINK_F                   default
set MAVLINK_COMPANION_DEVICE    /dev/ttyS2
if [ $MAVLINK_F == default ]
then
	# Normal mode, use baudrate 57600 (default) and data rate 1000 bytes/s
	set MAVLINK_F "-r 1200 -f"
```

# mavlink消息调试
> 通过调试信息，看出mavlink是否发送数据，或者mavlink的版本，端口的东西
```
instance #0:
        GCS heartbeat:  565075 us ago
        mavlink chan: #0
        type:           USB CDC
        flow control: OFF
        rates:
          tx: 13.472 kB/s
          txerr: 0.000 kB/s
          tx rate mult: 1.000
          tx rate max: 800000 B/s
          rx: 0.021 kB/s
        accepting commands: YES, FTP enabled: YES, TX enabled: YES
        mode: Config
        MAVLink version: 2
        transport protocol: serial (/dev/ttyACM0 @2000000)

instance #1:
        mavlink chan: #1
        no telem status.
        flow control: OFF
        rates:
          tx: 0.400 kB/s
          txerr: 0.000 kB/s
          tx rate mult: 0.384
          tx rate max: 1200 B/s
          rx: 0.000 kB/s
        accepting commands: YES, FTP enabled: NO, TX enabled: YES
        mode: Normal
        MAVLink version: 1
        transport protocol: serial (/dev/ttyS1 @57600)

instance #2:
        mavlink chan: #2
        no telem status.
        flow control: OFF
        rates:
          tx: 0.436 kB/s
          txerr: 0.000 kB/s
          tx rate mult: 0.222
          tx rate max: 1000 B/s
          rx: 0.000 kB/s
        accepting commands: YES, FTP enabled: NO, TX enabled: YES
        mode: OSD
        MAVLink version: 1
        transport protocol: serial (/dev/ttyS2 @57600)
```