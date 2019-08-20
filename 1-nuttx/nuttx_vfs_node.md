@[TOC]

# nuttx设备节点使用

1. i_peer当做右孩子， i_child是左孩子
2. /dev是根节点,同级目录节点是相当当前节点的右孩子

<div align="center">
<p>  </p> 
<img src="https://github.com/yangang123/yangang123.github.io/raw/master/1-nuttx/resource/device_tree.jpg" height="720" width="1280" > 
</div>

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

# 插入设备节点
register_driver("/dev/console", &g_serialops, 0666, dev);
  -> static int _inode_search(FAR struct inode_search_s *desc)
    -> while (node != NULL) 
     -> int result = _inode_compare(name, node);//根据设备节点名字首字母排序，找到1个要插入节点的位置
     -> node = node->i_peer;
  ->   node = inode_alloc(name); 
        inode_insert(node, left, parent); //插入节点
        *inode = node;   //返回节点
  ->  INODE_SET_DRIVER(node);  //节点数据赋值
      node->u.i_ops   = fops;
      node->i_private = priv;

# 删除节点
int unregister_driver(FAR const char *path)
  -> ret = inode_remove(path);
     ->  inode_free(node);

# 递归删除设备节点
void inode_free(FAR struct inode *node)
    ->  inode_free(node->i_peer);
    -> inode_free(node->i_child);

# 多个数据priv可以对应多个设备节点
>USB注册成/dev/yangang和/dev/ttyACM0
```
nsh> ls
/dev:
 adc0
 console
 led0
 mmcsd0
 null
 pwm_output0
 px4fmu
 ram0
 tone_alarm0
 ttyACM0
 ttyS0
 ttyS1
 ttyS2
 ttyS3
 ttyS4
 ttyS5
 yangang
 
//#ifdef CONFIG_CDCACM_CONSOLE
  priv->serdev.isconsole = true;
  ret = uart_register("/dev/yangang", &priv->serdev);
  if (ret < 0)
    {
      usbtrace(TRACE_CLSERROR(USBSER_TRACEERR_CONSOLEREGISTER), (uint16_t)-ret);
      goto errout_with_class;
    }
//#endif

 sprintf(devname, CDCACM_DEVNAME_FORMAT, minor);
  ret = uart_register(devname, &priv->serdev);
  if (ret < 0)
    {
      usbtrace(TRACE_CLSERROR(USBSER_TRACEERR_UARTREGISTER), (uint16_t)-ret);
      goto errout_with_class;
    }
```
```
0x2007ff50: /dev/yanang 设备节点
0x2007ff50：/dev/ttyACM0 设备节点
(gdb) p (((struct inode *)0x2007ff20)->i_private)
$3 = (void *) 0x200272d0
(gdb) p (char *)(((struct inode *)0x2007ff20)->i_name)
$4 = 0x2007ff34 "yangang"
(gdb) p (char *)(((struct inode *)0x2007ff50)->i_name)
$6 = 0x2007ff64 "ttyACM0"
(gdb) p (char *)(((struct inode *)0x2007ff50)->i_private)
$7 = 0x200272d0 ""   //
(gdb) p (((struct inode *)0x2007ff50)->i_private)
$8 = (void *) 0x200272d0
```
# 注意
1设备可以注册到多个设备节点，1个设备节点只能对应唯一的设备