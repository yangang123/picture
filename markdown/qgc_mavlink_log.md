# mavlink log 
<div align="center">
<p>  </p> 
<img src="https://github.com/yangang123/picture/raw/master/qgroundcontrol/qgc_downlog.png" height="720" width="1280" > 
</div>

## qml����
����logController�ͺ�LogDownloadController�����˰󶨡�
>AnalyzeView.qml
```javascript
Rectangle {
    id:     setupView

     
    GeoTagController {
        id: geoController
    }

    LogDownloadController {
        id: logController
	}

    MavlinkConsoleController {
        id: conController
    }
}
```
>LogDownloadPage.qml
> ����4��button��reflash, download, erase, cancel
```javascript
Column {
    spacing:            _margin
    Layout.alignment:   Qt.AlignTop | Qt.AlignLeft

    QGCButton {
        enabled:   
        text:       qsTr("Refresh")
        width:      _butttonWidth

    QGCButton {

       logController.refresh()


    QGCButton {
            onAcceptedForLoad: {
                logController.download(file)

    QGCButton {
            onClicked:  logController.cancel()
```

##  cpp����

### log���صĶ��󴴽�
class LogDownloadController : public QObject
{

    Q_INVOKABLE void refresh                ();
    Q_INVOKABLE void download               (QString path = QString());
    Q_INVOKABLE void eraseAll               ();
    Q_INVOKABLE void cancel                 ();

### LogDownloadController����
```cpp
    qmlRegisterType<LogDownloadController>          (kQGCControllers,                       1, 0, "LogDownloadController");

    qmlRegisterType<GeoTagController>               (kQGCControllers,                       1, 0, "GeoTagController");

    qmlRegisterType<MavlinkConsoleController>       (kQGCControllers,                       1, 0, "MavlinkConsoleController");
```

### ����log��ʵ�ֹ���
mavlink_msg_log_request_data_pack_chan��mavlink���Ľӿ���
```cpp
LogDownloadController::download(QString path)
   downloadToDirectory(dir);//ѡ���ļ�
    -> LogDownloadController::_setDownloading(bool active)
     -> emit downloadingLogsChanged();
     -> LogDownloadController::_receivedAllData()
       -> LogDownloadController::_prepareLogDownload()
       ->_downloadData = new LogDownloadData(entry);
        -> _downloadData->file.open(QIODevice::WriteOnly) //�����ļ�
         ->    _requestLogData(_downloadData->ID, 0, _downloadData->chunk_table.size()*MAVLINK_MSG_LOG_DATA_FIELD_DATA_LEN);
        _timer.start(kTimeOutMilliseconds);
          ->  mavlink_msg_log_request_data_pack_chan(
          -> _vehicle->sendMessageOnLink(_vehicle->priorityLink(), msg);
```
LogDownloadController::_setActiveVehicle(Vehicle* vehicle)
     connect(_uas, &UASInterface::logEntry, this, &LogDownloadController::_logEntry);
        connect(_uas, &UASInterface::logData,  this, &LogDownloadController::_logData);

### �������ݵĹ���
_logData���Ǵ�mavlink���͹���
```cpp
LogDownloadController::_logData(UASInterface* uas,
    -> const QString status = QString("%1 (%2/s)").arg(QGCMapEngine::bigSizeToString(_downloadData->written),
                                                    QGCMapEngine::bigSizeToString(_downloadData->rate_avg));
    -> _downloadData->entry->setStatus(status); 
    -> if(_downloadData->file.write((const char*)data, count)) {
```