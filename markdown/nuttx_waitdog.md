@[TOC]

# TCB中的waitdog实现
waitdog是挂载TCB上，用来监控超时等待函数接口，比如simwait或者poll机制等，在nuttx的链表中也是没有数据域
的,设计思想思想，保证链表的数据结构的第一个字节是指针域

# waitdog是挂载TCB上，用来监控超时等待函数接口

semaphore/sem_tickwait.c:99:  rtcb->waitdog = wd_create();
semaphore/sem_timedwait.c:123:  rtcb->waitdog = wd_create();
signal/sig_timedwait.c:252:          rtcb->waitdog = wd_create();
mqueue/mq_timedreceive.c:197:  rtcb->waitdog = wd_create();
mqueue/mq_timedsend.c:250:  rtcb->waitdog = wd_create();
timer/timer_create.c:184:  wdog = wd_create();
pthread/pthread_condtimedwait.c:208:      rtcb->waitdog = wd_create();
wdog/Make.defs:36:CSRCS += wd_initialize.c wd_create.c wd_start.c wd_cancel.c wd_delete.c
wdog/wd_create.c:2: * sched/wdog/wd_create.c
wdog/wd_create.c:57: * Name: wd_create
wdog/wd_create.c:60: *   The wd_create function will create a watchdog by allocating it from the
wdog/wd_create.c:74:WDOG_ID wd_create (void)

# wdog的数据结构
单向队列
```c
struct sq_entry_s
{
  FAR struct sq_entry_s *flink;
};
typedef struct sq_entry_s sq_entry_t;

struct sq_queue_s
{
  FAR sq_entry_t *head;
  FAR sq_entry_t *tail;
};
typedef struct sq_queue_s  sq_queue_t;
```

# 单向链表没有数据域的原因是，保证数据结构的第一指针是next指针
```c
struct wdog_s
{
  FAR struct wdog_s *next;       /* Support for singly linked lists. */
  wdentry_t          func;       /* Function to execute when delay expires */
#ifdef CONFIG_PIC
  FAR void          *picbase;    /* PIC base address */
#endif
  int                lag;        /* Timer associated with the delay */
  uint8_t            flags;      /* See WDOGF_* definitions above */
  uint8_t            argc;       /* The number of parameters to pass */
  wdparm_t           parm[CONFIG_MAX_WDOGPARMS];
};
static struct wdog_s g_wdpool[CONFIG_PREALLOC_WDOGS];
```

# 下面的操作会让静态数组的内存连接起来
```c
sq_init(&g_wdfreelist);
for (i = 0; i < CONFIG_PREALLOC_WDOGS; i++)
{
    sq_addlast((FAR sq_entry_t *)wdog++, &g_wdfreelist);
    ->queue->tail->flink = node;
    ->queue->tail        = node;
-> g_wdnfree = CONFIG_PREALLOC_WDOGS;
```

#  系统先分配系统内存，或者是在中断中需要快速分配内存使用内存池静态分配
WDOG_ID wd_create (void)
 if (g_wdnfree > CONFIG_WDOG_INTRESERVE || up_interrupt_context())
        wdog = (FAR struct wdog_s *)sq_remfirst(&g_wdfreelist);
 else 
    ->  wdog = (FAR struct wdog_s *)kmm_malloc(sizeof(struct wdog_s));


    