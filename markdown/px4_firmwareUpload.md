[TOC]
# 1. px4�豸�̼�����-��鲿��
qgroundcontrol����վ����px4�豸�Ǵ�qml��cpulsplus������Э��ȣ�1�����ӵ�����ܹ��������2���ֽ��⣬
һ������qml���֣���һ������cppʵ�ֲ���.

# 2. px4�豸�̼�����-qml����

## 2.1 qml�ĵ��ù�ϵ
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



##  2.2 ��FirmwareUpgradeControllerע�ᵽqml��  
```cpp 
void QGCApplication::_initCommon(void)
 -> qmlRegisterType<FirmwareUpgradeController>      (kQGCControllers,                       1, 0, "FirmwareUpgradeController");
```

## 2.3 �����ӿڹ�qml����
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

## 2.4 js����
FirmwareUpgradeController��1��������,id��controller�Ϳ��Ժ�
controller.startBoardSearch()
controller.flash(stack, firmwareType, vehicleType)


# 3. px4�豸�̼�����-cpp����
3������FirmwareUpgradeController��������������ģ�
PX4FirmwareUpgradeThreadController�ǿ���px4�����̵߳�,
PX4FirmwareUpgradeThreadWorker����������1���߳�ִ�е�

## 3.1 ����֮��Ĺ�ϵ
```cpp
FirmwareUpgradeController::FirmwareUpgradeController(void)
_threadController = new PX4FirmwareUpgradeThreadController(this);
_worker = new PX4FirmwareUpgradeThreadWorker(this); 
```

## 3.2 ��������flash
PX4FirmwareUpgradeThreadWorker::_flash��1���ۺ������ٺ�����������
���棬_controller�������ź���_flashOnThread��ͨ���Ҵ���PX4FirmwareUpgradeThreadController::flash������_flashOnThread�ź�


## 3.3 ��ѯ����
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
## 3.4 ����flash��д�̼�
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
## 3.5 ����bootloader�����_read����
Bootloader::_read������ĵĴ���
- ��1����ʱ��
    QTime timeout;
        timeout.start();
- ֱ���ж�elaspedʱ��
   if (timeout.elapsed() > readTimeout) {

- �ȴ�����
   port->waitForReadyRead(100);

- ֱ�Ӷ�����
   bytesRead = port->read((char*)&data[bytesAlreadyRead], maxSize);
```cpp

bool Bootloader::_read(QSerialPort* port, uint8_t* data, qint64 maxSize, int readTimeout)
{
    qint64 bytesAlreadyRead = 0;
    
    while (bytesAlreadyRead < maxSize) {

        //1. ��1����ʱ��
        QTime timeout;
        timeout.start();

        
        while (port->bytesAvailable() < 1) {
           
            if (timeout.elapsed() > readTimeout) {
                _errorString = tr("Timeout waiting for bytes to be available");
                return false;
            }
            //3. �ȴ�100ms
            port->waitForReadyRead(100);
        }
        // 4. ֱ�Ӷ�����
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



