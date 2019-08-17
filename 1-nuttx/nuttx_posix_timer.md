@[TOC]
# 定时器工作原理
>定时器主要是使用waitdog进行实现，周期处理是通过timer_timeout进行处理

# 资源
> nuttx_posix_timer.md 

# 定时器模块初始化
>主要是创建定时器的资源， 静态分配资源到链表中。

<div align="center">
<img src="https://github.com/yangang123/picture/raw/master/nuttx/nuttx_waitdog_datastruct.jpg" height="840" width="720" > 
</div>

void weak_function timer_initialize(void)
{
#if CONFIG_PREALLOC_TIMERS > 0
  int i;

  /* Place all of the pre-allocated timers into the free timer list */

  sq_init((FAR sq_queue_t *)&g_freetimers);

  for (i = 0; i < CONFIG_PREALLOC_TIMERS; i++)
    {
      g_prealloctimers[i].pt_flags = PT_FLAGS_PREALLOCATED;
      sq_addlast((FAR sq_entry_t *)&g_prealloctimers[i],
                 (FAR sq_queue_t *)&g_freetimers);
    }
#endif

  /* Initialize the list of allocated timers */

  sq_init((FAR sq_queue_t *)&g_alloctimers);
}

# 定时器创建
> 创建定时器主要是设置1个signal信号，后期通过发信号通知用户程序
int timer_create(clockid_t clockid, FAR struct sigevent *evp,
                 FAR timer_t *timerid)
wdog = wd_create();
ret = timer_allocate();
ret->pt_wdog  = wdog;
信号实现
ret->pt_event.sigev_notify            = SIGEV_SIGNAL;   //设置信号ALARM
ret->pt_event.sigev_signo             = SIGALRM;
ret->pt_event.sigev_value.sival_ptr   = ret;

# 启动定时器 
>启动定时器是通过timer_settime进行启动的
int timer_settime(timer_t timerid, int flags,
                  FAR const struct itimerspec *value,
                  FAR struct itimerspec *ovalue)
-> 
timer->pt_last = delay;
ret = wd_start(timer->pt_wdog, delay, (wdentry_t)timer_timeout,
    1, (uint32_t)((wdparm_t)timer));


# 定时器超时函数
static void timer_timeout(int argc, wdparm_t itimer)

  timer_restart(u.timer, itimer);