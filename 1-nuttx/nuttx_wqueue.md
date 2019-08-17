# 工作队列实现
优点：最短时间调度
缺点：工作队列执行完后，需要重新创建.

## 添加工作对象
static int work_qqueue(FAR struct usr_wqueue_s *wqueue,
                       FAR struct work_s *work, worker_t worker,
                       FAR void *arg, systime_t delay)
 -> dq_addlast((FAR dq_entry_t *)work, &wqueue->q);

## 工作队列执行进程
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
   