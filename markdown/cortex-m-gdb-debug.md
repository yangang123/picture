@[TOC]

# nuttx设备节点使用调试

# 节点数据类型
struct inode
{
  FAR struct inode *i_peer;     /* Link to same level inode */
  FAR struct inode *i_child;    /* Link to lower level inode */
  int16_t           i_crefs;    /* References to inode */
  uint16_t          i_flags;    /* Flags for inode */
  union inode_ops_u u;          /* Inode operations */
#ifdef CONFIG_FILE_MODE
  mode_t            i_mode;     /* Access mode flags */
#endif
  FAR void         *i_private;  /* Per inode driver private data */
  char              i_name[1];  /* Name of inode (variable) */
};

# GGB调试

## 1. 启动st-util
sudo st-util
st-util 1.4.0-50-g7fafee2
2019-08-07T02:48:14 INFO common.c: Loading device parameters....
2019-08-07T02:48:14 INFO common.c: Device connected is: F76xxx device, id 0x10006451
2019-08-07T02:48:14 INFO common.c: SRAM size: 0x80000 bytes (512 KiB), Flash: 0x200000 bytes (2048 KiB) in pages of 2048 bytes
2019-08-07T02:48:14 INFO gdb-server.c: Chip ID is 00000451, Core ID is  5ba02477.
2019-08-07T02:48:14 INFO gdb-server.c: Chip clidr: 05fa0004, I-Cache: on, D-Cache: off
2019-08-07T02:48:14 INFO gdb-server.c:  cache: LoUU: 0, LoC: 5, LoUIS: 7
2019-08-07T02:48:14 INFO gdb-server.c:  cache: ctr: 05fa0004, DminLine: 4096 bytes, IminLine: 64 bytes
2019-08-07T02:48:14 INFO gdb-server.c: D-Cache L0: 2019-08-07T02:48:14 INFO gdb-server.c: f00fe019 LineSize: 8, ways: 4, sets: 128 (width: 12)
2019-08-07T02:48:14 INFO gdb-server.c: D-Cache L5: 2019-08-07T02:48:14 INFO gdb-server.c: f00fe019 LineSize: 8, ways: 4, sets: 128 (width: 12)
2019-08-07T02:48:14 INFO gdb-server.c: Listening at *:4242...

## 2. 加载elf
arm-none-eabi-gdb nuttx_px4nucleoF767ZI-v1_default.elf

## 3. 链接gdb server 
> 程序加载完成后自动停止到复位向量位置
(gdb) target remote :4242
Remote debugging using :4242
__start () at chip/stm32_start.c:314
314	{

## 添加2个段断点

(gdb) info break
Num     Type           Disp Enb Address    What
2       breakpoint     keep y   0x08009154 in up_idle at common/up_idle.c:63

## 运行
(gdb) c
Continuing.
Note: automatically using hardware breakpoints for read-only addresses.

Breakpoint 2, up_idle () at common/up_idle.c:63
63	{
(gdb) 

## 查看全局变量g_root_inode
> 查看2个内存地址，数据显示格式16进行x, w表示是按照32位显示
(gdb) x /2xw g_root_inode
0x2007c010:	0x2007f7e0	0x2007c070

## 查看当前节点的name
>i_name偏移量是0x14
(gdb) p (char*)0x2007f7e0+0x14
$2 = 0x2007f7f4 "etc"

