
@[TOC]
# 资源
> px4_gps.md
> 标题： 闫刚 px4的gps驱动原理

# 1. rcS启动
> 固件版本： V1.8.0
```
gps start 
```

# 2. gps status内容
```
INFO  [gps] Main GPS
INFO  [gps] protocol: UBX
INFO  [gps] port: /dev/ttyS3, baudrate: 38400, status: OK
INFO  [gps] sat info: disabled
 vehicle_gps_position_s
    timestamp: 134147543 (0.175065 seconds ago)
    time_utc_usec: 0
    lat: 102184774
    lon: 1563467538
    alt: 629117
    alt_ellipsoid: 667558
    s_variance_m_s: 67.698
    c_variance_rad: 3.142
    eph: 4294967.500
    epv: 3750004.500
    hdop: 99.990
    vdop: 99.990
    noise_per_ms: 109
    jamming_indicator: 4
```

1. gps helper定义
> 可以明显看出是1个多态处理
```
//父类
class GPSHelper
//ubox子类

class GPSDriverUBX : public GPSHelper

//mtK原理
class GPSDriverMTK : public GPSHelper

//
class GPSDriverAshtech : public GPSHelper


//父类指针，实现多态
GPSHelper            *_helper;                    ///< instance of GPS parser

//创建子类对象
_helper = new GPSDriverUBX(_interface, &GPS::callback, this, &_report_gps_pos, _p_report_sat_info,
                                   param_gps_ubx_dynmodel);
_helper = new GPSDriverMTK(&GPS::callback, this, &_report_gps_pos);
_helper = new GPSDriverAshtech(&GPS::callback, this, &_report_gps_pos, _p_report_sat_info);
```


4. gps接收处理
> gps接收数据驱动，很迷惑人，调用有深度。
```
while ((helper_ret = _helper->receive(TIMEOUT_5HZ)) > 0 && !should_exit()) {
     ->     while (true) {
        bool ready_to_return = _configured ? (_got_posllh && _got_velned) : handled;

        /* Wait for only UBX_PACKET_TIMEOUT if something already received. */
        int ret = read(buf, sizeof(buf), ready_to_return ? UBX_PACKET_TIMEOUT : timeout);
        -> int read(uint8_t *buf, int buf_length, int timeout) {
        *((int *)buf) = timeout;
        -> return _callback(GPSCallbackType::readDeviceData, buf, buf_length, _callback_user);
          -> //poll -> read
            int GPS::callback(GPSCallbackType type, void *data1, int data2, void *user)
            case GPSCallbackType::readDeviceData: {
                    int num_read = gps->pollOrRead((uint8_t *)data1, data2, *((int *)data1));
                   -> int ret = poll(fds, sizeof(fds) / sizeof(fds[0]), math::min(max_timeout, timeout));
                   -> ret = ::read(_serial_fd, buf, buf_length);//gps
```