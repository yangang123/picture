@[TOC]

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

## pid 
nsh> ps
  PID PRI POLICY   TYPE    NPX STATE    EVENT     SIGMASK  COMMAND
    0   0 FIFO     Kthread N-- Ready              00000000 Idle Task
    1 249 FIFO     Kthread --- Waiting  Signal    00000000 hpwork
    2  50 FIFO     Kthread --- Waiting  Signal    00000000 lpwork
    3 100 FIFO     Task    --- Running            00000000 init
  133 100 FIFO     Task    --- Waiting  Signal    00000000 mavlink_if2 mavlink start -r 800000 -d /dev/ttyACM0 -m config -x
  135 105 FIFO     Task    --- Waiting  Semaphore 00000000 navigator start
   40 220 FIFO     Task    --- Waiting  Semaphore 00000000 gps start
  105 100 FIFO     Task    --- Waiting  Signal    00000000 mavlink_if1 mavlink start -d /dev/ttyS2 -b 57600 -m osd -r 1000
  106 175 FIFO     pthread --- Waiting  Semaphore 00000000 mavlink_rcv_if1 0x2002dc40
   76 249 FIFO     Task    --- Waiting  Semaphore 00000000 sensors start
  141 250 FIFO     Task    --- Waiting  Semaphore 00000000 ekf2 start
   78 140 FIFO     Task    --- Waiting  Signal    00000000 commander start
   79  50 FIFO     pthread --- Waiting  Semaphore 00000000 commander_low_prio 0x0
   90 100 FIFO     Task    --- Waiting  Signal    00000000 mavlink_if0 mavlink start -r 1200 -f
   91 175 FIFO     pthread --- Waiting  Semaphore 00000000 mavlink_rcv_if0 0x2002e410
  158 245 FIFO     Task    --- Waiting  Semaphore 00000000 logger start -b 14 -t
  159  60 FIFO     pthread --- Waiting  Semaphore 00000000 log_writer_file 0x2002ea90

## pid转hash表
···c
#include <stdio.h>
int pid[32] = {0,1,2,3,133,135,40,105,106,76,141,78,79,90,91,158,159};
#define CONFIG_MAX_TASKS 32
int main()
{	
	for (int i = 0; i < 32; i++) 
		printf("pid:%d\n", pid[i] % CONFIG_MAX_TASKS);
	return 0;
}
pid:0
pid:1
pid:2
pid:3
pid:5
pid:7
pid:8
pid:9
pid:10
pid:12
pid:13
pid:14
pid:15
pid:26
pid:27
pid:30
pid:31

``

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


markdown/nuttx_taskswitch.md