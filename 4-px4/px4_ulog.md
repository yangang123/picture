
@[TOC]
# 资源
> nuttx_wqueue.md
> 标题： 闫刚 px4的ulog原理

# 简介
>px4固件在v1.8.0，目前使用的ulog文件，介绍下ulog的使用

# ulog的使用
1. git clone https://github.com/PX4/pyulog.git
2. cd  pyulog
3. python setup.py build install
4. 打开**pyulog/test**文件夹，下面的测试都是通过**sample.ulg**文件作为测试

# 常用命令的使用
## 1. ulog_info [-h] [-v] file.ulg
- 查看ulog的信息
- 例子: ulog_info  sample.ulg

```
yangang@ubuntu:~/work/theone_proj/firmware/pyulog/test$ ulog_info  sample.ulg 
Logging start time: 0:01:52, duration: 0:01:08
Dropouts: count: 4, total duration: 0.1 s, max: 62 ms, mean: 29 ms
Info Messages:
 sys_name: PX4
 time_ref_utc: 0
 ver_hw: AUAV_X21
 ver_sw: fd483321a5cf50ead91164356d15aa474643aa73

Name (multi id, message size in bytes)    number of data points, total bytes
 actuator_controls_0 (0, 48)                 3269     156912
 actuator_outputs (0, 76)                    1311      99636
 commander_state (0, 9)                       678       6102
 control_state (0, 122)                      3268     398696
 cpuload (0, 16)                               69       1104
 ekf2_innovations (0, 140)                   3271     457940
 estimator_status (0, 309)                   1311     405099
 sensor_combined (0, 72)                    17070    1229040
 sensor_preflight (0, 16)                   17072     273152
 telemetry_status (0, 36)                      70       2520
 vehicle_attitude (0, 36)                    6461     232596
 vehicle_attitude_setpoint (0, 55)           3272     179960
 vehicle_local_position (0, 123)              678      83394
 vehicle_rates_setpoint (0, 24)              6448     154752
 vehicle_status (0, 45)                       294      13230

```


## 2. ulog_messages [-h] file.ulg
查看ulog中的uorb的消息
ulog_messages  sample.ulg

```
yangang@ubuntu:~/work/theone_proj/firmware/pyulog/test$ ulog_messages  sample.ulg
0:02:38 ERROR: [sensors] no barometer found on /dev/baro0 (2)
0:02:42 ERROR: [sensors] no barometer found on /dev/baro0 (2)
0:02:51 ERROR: [sensors] no barometer found on /dev/baro0 (2)
0:02:56 ERROR: [sensors] no barometer found on /dev/baro0 (2)
```


## 3. ulog_params [-h] [-d DELIMITER] [-i] [-o] file.ulg [params.txt]  
- 查看ulog中的参数
- ulog_params  sample.ul
```
yangang@ubuntu:~/work/theone_proj/firmware/pyulog/test$ ulog_params  sample.ulg
ATT_ACC_COMP,1
ATT_BIAS_MAX,0.0500000007451
ATT_EXT_HDG_M,0
ATT_MAG_DECL,0.0
ATT_MAG_DECL_A,1
ATT_VIBE_THRESH,0.20000000298
ATT_W_ACC,0.20000000298
ATT_W_EXT_HDG,0.10000000149
ATT_W_GYRO_BIAS,0.10000000149
ATT_W_MAG,0.10000000149
BAT_A_PER_V,15.3910303116
BAT_CAPACITY,-1.0
BAT_CNT_V_CURR,0.000805664050858
BAT_CNT_V_VOLT,0.000805664050858

```

## 4. ulog2csv [-h] [-m MESSAGES] [-d DELIMITER] [-o DIR] file.ulg
-    把ulog文件转换成csv文件
-    ulog2csv  sample.ulg


# logger的创建

## 1.1 logger对象的实例化
```
Logger::Logger(LogWriter::Backend backend, size_t buffer_size, uint32_t log_interval, const char *poll_topic_name,

     bool log_on_start, bool log_until_shutdown, bool log_name_timestamp, unsigned int queue_size) :

    //找到tooic的名字

    if (strcmp(poll_topic_name, topics[i]->o_name) == 0) {
```

