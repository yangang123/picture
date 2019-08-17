@[TOC]

# nuttx的ps命令实现原理

## 在没有添加proc虚拟文件系统实现方式
>实现方法主要通过获取全局变量的值进行获取系统的状态，nuttx版本是NuttX-6.27
```c 
int cmd_ps(FAR struct nsh_vtbl_s *vtbl, int argc, char **argv)   //ps命令
    ->sched_foreach(ps_task, vtbl);                              //遍历所有任务
        ->  nsh_output(vtbl, "%5d %3d %4s %7s%c%c %8s ",         //打印任务信息
             tcb->pid, tcb->sched_priority,
             tcb->flags & TCB_FLAG_ROUND_ROBIN ? "RR  " : "FIFO",
             g_ttypenames[(tcb->flags & TCB_FLAG_TTYPE_MASK) >> TCB_FLAG_TTYPE_SHIFT],
             tcb->flags & TCB_FLAG_NONCANCELABLE ? 'N' : ' ',
             tcb->flags & TCB_FLAG_CANCEL_PENDING ? 'P' : ' ',
             g_statenames[tcb->task_state]);
```

## Nuttx的proc文件系统
>proc是虚拟文件系统，是用户和内核交互的文件，可以通过操作proc文件系统的文件去获取系统信息，比如或者线程优先级
堆栈信息等

### nsh命令中实现"ps"命令
>下面是ps命令整个实现过程，主要是读取procfs文件系统下的文件，查看任务信息, Nuttx版本是NuttX-7.18
```cpp
#ifndef CONFIG_NSH_DISABLE_PS
  { "ps",       cmd_ps,       1, 1, NULL },
#endif文档1： 

#ifndef CONFIG_NSH_DISABLE_PS
  { "ps",       cmd_ps,       1, 1, NULL },
#endif
nsh_proccmds.c:574:int cmd_ps(FAR struct nsh_vtbl_s *vtbl, int argc, char **argv)
  -> nsh_output(vtbl, "%s\n", "COMMAND");
  -> return nsh_foreach_direntry(vtbl, "ps", CONFIG_NSH_PROC_MOUNTPOINT,
                              ps_callback, NULL);
int nsh_foreach_direntry(FAR struct nsh_vtbl_s *vtbl, FAR const char *cmd,
                         FAR const char *dirpath,
                         nsh_direntry_handler_t handler, void *pvarg)
  -> dirp = opendir(dirpath); //打开文件系统	
 for (; ; )
  if (handler(vtbl, dirpath, entryp, pvarg) <  0) //类如ps_callback的命令，读取各个文件的内容

nsh_proccmds.c:574:int cmd_ps(FAR struct nsh_vtbl_s *vtbl, int argc, char **argv)	
```

### 实现实例
```
Example
=======

  NuttShell (NSH) NuttX-6.31
  nsh> mount -t procfs /proc

  nsh> ls /proc
  /proc:
   0/
   1/

  nsh> ls /proc/1
  /proc/1:
   status
   cmdline

  nsh> cat /proc/1/status
  Name:       init
  Type:       Task
  State:      Running
  Priority:   100
  Scheduler:  SCHED_FIFO
  SigMask:    00000000
```




