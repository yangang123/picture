
nuttx中的nanosleep的实现方式

# 精度
精度是TICK的精度

# sleep，nanosleep

sleep
    -> nanosleep() 
        -> struct timespec
        {
        time_t tv_sec;                   /* Seconds */
        long   tv_nsec;                  /* Nanoseconds */
        };
        ->rqtp.tv_sec  = seconds; // 转成time时间
         rqtp.tv_nsec = 0;

        ->  ret = sigtimedwait(&set, NULL, rqtp); //等待信号或者超时 
            -> waitticks = MSEC2TICK(waitmsec); //转成tick 
            //注册回调函数sig_timeout
            -> wd_start(rtcb->waitdog, waitticks, (wdentry_t)sig_timeout, 1,
                       wdparm.pvarg);
            //阻塞任务   
            ->up_block_task(rtcb, TSTATE_WAIT_SIG);
           

usleep
    -> nanosleep()

阻塞任务
up_block_task(rtcb, TSTATE_WAIT_SIG);

释放线程
static void sig_timeout(int argc, wdparm_t itcb)
    up_unblock_task(u.wtcb); 