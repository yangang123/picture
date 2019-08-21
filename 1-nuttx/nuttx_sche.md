
@[TOC]
# 资源
> 闫刚 nuttx内核任务链表

# nuttx在等待信号量资源的过程中任务切换过程分析
> 当前任务需要等待信号量资源，才继续执行，所以需要把当前任务移除就绪列表，添加到等待资源列表
```c
int nxsem_wait(FAR sem_t *sem)
  ->  up_block_task(rtcb, TSTATE_WAIT_SEM);
     ->  switch_needed = sched_removereadytorun(tcb); // 当前任务移除就绪链表，下一个任务作为运行态
     -> sched_addblocked(tcb, TSTATE_WAIT_SEM); //当前任务设置为等待信号量 
        if (TLIST_ISPRIORITIZED(task_state))
            -> sched_addprioritized(btcb, tasklist); //优先级队列
        else
             -> dq_addlast((FAR dq_entry_t *)btcb, tasklist); //队列
        -> btcb->task_state = TSTATE_WAIT_SEM; //设置队列状态
     -> struct tcb_s *nexttcb = this_task(); 
        -> ((FAR struct tcb_s *)g_readytorun.head) //获取等待队列的头部
     —> up_switchcontext(rtcb->xcp.regs, nexttcb->xcp.regs); //任务切换
```

# nuttx在释放信号量资源后任务调度
> nuttx在释放信号量后会唤醒等待信号量的线程
```c
int nxsem_post(FAR sem_t *sem)
    -> up_unblock_task(stcb);
      -> sched_removeblocked(tcb); //移出去
        bool sched_removereadytorun(FAR struct tcb_s *rtcb)
        //找到下一个, 设置为运行态。
        ->FAR struct tcb_s *nxttcb = (FAR struct tcb_s *)rtcb->flink;
        -> nxttcb->task_state = TSTATE_TASK_RUNNING;
            doswitch = true;

        // 刪除当前任务块，并且状态设置为无效
        -> dq_rem((FAR dq_entry_t *)rtcb, (FAR dq_queue_t *)&g_readytorun);=
        -> rtcb->task_state = TSTATE_TASK_INVALID;
      ->if (sched_addreadytorun(tcb)) //添加到运行队列
        -> up_unblock_task(stcb);
            ->if (sched_addreadytorun(tcb))
                //根据优先级进行排队
                -> else if (sched_addprioritized(btcb, (FAR dq_queue_t *)&g_readytorun))
                    for (next = (FAR struct tcb_s *)list->head;
                    (next && sched_priority <= next->sched_priority);
                    next = next->flink);
                    tcb->flink = next;
                    tcb->blink = prev;
                    prev->flink = tcb;
                    next->blink = tcb;
                //当前任务变为运行态
                -> btcb->task_state = TSTATE_TASK_RUNNING;
                btcb->flink->task_state = TSTATE_TASK_READYTORUN;
       //获取最新的任务状态进行
       -> struct tcb_s *nexttcb = this_task();     
       -> up_switchcontext(rtcb->xcp.regs, nexttcb->xcp.regs);    
```

# 任务的状态
> 任务状态有下面几个， 优先级任务状态和队列是对应的, 所有的任务加起来就是任务的所有链表
```c
enum tstate_e
{
  TSTATE_TASK_INVALID    = 0, /* INVALID      - The TCB is uninitialized */
  TSTATE_TASK_PENDING,        /* READY_TO_RUN - Pending preemption unlock */
  TSTATE_TASK_READYTORUN,     /* READY-TO-RUN - But not running */
  TSTATE_TASK_RUNNING,        /* READY_TO_RUN - And running */

  TSTATE_TASK_INACTIVE,       /* BLOCKED      - Initialized but not yet activated */
  TSTATE_WAIT_SEM,            /* BLOCKED      - Waiting for a semaphore */
#ifndef CONFIG_DISABLE_SIGNALS
  TSTATE_WAIT_SIG,            /* BLOCKED      - Waiting for a signal */
#endif
#ifndef CONFIG_DISABLE_MQUEUE
  TSTATE_WAIT_MQNOTEMPTY,     /* BLOCKED      - Waiting for a MQ to become not empty. */
  TSTATE_WAIT_MQNOTFULL,      /* BLOCKED      - Waiting for a MQ to become not full. */
#endif
#ifdef CONFIG_PAGING
  TSTATE_WAIT_PAGEFILL,       /* BLOCKED      - Waiting for page fill */
#endif
#ifdef CONFIG_SIG_SIGSTOP_ACTION
  TSTATE_TASK_STOPPED,        /* BLOCKED      - Waiting for SIGCONT */
#endif

  NUM_TASK_STATES             /* Must be last */
};

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

  {                                              /* TSTATE_TASK_READYTORUN */
    &g_readytorun,
    TLIST_ATTR_PRIORITIZED | TLIST_ATTR_RUNNABLE
  },
  {                                              /* TSTATE_TASK_RUNNING */
    &g_readytorun,
    TLIST_ATTR_PRIORITIZED | TLIST_ATTR_RUNNABLE
  },
  {                                              /* TSTATE_TASK_INACTIVE */
    &g_inactivetasks,
    0
  },
  {                                              /* TSTATE_WAIT_SEM */
    &g_waitingforsemaphore,
    TLIST_ATTR_PRIORITIZED
  }
  ,
  {                                              /* TSTATE_WAIT_SIG */
    &g_waitingforsignal,
    0
  }
};
```

# 总结
nuttx的任务调度在查找最高优先级任务的算法是优先级队列，它的时间复杂度是O(n), 在任务数量少的时候，查找最有优先级的任务是没有问题的，但是
任务较多的时候，显然这种调度算法不合理，linux在2.4内核之前也是一样调度算法，可以学习ygOS, 使用bitmap，查找最高优先级的时间复杂度是O1