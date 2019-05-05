
qgroundcontrol开发者文档中说明了qgc中的各个链路流向在文档中说明的很清楚，下面配套源代码进行讲解整个qgc地面站的数据流向过程.

<div align="center">
<p>  </p> 
<img src="https://github.com/yangang123/picture/raw/master/qgroundcontrol/qgc_0504_02.png" height="720" width="1280" > 
</div>

# qgroundcontrol通信
在 https://dev.qgroundcontrol.com/en/communication_flow.html 中描述如下
```
Description of the high level communication flow which takes place during a vehicle auto-connect.

    LinkManager always has a UDP link open waiting for a Vehicle heartbeat
    LinkManager detects a new known device (Pixhawk, SiK Radio, PX4 Flow) connected to computer
        Creates a new SerialLink connected to the device
    Bytes comes through Link and are sent to MAVLinkProtocol
    MAVLinkProtocol converts the bytes into a MAVLink message
    If the message is a HEARTBEAT the MultiVehicleManager is notified
    MultiVehicleManager is notifed of the HEARTBEAT and creates a new Vehicle object based on the information in the HEARTBEAT message
    The Vehicle instantiates the plugins which match the vehicle type
    The ParameterLoader associated with the vehicle sends a PARAM_REQUEST_LIST to the vehicle to load params using the parameter protocol
    Once parameter load is complete, the MissionManager associated with the Vehicle requests the mission items from the Vehicle using the mission item protocol
    Once parameter load is complete, the VehicleComponents display their UI in the Setup view
```

# qgroundcontrol实现

### qgroundcontrol进程中线程

1. 搜索QGC地面站的进程ID
```
yangang@yangang-ubuntu:~/work/github_proj/picture$ ps -ef |grep "QG" |grep -v "grep"
yangang   8353  8340  6 14:06 ?        00:00:01 /home/yangang/work/github_proj/build-qgroundcontrol-Desktop_Qt_5_11_3_GCC_64bit-Debug/debug/QGroundControl -qmljsdebugger=port:35601,block,services:DebugMessages,QmlDebugger,V8Debugger,QmlInspector
```
2. 查找QGC进程中的线程
其实整个GGC的进程中，GUI是1个线程，UDPLINK是1个独立的线程
```
yangang@yangang-ubuntu:~/work/github_proj/picture$ ps -T -p 8353
  PID  SPID TTY          TIME CMD
 8353  8353 ?        00:00:01 QGroundControl
 8353  8368 ?        00:00:00 QXcbEventReader
 8353  8369 ?        00:00:00 gmain
 8353  8370 ?        00:00:00 gdbus
 8353  8386 ?        00:00:00 QGroundControl
 8353  8414 ?        00:00:00 QDBusConnection
 8353  8415 ?        00:00:00 Qt bearer threa
 8353  8416 ?        00:00:00 QGCCacheWorker
 8353  8417 ?        00:00:00 Thread (pooled)
 8353  8418 ?        00:00:00 QQmlThread
 8353  8419 ?        00:00:00 QQmlDebugServer
 8353  8423 ?        00:00:00 disk_cache:0
 8353  8426 ?        00:00:00 UDPLink
 8353  8427 ?        00:00:00 QNetworkAccessM
 8353  8457 ?        00:00:00 QNetworkAccessM
```
1. UDPlink对象创建过程
在main入口中创建QGCApplication对象，在对象的构造函数中创建QGCtoolbox，
注册一个定时器事件，用来自动连接设备，UDPLink是默认启动的，创建过程如下
```
int main(int argc, char *argv[])
->QGCApplication* app = new QGCApplication(argc, argv, runUnitTests);
   -> _toolbox = new QGCToolbox(this);
   ->  _toolbox->setChildToolboxes();
        -> _linkManager->setToolbox(this);
            ->  connect(_mavlinkProtocol, &MAVLinkProtocol::messageReceived, this, &LinkManager::_mavlinkMessageReceived);
            ->  connect(&_portListTimer, &QTimer::timeout, this, &LinkManager::_updateAutoConnectLinks);
            _portListTimer.start(1000); 
          
->exitCode = app->exec();
     -> void LinkManager::_updateAutoConnectLinks(void)
               -> if (!foundUDP && _autoConnectSettings->autoConnectUDP()->rawValue().toBool()) {
               ->  SharedLinkConfigurationPointer config = addConfiguration(udpConfig);
               -> createConnectedLink(config);
                   case LinkConfiguration::TypeUdp:
        		pLink = new UDPLink(config);
```

## UDPLINK工作原理
1. 连接完成，进行事件循环
```
UDPLink::UDPLink(SharedLinkConfigurationPointer& config)
    -> moveToThread(this);
```
2. 通过连接，进行启动线程
```
bool UDPLink::_connect(void)
    start(NormalPriority);
```
3. 在run中，连接完成后，进行exec循环
```
void UDPLink::run()
    if(_hardwareConnect()) {
        exec();
    }
```
4. UDP信号多线程执行
测试LinkInterface::bytesReceived信号的发生，MAVLinkProtocol::receiveBytes的slot执行线程
```
main: 0x7efe09670800
Adding target QHostAddress("127.0.0.1") 14556
UDPLink: 0x7efdb246e700
UDPLink: 0x7efdb246e700
MAVLinkProtocol: 0x7efe09670800
Switching outbound to mavlink 2.0 due to incoming mavlink 2.0 packet: 0x55a77c48ca48 1 2
MAVLinkProtocol: 0x7efe09670800
```
## SerialLink工作原理
1. 串口link的数据收发都是使用signal和slot进行处理的
```
bool SerialLink::_connect(void)
if (!_hardwareConnect(error, errorString)) {
    -> _port = new QSerialPort(_serialConfig->portName(), this);
    -> QObject::connect(_port, &QIODevice::readyRead, this, &SerialLink::_readBytes);
      -> _port->read(buffer.data(), buffer.size());
        emit bytesReceived(this, buffer);

void SerialLink::_writeBytes(const QByteArray data)
  ->  _port->write(data);
```

2. 测试LinkInterface::bytesReceived信号的发生，MAVLinkProtocol::receiveBytes的slot执行线程
```
main:id 0x7fb3aca56800
"v3.5.2"
QGCMessageBox (unit testing) "New Version Available" "There is a newer version of QGroundControl available. You can download it from qgroundcontrol.com."
MAVLINK:id 0x7fb3aca56800
Switching outbound to mavlink 2.0 due to incoming mavlink 2.0 packet: 0x564613ce9a70 2 2
serialLINK:id 0x7fb3aca56800
MAVLINK:id 0x7fb3aca56800
```

## mavlink解析
字节的收发和MAVLinkProtocol的槽函数进行连接，
void LinkManager::_addLink(LinkInterface* link)
  -> connect(link, &LinkInterface::bytesReceived,        _mavlinkProtocol,   &MAVLinkProtocol::receiveBytes);

   -> void MAVLinkProtocol::receiveBytes(LinkInterface* link, QByteArray b)
     ->  uint8_t mavlinkChannel = link->mavlinkChannel();
        ->  if (mavlink_parse_char(mavlinkChannel, static_cast<uint8_t>(b[position]), &_message, &_status)) {
            -> emit messageReceived(link, _message);

# qgroundcontrol总结
qgroundcontrol设计很优秀。

