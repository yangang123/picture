# WorkQueueManager线程
在很早以前px4的传感器数据是通过hrt在定时器中断服务程序中阻塞方式通过spi接口读取数据，由于spi的单个transfer的时间大约在几us,这种方式，当前线程，专门处理spi的请求，用来读取传感器数据

- spi dma
- queue
- sem_wait
- c++ oop


## spi dma

```
#ifdef CONFIG_STM32F7_SPI_DMA
static void spi_exchange(FAR struct spi_dev_s *dev, FAR const void *txbuffer,
                         FAR void *rxbuffer, size_t nwords)
{
  FAR struct stm32_spidev_s *priv = (FAR struct stm32_spidev_s *)dev;

  DEBUGASSERT(priv != NULL);

#ifdef CONFIG_STM32F7_DMACAPABLE
  if ((txbuffer && !stm32_dmacapable((uint32_t)txbuffer, nwords, priv->txccr)) ||
      (rxbuffer && !stm32_dmacapable((uint32_t)rxbuffer, nwords, priv->rxccr)))
    {
      /* Unsupported memory region, fall back to non-DMA method. */

      spi_exchange_nodma(dev, txbuffer, rxbuffer, nwords);
    }
  else
#endif
    {
      static uint8_t rxdummy[ARMV7M_DCACHE_LINESIZE]
        __attribute__((aligned(ARMV7M_DCACHE_LINESIZE)));
      static const uint16_t txdummy = 0xffff;
      size_t buflen = nwords;

      if (spi_9to16bitmode(priv))
        {
          buflen = nwords * sizeof(uint16_t);
        }

      spiinfo("txbuffer=%p rxbuffer=%p nwords=%d\n", txbuffer, rxbuffer, nwords);
      DEBUGASSERT(priv->spibase != 0);

      /* Setup DMAs */

      spi_dmarxsetup(priv, rxbuffer, (uint16_t *)rxdummy, nwords);
      spi_dmatxsetup(priv, txbuffer, &txdummy, nwords);

      /* Flush cache to physical memory */

      if (txbuffer)
        {
          arch_flush_dcache((uintptr_t)txbuffer, (uintptr_t)txbuffer + buflen);
        }

      /* Start the DMAs */

      spi_dmarxstart(priv);
      spi_dmatxstart(priv);

      /* Then wait for each to complete */

      spi_dmarxwait(priv);
      spi_dmatxwait(priv);

      /* Force RAM re-read */

      if (rxbuffer)
        {
          arch_invalidate_dcache((uintptr_t)rxbuffer,
                                 (uintptr_t)rxbuffer + buflen);
        }
      else
        {
          arch_invalidate_dcache((uintptr_t)rxdummy,
                                 (uintptr_t)rxdummy + sizeof(rxdummy));
        }
    }
}
```
## 信号量最高效率

等待信号量
spi_dmarxwait(priv);
spi_dmatxwait(priv);
```
//启动DMA，注册毁掉函数
static void spi_dmarxstart(FAR struct stm32_spidev_s *priv)
{
  priv->rxresult = 0;
  stm32_dmastart(priv->rxdma, spi_dmarxcallback, priv, false);
}

//执行回调函数
static void spi_dmarxcallback(DMA_HANDLE handle, uint8_t isr, void *arg)
{
  FAR struct stm32_spidev_s *priv = (FAR struct stm32_spidev_s *)arg;

  /* Wake-up the SPI driver */

  priv->rxresult = isr | 0x080;  /* OR'ed with 0x80 to assure non-zero */
  spi_dmarxwakeup(priv);
}

//发出信号量
static inline void spi_dmarxwakeup(FAR struct stm32_spidev_s *priv)
{
  (void)sem_post(&priv->rxsem);
}
```

## queue
> 单次触发hrt
```
void
MPU6000::start()
	//定义周期
	uint32_t last_call_interval = _call_interval;
	_call_interval = last_call_interval;
	ScheduleOnInterval(_call_interval - MPU6000_TIMER_REDUCTION, 1000);

	//等待周期到达执行回调
	-> hrt_call_every(&_call, delay_us, interval_us, (hrt_callout)&ScheduledWorkItem::schedule_trampoline, this);

	//把当前对象添加到队列中
        ->dev->ScheduleNow();
	  -> inline void ScheduleNow() { if (_wq != nullptr) _wq->Add(this); }
```
## c++ oop

px4_add_library(px4_work_queue
	ScheduledWorkItem.cpp
	WorkItem.cpp
	WorkQueue.cpp
	WorkQueueManager.cpp
)

```
int WorkQueueManagerStart()
{
	//创建任务
	int task_id = px4_task_spawn_cmd("wq:manager",
					 SCHED_DEFAULT,
					 PX4_WQ_HP_BASE,
					 1200,
					 (px4_main_t)&WorkQueueManagerRun,
					 nullptr);
       //执行队列中的对象
       -> static void WorkQueueManagerRun()
	_wq_manager_wqs_list = new BlockingList<WorkQueue *>();
	_wq_manager_create_queue = new BlockingQueue<const wq_config_t *, 1>();

	while (!_wq_manager_should_exit.load()) {
		// create new work queues as needed
		const wq_config_t *wq = _wq_manager_create_queue->pop();


class MPU6000 : public cdev::CDev, public px4::ScheduledWorkItem
{
private:
	void Run() override;


static void *WorkQueueRunner(void *context)
{
	wq_config_t *config = static_cast<wq_config_t *>(context);
	WorkQueue wq(*config);

	// add to work queue list
	_wq_manager_wqs_list->add(&wq);

	wq.Run();
```
