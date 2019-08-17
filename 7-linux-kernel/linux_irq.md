# 简介
>讲解RTOS和linux内核对于临界区处理

# 资源
> linux_irq.md 
> 标题:闫刚 linux应用程序为什么不要关闭中断

# RTOS处理临界区
>RTOS中,中断和线程的打断，需要做原子操作，那么就是普通的关闭中断处理，因为在RTOS中，我们经常和中断打交道。


# linux处理临界区
>linux中用户态的进程是调用内核态的资源, 中断和内核态会出现打断的线程,所有在内核态的部分要处理关闭中断的处理

> 在linux1.0中在_sleep_on中调用save_flags(flags)和save_flags();
```c
static inline void __sleep_on(struct wait_queue **p, int state)
{
	unsigned long flags;
	struct wait_queue wait = { current, NULL };

	if (!p)
		return;
	if (current == task[0])
		panic("task[0] trying to sleep");
	current->state = state;
	add_wait_queue(p, &wait);
	save_flags(flags);
	sti();
	schedule();
	remove_wait_queue(p, &wait);
	restore_flags(flags);
}
```
