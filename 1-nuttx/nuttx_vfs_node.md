@[TOC]

# nuttx设备节点使用

1. i_peer当做右孩子， i_child是左孩子
2. /dev是根节点,同级目录节点是相当当前节点的右孩子

<div align="center">
<p>  </p> 
<img src="https://github.com/yangang123/picture/raw/master/1-nuttx/resource/device_tree.jpg" height="720" width="1280" > 
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


