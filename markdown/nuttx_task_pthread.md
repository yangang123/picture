# 说明
线程是如何创建，进程是如何创建的。

# TCB基础知识
TCB主要是任务管理和group管理的内容
1. task_group_s
task_group_s {
    1. 进程id
    2. 组id
    3. 信号资源
    4. 消息队列
    5. 文件句柄
    6. soket句柄
}

2. struct tcb_s
struct tcb_s
{
    1. 任务的基本信息，名字
    2. wait信号量
    3. 信号部分
}

3. 一共3种类型的TCB
#  define TCB_FLAG_TTYPE_TASK      (0 << TCB_FLAG_TTYPE_SHIFT)  /* Normal user task */
#  define TCB_FLAG_TTYPE_PTHREAD   (1 << TCB_FLAG_TTYPE_SHIFT)  /* User pthread */
#  define TCB_FLAG_TTYPE_KERNEL    (2 << TCB_FLAG_TTYPE_SHIFT)  /* Kernel thread */


```c 
struct task_group_s
{
#if defined(HAVE_GROUP_MEMBERS) || defined(CONFIG_ARCH_ADDRENV)
  struct task_group_s *flink;       /* Supports a singly linked list            */
  gid_t tg_gid;                     /* The ID of this task group                */
#endif
#ifdef HAVE_GROUP_MEMBERS
  gid_t tg_pgid;                    /* The ID of the parent task group          */
#endif
#if !defined(CONFIG_DISABLE_PTHREAD) && defined(CONFIG_SCHED_HAVE_PARENT)
  pid_t tg_task;                    /* The ID of the task within the group      */
#endif
  uint8_t tg_flags;                 /* See GROUP_FLAG_* definitions             */

  /* Group membership ***********************************************************/

  uint8_t    tg_nmembers;           /* Number of members in the group           */
#ifdef HAVE_GROUP_MEMBERS
  uint8_t    tg_mxmembers;          /* Number of members in allocation          */
  FAR pid_t *tg_members;            /* Members of the group                     */
#endif

#if defined(CONFIG_SCHED_ATEXIT) && !defined(CONFIG_SCHED_ONEXIT)
  /* atexit support ************************************************************/

# if defined(CONFIG_SCHED_ATEXIT_MAX) && CONFIG_SCHED_ATEXIT_MAX > 1
  atexitfunc_t tg_atexitfunc[CONFIG_SCHED_ATEXIT_MAX];
# else
  atexitfunc_t tg_atexitfunc;       /* Called when exit is called.             */
# endif
#endif

#ifdef CONFIG_SCHED_ONEXIT
  /* on_exit support ***********************************************************/

# if defined(CONFIG_SCHED_ONEXIT_MAX) && CONFIG_SCHED_ONEXIT_MAX > 1
  onexitfunc_t tg_onexitfunc[CONFIG_SCHED_ONEXIT_MAX];
  FAR void *tg_onexitarg[CONFIG_SCHED_ONEXIT_MAX];
# else
  onexitfunc_t tg_onexitfunc;       /* Called when exit is called.             */
  FAR void *tg_onexitarg;           /* The argument passed to the function     */
# endif
#endif

#ifdef CONFIG_SCHED_HAVE_PARENT
  /* Child exit status **********************************************************/

#ifdef CONFIG_SCHED_CHILD_STATUS
  FAR struct child_status_s *tg_children; /* Head of a list of child status     */
#endif

#ifndef HAVE_GROUP_MEMBERS
  /* REVISIT: What if parent thread exits?  Should use tg_pgid. */

  pid_t    tg_ppid;                 /* This is the ID of the parent thread      */
#ifndef CONFIG_SCHED_CHILD_STATUS
  uint16_t tg_nchildren;            /* This is the number active children       */
#endif
#endif /* HAVE_GROUP_MEMBERS */
#endif /* CONFIG_SCHED_HAVE_PARENT */

#if defined(CONFIG_SCHED_WAITPID) && !defined(CONFIG_SCHED_HAVE_PARENT)
  /* waitpid support ************************************************************/
  /* Simple mechanism used only when there is no support for SIGCHLD            */

  uint8_t tg_nwaiters;              /* Number of waiters                        */
  sem_t tg_exitsem;                 /* Support for waitpid                      */
  int *tg_statloc;                  /* Location to return exit status           */
#endif

#ifndef CONFIG_DISABLE_PTHREAD
  /* Pthreads *******************************************************************/
                                    /* Pthread join Info:                       */
  sem_t tg_joinsem;                 /*   Mutually exclusive access to join data */
  FAR struct join_s *tg_joinhead;   /*   Head of a list of join data            */
  FAR struct join_s *tg_jointail;   /*   Tail of a list of join data            */
  uint8_t tg_nkeys;                 /* Number pthread keys allocated            */
#endif

#ifndef CONFIG_DISABLE_SIGNALS
  /* POSIX Signal Control Fields ************************************************/

  sq_queue_t tg_sigactionq;         /* List of actions for signals              */
  sq_queue_t tg_sigpendingq;        /* List of pending signals                  */
#endif

#ifndef CONFIG_DISABLE_ENVIRON
  /* Environment variables ******************************************************/

  size_t     tg_envsize;            /* Size of environment string allocation    */
  FAR char  *tg_envp;               /* Allocated environment strings            */
#endif

  /* PIC data space and address environments ************************************/
  /* Logically the PIC data space belongs here (see struct dspace_s).  The
   * current logic needs review:  There are differences in the away that the
   * life of the PIC data is managed.
   */

#if CONFIG_NFILE_DESCRIPTORS > 0
  /* File descriptors ***********************************************************/

  struct filelist tg_filelist;      /* Maps file descriptor to file             */
#endif

#if CONFIG_NFILE_STREAMS > 0
  /* FILE streams ***************************************************************/
  /* In a flat, single-heap build.  The stream list is allocated with this
   * structure.  But kernel mode with a kernel allocator, it must be separately
   * allocated using a user-space allocator.
   */

#if (defined(CONFIG_BUILD_PROTECTED) || defined(CONFIG_BUILD_KERNEL)) && \
     defined(CONFIG_MM_KERNEL_HEAP)
  FAR struct streamlist *tg_streamlist;
#else
  struct streamlist tg_streamlist;  /* Holds C buffered I/O info                */
#endif
#endif

#if CONFIG_NSOCKET_DESCRIPTORS > 0
  /* Sockets ********************************************************************/

  struct socketlist tg_socketlist;  /* Maps socket descriptor to socket         */
#endif

#ifndef CONFIG_DISABLE_MQUEUE
  /* POSIX Named Message Queue Fields *******************************************/

  sq_queue_t tg_msgdesq;            /* List of opened message queues           */
#endif

#ifdef CONFIG_ARCH_ADDRENV
  /* Address Environment ********************************************************/

  group_addrenv_t tg_addrenv;       /* Task group address environment           */
#endif

#ifdef CONFIG_MM_SHM
  /* Shared Memory **************************************************************/

  struct group_shm_s tg_shm;        /* Task shared memory logic                 */
#endif
};
#endif

struct tcb_s
{
  /* Fields used to support list management *************************************/

  FAR struct tcb_s *flink;               /* Doubly linked list                  */
  FAR struct tcb_s *blink;

  /* Task Group *****************************************************************/

#ifdef HAVE_TASK_GROUP
  FAR struct task_group_s *group;        /* Pointer to shared task group data   */
#endif

  /* Task Management Fields *****************************************************/

  pid_t    pid;                          /* This is the ID of the thread        */
  start_t  start;                        /* Thread start function               */
  entry_t  entry;                        /* Entry Point into the thread         */
  uint8_t  sched_priority;               /* Current priority of the thread      */
  uint8_t  init_priority;                /* Initial priority of the thread      */

#ifdef CONFIG_PRIORITY_INHERITANCE
#if CONFIG_SEM_NNESTPRIO > 0
  uint8_t  npend_reprio;                 /* Number of nested reprioritizations  */
  uint8_t  pend_reprios[CONFIG_SEM_NNESTPRIO];
#endif
  uint8_t  base_priority;                /* "Normal" priority of the thread     */
#endif

  uint8_t  task_state;                   /* Current state of the thread         */
#ifdef CONFIG_SMP
  uint8_t  cpu;                          /* CPU index if running or assigned    */
  cpu_set_t affinity;                    /* Bit set of permitted CPUs           */
#endif
  uint16_t flags;                        /* Misc. general status flags          */
  int16_t  lockcount;                    /* 0=preemptable (not-locked)          */
#ifdef CONFIG_SMP
  int16_t  irqcount;                     /* 0=interrupts enabled                */
#endif
#ifdef CONFIG_CANCELLATION_POINTS
  int16_t  cpcount;                      /* Nested cancellation point count     */
#endif

#if CONFIG_RR_INTERVAL > 0 || defined(CONFIG_SCHED_SPORADIC)
  int32_t  timeslice;                    /* RR timeslice OR Sporadic budget     */
                                         /* interval remaining                  */
#endif
#ifdef CONFIG_SCHED_SPORADIC
  FAR struct sporadic_s *sporadic;       /* Sporadic scheduling parameters      */
#endif

  FAR struct wdog_s *waitdog;            /* All timed waits use this timer      */

  /* Stack-Related Fields *******************************************************/

  size_t    adj_stack_size;              /* Stack size after adjustment         */
                                         /* for hardware, processor, etc.       */
                                         /* (for debug purposes only)           */
  FAR void *stack_alloc_ptr;             /* Pointer to allocated stack          */
                                         /* Need to deallocate stack            */
  FAR void *adj_stack_ptr;               /* Adjusted stack_alloc_ptr for HW     */
                                         /* The initial stack pointer value     */

  /* External Module Support ****************************************************/

#ifdef CONFIG_PIC
  FAR struct dspace_s *dspace;           /* Allocated area for .bss and .data   */
#endif

  /* POSIX Semaphore Control Fields *********************************************/

  sem_t *waitsem;                        /* Semaphore ID waiting on             */

  /* POSIX Signal Control Fields ************************************************/

#ifndef CONFIG_DISABLE_SIGNALS
  sigset_t   sigprocmask;                /* Signals that are blocked            */
  sigset_t   sigwaitmask;                /* Waiting for pending signals         */
  sq_queue_t sigpendactionq;             /* List of pending signal actions      */
  sq_queue_t sigpostedq;                 /* List of posted signals              */
  siginfo_t  sigunbinfo;                 /* Signal info when task unblocked     */
#endif

  /* POSIX Named Message Queue Fields *******************************************/

#ifndef CONFIG_DISABLE_MQUEUE
  FAR struct mqueue_inode_s *msgwaitq;   /* Waiting for this message queue      */
#endif

  /* Library related fields *****************************************************/

  int pterrno;                           /* Current per-thread errno            */

  /* State save areas ***********************************************************/
  /* The form and content of these fields are platform-specific.                */

  struct xcptcontext xcp;                /* Interrupt register save area        */

#if CONFIG_TASK_NAME_SIZE > 0
  char name[CONFIG_TASK_NAME_SIZE+1];    /* Task name (with NUL terminator)     */
#endif
};
``` 

