
 @[TOC]
# 资源
> px4_module.md
> 闫刚 px4_modules.h的原理 

# 简介
- 优点
这个模块类的，为其他模块提供了很方便的入口，简化了我们的代码，同时，提供了安全的start/stop的静态方法.
- 缺点
这个模板类仅仅支持单实例（不实用在mavlink中）

## px4_module.h学习

###1.查看px4_module.h的历史提交记录

$ `git log px4_module.h`

```
commit 04c4339ca3a026e682412946fdc5e554c94e19f7
Author: Beat Küng <beat-kueng@gmx.net>
Date:   Thu Apr 27 17:00:03 2017 +0200
```


###2. 类方法说明 
```
extern pthread_mutex_t px4_modules_mutex;
template<class T>
class ModuleBase
{
public:
	ModuleBase() = default;
	virtual ~ModuleBase() {}

	//单个线程的实例的入口就是这个modulebase的静态main的入口
	static int main(int argc, char *argv[])
	{
		if (argc <= 1 || strcmp(argv[1], "help") == 0 || strcmp(argv[1], "-h") == 0) {
			return T::print_usage();
		}
         
        //start用来创建线程的实例, 同时会创建线程入口
		if (strcmp(argv[1], "start") == 0) {
			return start_command_base(argc - 1, argv + 1);
		}
    
        //杀死线程，销毁资源
		if (strcmp(argv[1], "stop") == 0) {
			return stop_command();
		}
        
        //查询状态
		if (strcmp(argv[1], "status") == 0) {
			return status_command();
		}
        
        //执行用户命令
		lock_module(); // we better lock here, as the method could access _object
		int ret = T::custom_command(argc - 1, argv + 1);
		unlock_module();

		return ret;
	}

	static int run_trampoline(int argc, char *argv[])
	{
		int ret = 0;
		_object = T::instantiate(argc, argv);

		if (_object) {
			T *object = (T *)_object;
			object->run();

		} else {
			PX4_ERR("failed to instantiate object");
			ret = -1;
		}

		exit_and_cleanup();

		return ret;
	}

    //线程入口的创建
	static int start_command_base(int argc, char *argv[])
	{
		int ret = 0;
		lock_module();

		if (is_running()) {
			ret = -1;
			PX4_ERR("Task already running");

		} else {
			ret = T::task_spawn(argc, argv);

			if (ret < 0) {
				PX4_ERR("Task start failed (%i)", ret);
			}
		}

		unlock_module();
		return ret;
	}

	//杀死线程，同时释放线程的资源
	static int stop_command()
	{
		int ret = 0;
		lock_module();

		if (is_running()) {
			if (_object) {
				T *object = (T *)_object;
				object->request_stop();

				unsigned int i = 0;

				do {
					unlock_module();
					usleep(20000); // 20 ms
					lock_module();

					if (++i > 100 && _task_id != -1) { // wait at most 2 sec
						if (_task_id != task_id_is_work_queue) {
							px4_task_delete(_task_id);
						}

						_task_id = -1;

						if (_object) {
							delete _object;
							_object = nullptr;
						}

						ret = -1;
						break;
					}
				} while (_task_id != -1);

			} else {
				// very unlikely event and can only happen on work queues:
				// the starting thread does not wait for the work queue to initialize,
				// and inside the work queue, the allocation of _object fails
				// and exit_and_cleanup() is not called.
				_task_id = -1;
			}
		}

		unlock_module();
		return ret;
	}

	// status是打印线程的状态
	static int status_command()
	{
		int ret = -1;
		lock_module();

		if (is_running() && _object) {
			T *object = (T *)_object;
			ret = object->print_status();

		} else {
			PX4_INFO("not running");
		}

		unlock_module();
		return ret;
	}

	//子类需要实现这个虚函数, 线程的入口
	virtual void run() {}

    //这里是判断线程是否执行的方法: 通过任务id进行判断
	static bool is_running() { return _task_id != -1; }

protected:
	// there will be one instance for each template type
	static volatile T *_object; ///< instance if the module is running
	static int _task_id;        ///< task handle: -1 = invalid, otherwise task is assumed to be running

};

//静态变量初始化， _object和_task_id
template<class T>
volatile T *ModuleBase<T>::_object = nullptr;

template<class T>
int ModuleBase<T>::_task_id = -1;
```
###3. logger使用modulebase的例子
```
class Logger : public ModuleBase<Logger>
{
public:
	Logger(LogWriter::Backend backend, size_t buffer_size, uint32_t log_interval, const char *poll_topic_name,
	       bool log_on_start, bool log_until_shutdown, bool log_name_timestamp, unsigned int queue_size);

	~Logger();

	/** @see ModuleBase */
	static int task_spawn(int argc, char *argv[]);

	/** @see ModuleBase */
	static Logger *instantiate(int argc, char *argv[]);

	/** @see ModuleBase */
	static int custom_command(int argc, char *argv[]);

	/** @see ModuleBase */
	static int print_usage(const char *reason = nullptr);

	/** @see ModuleBase::run() */
	void run() override;

	/** @see ModuleBase::print_status() */
	int print_status() override;

```
```
//logger线程
int logger_main(int argc, char *argv[])
-> return Logger::main(argc, argv);
    ->static int main(int argc, char *argv[]) //modulesbase 静态函数
		-> static int start_command_base(int argc, char *argv[]) //静态函数 
			-> 	ret = T::task_spawn(argc, argv);
				-> _task_id = px4_task_spawn_cmd("logger",
					 ->static int run_trampoline(int argc, char *argv[]) //静态函数
					 	-> _object = T::instantiate(argc, argv); //用户的实例入口
					 		//解析输入的参数
					 		->while ((ch = px4_getopt(argc, argv, "r:b:etfm:q:p:", &myoptind, &myoptarg)) != EOF) {
					 	->if (_object) {
					 	//执行周期的处理
					 	->object->run();
	
//子类继承ModuleBase	 	
class Logger : public ModuleBase<Logger>
{
public:
	Logger(LogWriter::Backend backend, size_t buffer_size, uint32_t log_interval, const char *poll_topic_name,
	       bool log_on_start, bool log_until_shutdown, bool log_name_timestamp, unsigned int queue_size);

	~Logger();
	//子类 实现run周期性方法
	void run() override;
	//子类 实现打印的
	int print_status() override;
	
```