## 1.2 logger 整个文件的入口,是 1个静态函数
```
static int main(int argc, char *argv[]) //静态函数
        -> static int start_command_base(int argc, char *argv[]) //静态函数
            ->  ret = T::task_spawn(argc, argv);
                -> _task_id = px4_task_spawn_cmd("logger",
                     ->static int run_trampoline(int argc, char *argv[]) //静态函数
                        -> _object = T::instantiate(argc, argv); //用户的实例入口
                            //解析输入的参数
                            ->while ((ch = px4_getopt(argc, argv, "r:b:etfm:q:p:", &myoptind, &myoptarg)) != EOF) {
                        ->if (_object) {
                        //执行周期的处理
                        ->object->run();
```

## 1.3 指定logger的周期，使用那个orb进行控制
```
case 'p':
    poll_topic = myoptarg;
    break;
```

## 1.4 启动logger,记录数据
```
void Logger::start_log_file()
   ->_writer.start_log_file(file_name);
       -> void LogWriter::start_log_file(const char *filename)
        if (_log_writer_file) {
             _log_writer_file->start_log(filename);
-> _fd = ::open(filename, O_CREAT | O_WRONLY, PX4_O_MODE_666); //create log file
```
## 1.5 logger_write_file.cpp中对象的成员
```
void lock()
{
    pthread_mutex_lock(&_mtx);

}

void unlock()
{

    pthread_mutex_unlock(&_mtx);

}
void notify()
{
    pthread_cond_broadcast(&_cv);
}
```

# logger write file
## 创建logwritefile的对象： 最小写入的文件大小是4096字节
```
//sdlog2 
modules/sdlog2/sdlog2.c:154:static const int MIN_BYTES_TO_WRITE = 2048;
modules/sdlog2/sdlog2.c:2352:       if (logbuffer_count(&lb) > MIN_BYTES_TO_WRITE) {

//logger
log_writer_file.h:149:    static constexpr size_t    _min_write_chunk = 4096;
log_writer_file.cpp:58:    _buffer_size(math::max(buffer_size, _min_write_chunk + 300))
```
## 写入文件log创建实例
```
//create
LogWriterFile::LogWriterFile(size_t buffer_size) :
    //循环缓冲区的大小，至少是写个flash的大小4096
    _buffer_size(math::max(buffer_size, _min_write_chunk + 300))
{
    pthread_mutex_init(&_mtx, nullptr);
    pthread_cond_init(&_cv, nullptr);
```

## 写1个message
```
//buffer overflow, writebuf
int LogWriterFile::write_message(void *ptr, size_t size, uint64_t dropout_start)
{
    if (_need_reliable_transfer) {
        int ret;
        while ((ret = write(ptr, size, dropout_start)) == -1) {
            unlock();
            notify(); //read
```

## 线程内部等待接收到最大缓冲的数据，就行写入
```
while (true) {

        available = get_read_ptr(&read_ptr, &is_part);


        /* if sufficient data available or partial read or terminating, exit this wait loop */

        if ((available >= _min_write_chunk) || is_part || !_should_run) {

            /* GOTO end of block */

            break;

        }

        pthread_cond_wait(&_cv, &_mtx);

    }


    pthread_mutex_unlock(&_mtx);

    written = 0;


    if (available > 0) {

        perf_begin(_perf_write)

    //sdlog2

    if (logbuffer_count(&lb) > MIN_BYTES_TO_WRITE) {

    -> if (available > 0) {

    perf_begin(_perf_write);

    written = ::write(_fd, read_ptr, available);

    perf_end(_perf_write);
```
## logger->run进行处理
```
//遍历所有的节点
    for (int instance = 0; instance < ORB_MULTI_MAX_INSTANCES; instance++) {

        if (copy_if_updated_multi(sub, instance, _msg_buffer + sizeof(ulog_message_data_header_s),

            sub_idx == next_subscribe_topic_index)) {

                //写入数据，同时给data_written进行true

                if (write_message(_msg_buffer, msg_size)) {

                data_written = true;


    //有数据要写入，就进行触发条件变量

    if (data_written) {
        _writer.notify();

    }
```

# 总结
>sdlog2是通过sdlog2的run进行，判断是否大于最小长度的条件变量
>logger是通过logger有数据要写入，直接触发线程的条件变量去判断条件是否满足