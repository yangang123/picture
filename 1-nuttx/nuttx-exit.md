了解下px4的nuttx是exit是如何释放任务资源的

# 平台

os: nuttx
hardware: pixhawk

# 死循环退出会自动执行exit，释放TCB资源

```
#ifdef CONFIG_NUTTX_KERNEL
  if ((tcb->cmn.flags & TCB_FLAG_TTYPE_MASK) != TCB_FLAG_TTYPE_KERNEL)
    {
      up_task_start(tcb->cmn.entry.main, argc, tcb->argv);
      exitcode = EXIT_FAILURE; /* Should not get here */
    }
  else
#endif
    {
      exitcode = tcb->cmn.entry.main(argc, tcb->argv);
    }

  /* Call exit() if/when the task returns */

  exit(exitcode);
}
```

# 举例说明
调用stop（）=> 对象的析构函数 =>对任务id置-1
```
void stop() {
    //销毁对象
    delete dev;
    dev = nullptr;
}

demo::~demo() {
   //让task线程退出死循环
    if (_task_should_exit) {
       _task_should_exit = false;
       _task = -1;
    }

   //关闭打开的IO资源
   if (_fd > 0) {
        ::close(_fd);
    }

    //对象的指镇
    dev = nullptr;
}
```