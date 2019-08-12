@

# 任务pidhash

## pid分配

> 最多需要计算32次
> g_lastpid记录pid的值
#define CONFIG_MAX_TASKS 32

static int task_assignpid(FAR struct tcb_s *tcb)
  for (tries = 0; tries < CONFIG_MAX_TASKS; tries++)
    {
      next_pid = ++g_lastpid;
      hash_ndx = PIDHASH(next_pid);
      if (!g_pidhash[hash_ndx].tcb)

## TCB删除
```c 
static void sched_releasepid(pid_t pid)
{
    int hash_ndx = PIDHASH(pid);
    g_pidhash[hash_ndx].tcb   = NULL;
    g_pidhash[hash_ndx].pid   = INVALID_PROCESS_ID;
```

# 任务状态
> nuttx每个任务状态都是1个队列
```
const struct tasklist_s g_tasklisttable[NUM_TASK_STATES] =
{
  {                                              /* TSTATE_TASK_INVALID */
    NULL,
    0
  },
  {                                              /* TSTATE_TASK_PENDING */
    &g_pendingtasks,
    TLIST_ATTR_PRIORITIZED
  },
#ifdef CONFIG_SMP
  {                                              /* TSTATE_TASK_READYTORUN */
    &g_readytorun,
    TLIST_ATTR_PRIORITIZED
  },
  {                                              /* TSTATE_TASK_ASSIGNED */
    g_assignedtasks,
    TLIST_ATTR_PRIORITIZED | TLIST_ATTR_INDEXED | TLIST_ATTR_RUNNABLE
  },
  {                                              /* TSTATE_TASK_RUNNING */
    g_assignedtasks,
    TLIST_ATTR_PRIORITIZED | TLIST_ATTR_INDEXED | TLIST_ATTR_RUNNABLE
  },
#else
  {                                              /* TSTATE_TASK_READYTORUN */
    &g_readytorun,
    TLIST_ATTR_PRIORITIZED | TLIST_ATTR_RUNNABLE
  },
  {                                              /* TSTATE_TASK_RUNNING */
    &g_readytorun,
    TLIST_ATTR_PRIORITIZED | TLIST_ATTR_RUNNABLE
  },
#endif
  {                                              /* TSTATE_TASK_INACTIVE */
    &g_inactivetasks,
    0
  },
  {                                              /* TSTATE_WAIT_SEM */
    &g_waitingforsemaphore,
    TLIST_ATTR_PRIORITIZED
  }
#ifndef CONFIG_DISABLE_SIGNALS
  ,
  {                                              /* TSTATE_WAIT_SIG */
    &g_waitingforsignal,
    0
  }
#endif
#ifndef CONFIG_DISABLE_MQUEUE
  ,
  {                                              /* TSTATE_WAIT_MQNOTEMPTY */
    &g_waitingformqnotempty,
    TLIST_ATTR_PRIORITIZED
  },
  {                                              /* TSTATE_WAIT_MQNOTFULL */
    &g_waitingformqnotfull,
    TLIST_ATTR_PRIORITIZED
  }
#endif
#ifdef CONFIG_PAGING
  ,
  {                                              /* TSTATE_WAIT_PAGEFILL */
    &g_waitingforfill,
    TLIST_ATTR_PRIORITIZED
  }
#endif
};
```

## usleep触发的任务调度过程
```
#0  up_switchcontext () at armv7-m/gnu/up_switchcontext.S:87
#1  0x08004ab0 in sigtimedwait (set=set@entry=0x2007d670, info=info@entry=0x0, timeout=timeout@entry=0x2007d698) at signal/sig_timedwait.c:301
#2  0x080156cc in nanosleep (rqtp=rqtp@entry=0x2007d698, rmtp=rmtp@entry=0x0) at signal/sig_nanosleep.c:145
#3  0x0800daa8 in usleep (usec=usec@entry=5000) at unistd/lib_usleep.c:123
#4  0x08004744 in work_process (wqueue=wqueue@entry=0x2002254c <g_hpwork>, period=<optimized out>, wndx=wndx@entry=0) at wqueue/kwork_process.c:261
#5  0x08003e2a in work_hpthread (argc=<optimized out>, argv=<optimized out>) at wqueue/kwork_hpthread.c:119
#6  0x08003cd2 in task_start () at task/task_start.c:131
#7  0x00000000 in ?? ()
```

## 通过swi触发任务切换
```asm
82		.type	up_switchcontext, function
83	up_switchcontext:
84	
85		/* Perform the System call with R0=1, R1=saveregs, R2=restoreregs */
86	
87		mov		r2, r1					/* R2: restoreregs */
88		mov		r1, r0					/* R1: saveregs */
89		mov		r0, #SYS_switch_context	/* R0: context switch */
90		svc		0						/* Force synchronous SVCall (or Hard Fault) */
91	
```

## 软中断执行过程
```asm
#0  up_svcall (irq=11, context=0x2002595c, arg=0x0) at armv7-m/up_svcall.c:163
#1  0x080003b0 in up_doirq (irq=11, regs=0x2002595c) at armv7-m/up_doirq.c:86
#2  0x0800023e in exception_common () at armv7-m/gnu/up_exception.S:209
Backtrace stopped: previous frame identical to this frame (corrupt stack?)
```


