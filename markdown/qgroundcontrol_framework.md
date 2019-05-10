
# linkmanage���֣��Բ�ͬ��·�Ĺ���
v2.6.0
Ŀǰ֧�ֵ�UDP,TCP,serial 

1. ������ʵ��linkmanager
> ������ʵ����1���궨��,�����ƻ��Ǻ�����ģ�������ʵ���Ĵ���
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
##  ��������

1. ����������
```cpp
void MAVLinkProtocol::sendHeartbeat()
```
2. ������Ϣ
```cpp
sendMessage(beat);
```

3. ������·
```cpp
 link->writeBytes((const char*)buffer, len);
```

4. ������·��IOд����
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

##  ��������

1. ���ҵ�����mavlink����λ��
```cpp
void MAVLinkProtocol::receiveBytes(LinkInterface* link, QByteArray b)
    for (int position = 0; position < b.size(); position++) {
        unsigned int decodeState = mavlink_parse_char(mavlinkChannel, (uint8_t)(b[position]), &message, &status);
```

2. �ҵ��Ǹ�λ�õ����˽����ֽڵķ���
������ǵ�����receiveBytes�ĵط��������qt��1����õ���ƾ����ź�/�ۺ���
```cpp
src/comm/LinkManager.cc:153:    connect(link, &LinkInterface::bytesReceived, mavlink, &MAVLinkProtocol::receiveBytes);
```

3. �ҵ��ۺ��������ĵط�
```cpp
void SerialLink::readBytes()
        qint64 numBytes = _port->bytesAvailable();
            emit bytesReceived(this, b);
     
    }
}
``` 

4. �ҵ�����λ������SerialLink::readBytes()
```cpp
QObject::connect(_port, &QIODevice::readyRead, this, &SerialLink::_readBytes);
```