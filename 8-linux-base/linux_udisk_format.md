@[TOC]

# 资源
> linux_udisk_format.md
> 闫刚 linux下对u盘进行分区格式化

# 基础知识
> /dev/sdc /dev/sdc1是不一样的意思，/dev/sdc1是分区节点，/dev/sdc是磁盘的设备节点

# 格式化u盘分区

## 1. 查看当前u盘被挂载到那个设备节点上
```
[ 2655.353374] scsi 34:0:0:0: Direct-Access     SanDisk  Ultra            1.00 PQ: 0 ANSI: 6
[ 2655.354609] sd 34:0:0:0: Attached scsi generic sg1 type 0
[ 2655.362201] sd 34:0:0:0: [sdc] 60063744 512-byte logical blocks: (30.8 GB/28.6 GiB)
[ 2655.370525] sd 34:0:0:0: [sdc] Write Protect is off
[ 2655.370532] sd 34:0:0:0: [sdc] Mode Sense: 43 00 00 00
[ 2655.377438] sd 34:0:0:0: [sdc] Write cache: disabled, read cache: enabled, doesn't support DPO or FUA
[ 2655.422249]  sdc: sdc1
[ 2655.450957] sd 34:0:0:0: [sdc] Attached SCSI removable disk
[ 2656.037073] FAT-fs (sdc1): Volume was not properly unmounted. Some data may be corrupt. Please run fsck.
```
## 2. 进入u盘
```
yangang@ubuntu:/mnt/udisk$ fdisk /dev/sdc 

欢迎使用 fdisk (util-linux 2.31.1)。
更改将停留在内存中，直到您决定将更改写入磁盘。
使用写入命令前请三思。

fdisk: 打不开 /dev/sdc: 权限不够
yangang@ubuntu:/mnt/udisk$ sudo fdisk /dev/sdc 

欢迎使用 fdisk (util-linux 2.31.1)。
更改将停留在内存中，直到您决定将更改写入磁盘。
使用写入命令前请三思。
```

## 3. 查看u盘分区表
```
命令(输入 m 获取帮助)： p
Disk /dev/sdc：28.7 GiB，30752636928 字节，60063744 个扇区
单元：扇区 / 1 * 512 = 512 字节
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：dos
磁盘标识符：0x860fee1b

设备       启动     起点     末尾     扇区  大小 Id 类型
/dev/sdc1       30063743 60063743 30000001 14.3G  b W95 FAT32

命令(输入 m 获取帮助)： m

帮助：

  DOS (MBR)
   a   开关 可启动 标志
   b   编辑嵌套的 BSD 磁盘标签
   c   开关 dos 兼容性标志

  常规
   d   删除分区
```
## 4.  修改u盘分区格式
```
可以通过输入字符't'，然后输入'b', 然后修改当前分区类型为FAT32格式
```

## 5. 把u盘制作成FAT文件系统
```
sudo mkfs.vfat /dev/sdc 
```
## 6.  把u盘制作ext4格式
```
sudo mkfs.ext4 /dev/sdd1
mke2fs 1.44.1 (24-Mar-2018)
创建含有 7507712 个块（每块 4k）和 1880480 个inode的文件系统
文件系统UUID：014dcc39-9e0a-45dd-a4c7-53bb80b930a1
超级块的备份存储于下列块：
    32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
    4096000

正在分配组表： 完成                            
正在写入inode表： 完成                            
创建日志（32768 个块） 完成
写入超级块和文件系统账户统计信息：        
已完成
```