# 进程创建

## 进程数据结构
struct task_tcb_s
{
  /* Common TCB fields **********************************************************/

  struct tcb_s cmn;                      /* Common TCB fields                   */

  /* Task Management Fields *****************************************************/

#ifdef CONFIG_SCHED_STARTHOOK
  starthook_t starthook;                 /* Task startup function               */
  FAR void *starthookarg;                /* The argument passed to the function */
#endif

  /* [Re-]start name + start-up parameters **************************************/

  FAR char **argv;                       /* Name+start-up parameters            */
};

## 进程创建过程 
static int thread_create(FAR const char *name, uint8_t ttype, int priority,
                         int stack_size, main_t entry,
                         FAR char * const argv[])
 tcb = (FAR struct task_tcb_s *)kmm_zalloc(sizeof(struct task_tcb_s));
    -> group_setuptaskfiles(tcb); // 处理子进程和父进程的资源问题,拷贝父进程的资源到子进程,标准输入，标准输出等

# 线程创建

## 线程数据结构
struct pthread_tcb_s
{
  /* Common TCB fields **********************************************************/

  struct tcb_s cmn;                      /* Common TCB fields                   */

  /* Task Management Fields *****************************************************/

  pthread_addr_t arg;                    /* Startup argument                    */
  FAR void *joininfo;                    /* Detach-able info to support join    */

  /* Robust mutex support *******************************************************/

#ifndef CONFIG_PTHREAD_MUTEX_UNSAFE
  FAR struct pthread_mutex_s *mhead;     /* List of mutexes held by thread      */
#endif

  /* Clean-up stack *************************************************************/

#ifdef CONFIG_PTHREAD_CLEANUP
  /* tos   - The index to the next avaiable entry at the top of the stack.
   * stack - The pre-allocated clean-up stack memory.
   */

  uint8_t tos;
  struct pthread_cleanup_s stack[CONFIG_PTHREAD_CLEANUP_STACKSIZE];
#endif

  /* POSIX Thread Specific Data *************************************************/

#if CONFIG_NPTHREAD_KEYS > 0
  FAR void *pthread_data[CONFIG_NPTHREAD_KEYS];
#endif
};

## 线程创建过程
 线程是通过1个指针共享父进程的资源，这个设计很不错

 分配内存空间
 ptcb = (FAR struct pthread_tcb_s *)kmm_zalloc(sizeof(struct pthread_tcb_s));
 ret = group_bind(ptcb);
   tcb->cmn.group = ptcb->group; //子线程的groug共享父进程的group
