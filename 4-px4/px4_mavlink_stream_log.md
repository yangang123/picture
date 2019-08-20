@[TOC]
# 资源
> px4_mavlink_stream_log.md
> 标题： 闫刚 px4_mavlink_stream_log的原理


# mavlink 消息优点
- log消息支持输出到mavlink通道和标准错误
- log消息是通过ioctl机制进入mavlink
- log消息支持队列缓冲
- log消息可以控制输出频率
- log消息可以写入sd卡中
  
# mavlink stream

## mavlink stream
> 在mavlink发送中，每一个功能是1个mavlink流。
```
public MavlinkStream {
protected:
    Mavlink     *_mavlink; //父类中，指定了每一个mavlink的对象
}

class MavlinkStreamAgriBigdata : public MavlinkStream
  -> static MavlinkStream *new_instance(Mavlink *mavlink)
    {
        return new MavlinkStreamAgriBigdata(mavlink);
    }
```

## StreamListItem的定义
```
class StreamListItem {
public:
    MavlinkStream* (*new_instance)(Mavlink *mavlink);
    const char* (*get_name)();
    StreamListItem(MavlinkStream* (*inst)(Mavlink *mavlink), const char* (*name)()) :
        new_instance(inst),
        get_name(name) {};
    ~StreamListItem() {};
};
```

## 创建好了，就可以直接使用数组，进行访问了
>new StreamListItem(&MavlinkStreamHeartbeat::new_instance, &MavlinkStreamHeartbeat::get_name_static),

##  通过名字进行，和发送的间隔进行决定是否，创建这个流，同时设置这个时间间隔
```
if (strcmp(stream_name, streams_list[i]->get_name()) == 0) {
    /* create new instance */
    stream = streams_list[i]->new_instance(this);
    stream->set_interval(interval);
```
##  mavlink多通道原理
>每个mavlink都会有一份stream。那么，获取同一个orb,同时发出。
>每个stream中，也不需要判断是否，由于没有分出来一个具体的函数，所以功能都是一样

# mavlink log 

## mavlink log接口
```
src/include/mavlink/mavlink_log.h:85:#define mavlink_log_emergency(_fd, _text, ...)		mavlink_vasprintf(_fd, MAVLINK_IOC_SEND_TEXT_EMERGENCY, _text, ##__VA_ARGS__);
src/include/mavlink/mavlink_log.h:93:#define mavlink_log_critical(_fd, _text, ...)		mavlink_vasprintf(_fd, MAVLINK_IOC_SEND_TEXT_CRITICAL, _text, ##__VA_ARGS__);
src/include/mavlink/mavlink_log.h:101:#define mavlink_log_info(_fd, _text, ...)		mavlink_vasprintf(_fd, MAVLINK_IOC_SEND_TEXT_INFO, _text, ##__VA_ARGS__);
```

## mavlink_vasprintf实现
> 调用了px4_ioctl(_fd, severity, (unsigned long)&text[0]);
```
__EXPORT void mavlink_vasprintf(int _fd, int severity, const char *fmt, ...)
{
	va_list ap;
	va_start(ap, fmt);
	char text[MAVLINK_LOG_MAXLEN + 1];
	vsnprintf(text, sizeof(text), fmt, ap);
	va_end(ap);
//进入ioctl 
	px4_ioctl(_fd, severity, (unsigned long)&text[0]);
}
```
## 设备节点
```
#define MAVLINK_LOG_DEVICE			"/dev/mavlink"

//找到打开设备节点的地方
src/drivers/px4io/px4io.cpp:1017:				_mavlink_fd = ::open(MAVLINK_LOG_DEVICE, 0);

//找到注册的设备驱动
src/modules/mavlink/mavlink_main.cpp:1696:	register_driver(MAVLINK_LOG_DEVICE, &fops, 0666, NULL);

//查找这个文件
fops

//重载这个ioctl函数
#ifdef __PX4_NUTTX
	fops.ioctl = (int (*)(file *, int, long unsigned int))&mavlink_dev_ioctl;
#endif

//找到mavlink_dev_ioctl的实现
int
#ifdef __PX4_NUTTX
Mavlink::mavlink_dev_ioctl(struct file *filp, int cmd, unsigned long arg)
#else
Mavlink::ioctl(device::file_t *filp, int cmd, unsigned long arg)
#endif
{
	switch (cmd) {
	case (int)MAVLINK_IOC_SEND_TEXT_INFO:
	case (int)MAVLINK_IOC_SEND_TEXT_CRITICAL:
	case (int)MAVLINK_IOC_SEND_TEXT_EMERGENCY: {

			const char *txt = (const char *)arg;
			struct mavlink_logmessage msg;
			strncpy(msg.text, txt, sizeof(msg.text));

			switch (cmd) {
			case MAVLINK_IOC_SEND_TEXT_INFO:
				msg.severity = MAV_SEVERITY_INFO;
				break;
         
```

##  通过mavlink写到buffer中
```
//遍历所有mavlink通道Mavlink *inst;=
->     mavlink_logbuffer_write(&inst->_logbuffer, &msg);  
->inst->_total_counter++;
```

## mavlink读取buffer数据发送
>mavlink线程周期性的读取mavlink logbuffer的数据，然后写入到串口或者文件中
```
//mavlink_message.cpp
class MavlinkStreamStatustext : public MavlinkStream
   ->void send(const hrt_abstime t)
    int lb_ret = mavlink_logbuffer_read(_mavlink->get_logbuffer(), &logmsg);
    //发送mavlink消息
   _mavlink->send_message(MAVLINK_MSG_ID_STATUSTEXT, &msg);
   //写mavlink消息到sd卡中
   if (_mavlink->get_instance_id() == 0) {
                            (void)fputs("\n", fp);
                            (void)fsync(fileno(fp));
                        }
```

##  通过mavlink输出log信息应用
```
mavlink_log_info(mavlink_fd, "[The-One] Update Param : %s", "COM_SENSOR_CHNEL");
if (!strcmp(argv[1], "testmavlinklog"))
	{
		mavlink_log_info(mavlink_fd, "[cmd] test mavlink log");
		mavlink_and_console_log_info(mavlink_fd, "[cmd] test mavlink log");
		return 0;
	}
```


