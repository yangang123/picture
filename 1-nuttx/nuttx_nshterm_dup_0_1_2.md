@[TOC]
# 资源
> nuttx_wqueue.md
> nshterm实现重定向原理

# 代码分析
> 后台运行，肯定直接启动1个线程执行
> $ nshterm /dev/ttyACM0  & 

```c
 重要代码分析
 nshterm_main(int argc, char *argv[])
      请问当前这里的fd是多少？ 答案是3
	 fd = open(argv[1], O_RDWR);
	关闭标准输入，标准输入，标准输入
       close(0);
	close(1);
	close(2);
	
	关闭0,1,2, 通过dup2决定把fd2复制给0,1,2，同时0,1,2可以正常使用了
	dup2(fd, 0);
	dup2(fd, 1);
	dup2(fd, 2);
       
       进入nsh解析，创建了1个新的nsh控制台
	nsh_consolemain(0, NULL);
       
       关闭打开的当前描述符
	close(fd);
```
通过代码分析，可以有多个控制台.

# nsh_initscript
```javascript
#ifdef CONFIG_NSH_ROMFSETC
int nsh_initscript(FAR struct nsh_vtbl_s *vtbl)
{
  static bool initialized;
  bool already;
  int ret = OK;

  /* Atomic test and set of the initialized flag */

  sched_lock();
  already     = initialized;
  initialized = true;
  sched_unlock();

  /* If we have not already executed the init script, then do so now */
  //
  if (!already)
    {
      ret = nsh_script(vtbl, "init", NSH_INITPATH);
    }

  return ret;
}
```
# 通过node分配files
```c
//分配1个文件，通过node进行分配
  fd = files_allocate(inode, oflags, 0, 0);
     -》 if (!list->fl_files[i].f_inode) //通过这个指针是否是空确定？
  if (fd < 0)
    {
      ret = EMFILE;
      goto errout_with_inode;
    }

//节点释放后，就不能访问了。


也就是dup2, 只有先的文件句柄了，而且

ret = _files_close(filep2);
  if (ret < 0)
    {
      /* An error occurred while closing the driver */

      goto errout_with_ret;
    }

  /* Increment the reference count on the contained inode */
//节点
  inode = filep1->f_inode;
  inode_addref(inode);

  /* Then clone the file structure */

  filep2->f_oflags = filep1->f_oflags;
  filep2->f_pos    = filep1->f_pos;
  filep2->f_inode  = inode;

//通过看出，dup(new, old)
相当于，new替代了old , 但是执行1个close操作, 
```