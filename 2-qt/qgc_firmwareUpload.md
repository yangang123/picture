[TOC]
# 1. px4设备固件升级-简介部分
qgroundcontrol地面站升级px4设备是从qml到cpulsplus到串口协议等，1个复杂的软件架构。下面分2部分讲解，
一部分是qml部分，另一部分是cpp实现部分.

# 2. px4设备固件升级-qml部分

## 2.1 qml的调用关系
```
MainWindowHybrid.qml
 -> MainWindowInner.qml
  -> SetupView.qml
   -> FirmwareUpgrade.qml
```

```cpp
int main(int argc, char *argv[])
    -> static MainWindow::create();
      ->  _mainQmlWidgetHolder = qgcApp()->toolbox()->corePlugin()->createMainQmlWidgetHolder(_centralLayout, this);
         ->QGCQmlWidgetHolder* pMainQmlWidgetHolder = new QGCQmlWidgetHolder(QString(), nullptr, parent);
         ->pMainQmlWidgetHolder->setSource(QUrl::fromUserInput("qrc:qml/MainWindowHybrid.qml")); 

```
> MainWindowHybrid.qml
```javascript

Loader {
        id:             mainWindowInner
        anchors.fill:   parent
        source:         "MainWindowInner.qml"
        Connections {
            target: mainWindowInner.item
            onReallyClose: controller.reallyClose()
        }
    }
```
> MainWindowInner.qml
```javascript
Rectangle {
    id:     setupView
    color:  qgcPal.window


    readonly property string _setupViewSource:      "SetupView.qml"
```
>  SetupView.qml
```javascript

    function showFirmwarePanel()
    {
        if (!ScreenTools.isMobile) {
            panelLoader.setSource("FirmwareUpgrade.qml")
        }
    }

  SubMenuButton {
                id:                 firmwareButton
                imageResource:      "/qmlimages/FirmwareUpgradeIcon.png"
                setupIndicator:     false
                exclusiveGroup:     setupButtonGroup
                visible:            !ScreenTools.isMobile && _corePlugin.options.showFirmwareUpgrade
                text:               qsTr("Firmware")
                Layout.fillWidth:   true

                onClicked: showFirmwarePanel()
            }
```
> FirmwareUpgrade.qml
```javascript
   FirmwareUpgradeController {
                id:             controller
                progressBar:    progressBar
                statusLog:      statusTextArea

                property var activeVehicle: QGroundControl.multiVehicleManager.activeVehicle

                Component.onCompleted: {
                    controllerCompleted = true
                    if (qgcView.completedSignalled) {
                        // We can only start the board search when the Qml and Controller are completely done loading
                        controller.startBoardSearch()
                    }
                }
```



##  2.2 将FirmwareUpgradeController注册到qml中  
```cpp 
void QGCApplication::_initCommon(void)
 -> qmlRegisterType<FirmwareUpgradeController>      (kQGCControllers,                       1, 0, "FirmwareUpgradeController");
```

## 2.3 声明接口供qml调用
```cpp
Q_INVOKABLE void startBoardSearch(void);
    
    /// Cancels whatever state the upgrade worker thread is in
    Q_INVOKABLE void cancel(void);
    
    /// Called when the firmware type has been selected by the user to continue the flash process.
    Q_INVOKABLE void flash(AutoPilotStackType_t stackType,
                           FirmwareType_t firmwareType = StableFirmware,
                           FirmwareVehicleType_t vehicleType = DefaultVehicleFirmware );

    /// Called to flash when upgrade is running in singleFirmwareMode
    Q_INVOKABLE void flashSingleFirmwareMode(FirmwareType_t firmwareType);

    Q_INVOKABLE FirmwareVehicleType_t vehicleTypeFromVersionIndex(int index);
```

## 2.4 js部分
FirmwareUpgradeController是1个类名字,id是controller就可以和
controller.startBoardSearch()
controller.flash(stack, firmwareType, vehicleType)


# 3. px4设备固件升级-cpp部分
3个对象，FirmwareUpgradeController这个最顶层控制升级的，
PX4FirmwareUpgradeThreadController是控制px4升级线程的,
PX4FirmwareUpgradeThreadWorker升级过程是1个线程执行的

## 3.1 对象之间的关系
```cpp
FirmwareUpgradeController::FirmwareUpgradeController(void)
_threadController = new PX4FirmwareUpgradeThreadController(this);
_worker = new PX4FirmwareUpgradeThreadWorker(this); 
```

