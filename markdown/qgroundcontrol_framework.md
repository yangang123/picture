
# linkmanage部分，对不同链路的管理
v2.6.0
目前支持的UDP,TCP,serial 

1. 创建单实例linkmanager
> 创建单实例是1个宏定义,这个设计还是很巧妙的，隐藏了实例的创建
```cpp
IMPLEMENT_QGC_SINGLETON(LinkManager, LinkManager)

#define IMPLEMENT_QGC_SINGLETON(className, interfaceName) \
    interfaceName* className::_instance = NULL; \
    interfaceName* className::_mockInstance = NULL; \
    interfaceName* className::_realInstance = NULL; \
    \
    interfaceName* className::_createSingleton(void) \
    { \
        Q_ASSERT(_instance == NULL); \
        _instance = new className(qgcApp()); \
        return _instance; \
    } \
    \

``` 
##  发送数据

1. 发送心跳包
```cpp
void MAVLinkProtocol::sendHeartbeat()
```
2. 发送消息
```cpp
sendMessage(beat);
```

3. 发送链路
```cpp
 link->writeBytes((const char*)buffer, len);
```

4. 发送链路的IO写函数
```cpp
void SerialLink::writeBytes(const char* data, qint64 size)
{
    if(_port && _port->isOpen()) {
        _logOutputDataRate(size, QDateTime::currentMSecsSinceEpoch());
        _port->write(data, size);
    } else {
        // Error occured
        _emitLinkError(tr("Could not send data - link %1 is disconnected!").arg(getName()));
    }
}
```

##  接收数据

1. 先找到解析mavlink包的位置
```cpp
void MAVLinkProtocol::receiveBytes(LinkInterface* link, QByteArray b)
    for (int position = 0; position < b.size(); position++) {
        unsigned int decodeState = mavlink_parse_char(mavlinkChannel, (uint8_t)(b[position]), &message, &status);
```

2. 找到那个位置调用了接收字节的方法
下面就是调用了receiveBytes的地方，这个是qt中1个最好的设计就是信号/槽函数
```cpp
src/comm/LinkManager.cc:153:    connect(link, &LinkInterface::bytesReceived, mavlink, &MAVLinkProtocol::receiveBytes);
```

3. 找到槽函数触发的地方
```cpp
void SerialLink::readBytes()
        qint64 numBytes = _port->bytesAvailable();
            emit bytesReceived(this, b);
     
    }
}
``` 

4. 找到是哪位调用了SerialLink::readBytes()
```cpp
QObject::connect(_port, &QIODevice::readyRead, this, &SerialLink::_readBytes);
```