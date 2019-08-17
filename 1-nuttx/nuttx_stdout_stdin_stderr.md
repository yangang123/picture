@[TOC]
# 资源
> nuttx_wqueue.md

# 1. linux重定向, 通道重定向符号，写入1个字符到hello.txt文件中
标准输入，标准输出，标准错误，也算是编程基础了，我讲一讲这功能在nuttx的os实现过程
```javascript
$ echo "abc" > hello.txt 
$ cat hello.txt
    abc
```

# 2. nuttx os start如何启动内核线程的
```javascript
 void os_start(void)
   创建空闲任务
   ->   g_pidhash[ PIDHASH(0)].tcb = &g_idletcb.cmn;
        g_pidhash[ PIDHASH(0)].pid = 0;
  
   创建标注输入标准输入标准错误
   ->  group_setupidlefiles(&g_idletcb)); 
     -> fd = open("/dev/console", O_RDWR);
      (void)fs_dupfd2(0, 1);
      (void)fs_dupfd2(0, 2);
   创建队列线程
   -> void)work_hpstart();
   -> (void)work_lpstart();
   -> os_do_appstart()
      -> board_initialize();
  创建init线程，其实就是insh线程
      -> pid = task_create("init", SCHED_PRIORITY_DEFAULT,
                        CONFIG_USERMAIN_STACKSIZE, USERSPACE->us_entrypoint,
                        (FAR char * const *)NULL);
```


# 3. task_creath介绍
每个线程创建的时候，会自动的复制父进程的文件句柄，这种克隆方式是一种继承思想
```javascript
static int thread_create(FAR const char *name, uint8_t ttype, int priority,
                         int stack_size, main_t entry, FAR char * const argv[])
group_setuptaskfiles(FAR struct task_tcb_s *tcb)
  -> sched_dupfiles(tcb);
  -> group_setupstreams(tcb);
      ->(void)fs_fdopen(0, O_RDONLY,         (FAR struct tcb_s *)tcb);
      ->(void)fs_fdopen(1, O_WROK | O_CREAT, (FAR struct tcb_s *)tcb);
      ->(void)fs_fdopen(2, O_WROK | O_CREAT, (FAR struct tcb_s *)tcb);
```

# 4. rcS启动nshterm线程后，为什么控制台被重定向，线程需要stop，然后start才能看到调试信息
```javascript
rcS:
 nshterm /dev/ttyACM0 &  /dev/ttyACM0作为新的控制台
 thread1 start 
 thread2 start 
 thread3 start 
```
当我们控制其切换到其他控制台的时候，我们rcs中的线程都已经启动，所以需要我们先对线程stop, 后进行start, 任务创建的时候会自动继承nsh标准输入标注输出。

# 提出一个标准输入和标准输出是阻塞式设计还是非阻塞？
```
#define O_RDONLY    (1 << 0)       
#define O_RDOK      O_RDONLY       
#define O_WRONLY    (1 << 1)        
#define O_WROK      O_WRONLY        
#define O_RDWR      (O_RDOK|O_WROK) 
#define O_CREAT     (1 << 2)        
#define O_EXCL      (1 << 3)       
#define O_APPEND    (1 << 4)        
#define O_TRUNC     (1 << 5)        
#define O_NONBLOCK  (1 << 6)        
#define O_NDELAY    O_NONBLOCK      
#define O_SYNC      (1 << 7)        
#define O_DSYNC     O_SYNC          
#define O_BINARY    (1 << 8)       
```

fd = open("/dev/console", O_RDWR);
可以看出标准输入和标准输出都是阻塞式设计，

 




