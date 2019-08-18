@[TOC]
# 资源
> nuttx_wqueue.md

# 工作队列实现
优点：最短时间调度
缺点：工作队列执行完后，需要重新创建.


## 添加工作对象
```c
static int work_qqueue(FAR struct usr_wqueue_s *wqueue,
                       FAR struct work_s *work, worker_t worker,
                       FAR void *arg, systime_t delay)
 -> dq_addlast((FAR dq_entry_t *)work, &wqueue->q);
```

## 工作队列执行进程
```c
void work_process(FAR struct kwork_wqueue_s *wqueue, systime_t period, int wndx)
{

  work = (FAR struct work_s *)wqueue->q.head;
  while (work)

	ctick   = clock_systimer();
	elapsed = ctick - work->qtime;
	if (elapsed >= work->delay)
	{

	//时间到了，就执行这个节点， 执行完后，立即删除这个节点，
			(void)dq_rem((struct dq_entry_s *)work, &wqueue->q);
			work  = (FAR struct work_s *)wqueue->q.head;
			worker(arg); //执行函数

     else /* elapsed < work->delay */
        {
    //找到需要等待最短时间的那个节点。
          elapsed += (ctick - stick);
          if (elapsed > work->delay)
            {
              elapsed = work->delay;
            
          remaining = work->delay - elapsed;
          if (remaining < next)
            {
              next = remaining;
            }
          work = (FAR struct work_s *)work->dq.flink;
      

   //工作队列的对象的时间小于hpwork队列时间，则使用任务调用的时间period: 100ms
  elapsed = clock_systimer() - stick;
  if (elapsed < period && next > 0)
    {
      remaining = period - elapsed;
      next      = MIN(next, remaining);
      wqueue->worker[wndx].busy = false;
      usleep(next * USEC_PER_TICK);
      wqueue->worker[wndx].busy = true;
     }
```

# 使用注意
>注意队列中必须设计非阻塞的接口，不能调用poll类型的非阻塞接口
work_queue中，注意打开句柄的时候，一定在callback中打开，
因为在callback外打开文件句柄属于nsh，而不属于work_queue. 
所以如果调用fd在callback外部打开
会导致，收发数据不正常。 类如px4的hmc5883在work_queue中的measure奇怪的写法: 
公告主题和发布数据在1个函数中实现，和mpu6000中的公告发布不一致的问题.
```cpp
if (_mag_topic != nullptr) {
      /* publish it */
      orb_publish(ORB_ID(sensor_mag), _mag_topic, &new_report);

    } else {
      _mag_topic = orb_advertise_multi(ORB_ID(sensor_mag), &new_report,
               &_orb_class_instance, (sensor_is_onboard) ? ORB_PRIO_HIGH : ORB_PRIO_MAX);

      if (_mag_topic == nullptr) {
        DEVICE_DEBUG("ADVERT FAIL");
      }
    }
```