## 3.2 分析升级flash
PX4FirmwareUpgradeThreadWorker::_flash是1个槽函数，操函数的连接在
下面，_controller发出的信号是_flashOnThread，通过找代码PX4FirmwareUpgradeThreadController::flash发出的_flashOnThread信号


## 3.3 查询板子
```cpp
 controller.startBoardSearch() 
   -> FirmwareUpgradeController::startBoardSearch(void)
       -> linkMgr->disconnectAll();
       -> PX4FirmwareUpgradeThreadController::startFindBoardLoop(void);
           ->  emit _startFindBoardLoopOnThread();
              -> connect(_controller, &PX4FirmwareUpgradeThreadController::_startFindBoardLoopOnThread,  
                  this, &PX4FirmwareUpgradeThreadWorker::_startFindBoardLoop);
                  -> PX4FirmwareUpgradeThreadWorker::_startFindBoardLoop
                    -> PX4FirmwareUpgradeThreadWorker::_findBoardOnce
                      -> PX4FirmwareUpgradeThreadWorker::_findBootloader
                        ->  _bootloaderPort = new QSerialPort();
                        -> if (_bootloader->open(_bootloaderPort, portInfo.systemLocation())) {
                        -> if (_bootloader->sync(_bootloaderPort)) {
                              -> if (_sendCommand(port, PROTO_GET_SYNC)) {
                                 ->  if (!_write(port, buf, 2)) {
                                 ->  if (!_getCommandResponse(port, responseTimeout)) {
                                      -> if (!_read(port, response, 2, responseTimeout)) {
                                           -> bytesRead = port->read((char*)&data[bytesAlreadyRead], maxSize);
```
## 3.4 进行flash烧写固件
```cpp
controller.flash(stack, firmwareType, vehicleType)
  -> _FirmwareUpgradeController::flash(AutoPilotStackType_t stackType,FirmwareType_t,FirmwareVehicleType_t)
     ->  _getFirmwareFile(firmwareId);
       ->  _downloadFirmware();
          -> QGCFileDownload* downloader = new QGCFileDownload(this);
             connect(downloader, &QGCFileDownload::downloadFinished, this, &FirmwareUpgradeController::_firmwareDownloadFinished);
             -> void FirmwareUpgradeController::_firmwareDownloadFinished(QString remoteFile, QString localFile)
                -> FirmwareImage* image = new FirmwareImage(this);
                -> _threadController->flash(image);
                        ->void PX4FirmwareUpgradeThreadController::flash(const FirmwareImage* image)
                            ->connect(_controller, &PX4FirmwareUpgradeThreadController::_flashOnThread,               this, &PX4FirmwareUpgradeThreadWorker::_flash);
                            -> PX4FirmwareUpgradeThreadWorker::_flash()
                            if (_erase()) {
                                if (_bootloader->program(_bootloaderPort, _controller->image())) {
                                 emit _reboot();
                                 emit flashComplete();
```
## 3.5 分析bootloader对象的_read函数
Bootloader::_read是最核心的代码
- 打开1个定时器
    QTime timeout;
        timeout.start();
- 直接判断elasped时间
   if (timeout.elapsed() > readTimeout) {

- 等待数据
   port->waitForReadyRead(100);

- 直接读数据
   bytesRead = port->read((char*)&data[bytesAlreadyRead], maxSize);
```cpp

bool Bootloader::_read(QSerialPort* port, uint8_t* data, qint64 maxSize, int readTimeout)
{
    qint64 bytesAlreadyRead = 0;
    
    while (bytesAlreadyRead < maxSize) {

        //1. 打开1个定时器
        QTime timeout;
        timeout.start();

        
        while (port->bytesAvailable() < 1) {
           
            if (timeout.elapsed() > readTimeout) {
                _errorString = tr("Timeout waiting for bytes to be available");
                return false;
            }
            //3. 等待100ms
            port->waitForReadyRead(100);
        }
        // 4. 直接读数据
        qint64 bytesRead;
        bytesRead = port->read((char*)&data[bytesAlreadyRead], maxSize);
        
        if (bytesRead == -1) {
            _errorString = tr("Read failed: error: %1").arg(port->errorString());
            return false;
        } else {
            Q_ASSERT(bytesRead != 0);
            bytesAlreadyRead += bytesRead;
        }
    }
    
    return true;
}

```



