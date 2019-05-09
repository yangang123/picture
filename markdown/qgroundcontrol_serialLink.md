
C++��static_cast��dynamic_castǿ������ת��

> ��C++��׼�У��ṩ�˹������Ͳ��ת���е������ؼ���static_cast��dynamic_cast��

# һ. ��ȷstatic_cast��static_cast����
```cpp
ParentClass *obj = new ParentClass;
Child *objChild1 = static_cast<ParentClass *>(obj);
Child *objChild2 = dynamic_cast<ParentClass *>(obj);
```
## 1.1 static_cast
static_castת���ڱ���ʱ���ᱨ�����������ǲ���ȫ�ģ�������ʱ���ܻ������⣬��Ϊ�����а���������û�е����ݺͺ�����Ա�������������ĳ�Ա�ͺ������ͻ���ַ���Խ��Ĵ��󣬵��³��������

## 1.2 dynamic_cast
dynamic_cast���ھ�������ʱ���ͼ�鹦�ܣ����ܼ��obj�����ͣ���������ת���ǲ�����ģ�����������NULL��

# ��. ʵ��
����������px4��qGroundControl��serialLink.h ��serialLink.cc�����

## ����
```cpp
class SerialConfiguration : public LinkConfiguration
{

}

class SerialLink : public LinkInterface
{
    Q_OBJECT
    
    friend class SerialConfiguration;
private:
    SerialLink(SerialConfiguration* config);
}
```

## ����
> ����SerialLink��Ĺ��캯����Ҫ����SerialConfiguration���͵Ķ���Ȼ��config������ȷʵ
> LinkConfiguration��SerialConfiguration�������࣬������Ҫdynamic_cast���ж�̬���config��ָ��
> ������SerialConfiguration�����������LinkConfiguration�ĸ����������ͨ������ȷ��
> LinkConfiguration�ڴ�����ʱ��ÿ�ο���ͨtype����ʵ��������. ͨ������, qgroundcontrol����ư�ȫ��
```cpp

on_typeCombo_currentIndexChanged
    -> changeLinkType(type);
       -> _config = LinkConfiguration::createSettings(type, name);
          ->  config = new SerialConfiguration(name);

 pLink = new SerialLink(dynamic_cast<SerialConfiguration*>(config));


//combox��ѡ�����ݸı�
void QGCCommConfiguration::on_typeCombo_currentIndexChanged(int index)
{
    int type = _ui->typeCombo->itemData(index).toInt();
    _changeLinkType(type);
}

//����1��link����ѡ��
void QGCCommConfiguration::_changeLinkType(int type)
{
    // Create new config instance
    QString name = _ui->nameEdit->text();
    if(name.isEmpty()) {
        name = tr("Untitled");
        _ui->nameEdit->setText(name);
    }
    _config = LinkConfiguration::createSettings(type, name);
}

LinkConfiguration* LinkConfiguration::createSettings(int type, const QString& name)
{
    LinkConfiguration* config = NULL;
    switch(type) {
#ifndef __ios__
        case LinkConfiguration::TypeSerial:
            config = new SerialConfiguration(name);
            break;
#endif
        case LinkConfiguration::TypeUdp:
            config = new UDPConfiguration(name);
            break;
        case LinkConfiguration::TypeTcp:
            config = new TCPConfiguration(name);
            break;
#ifdef QT_DEBUG
        case LinkConfiguration::TypeMock:
            config = new MockConfiguration(name);
            break;
#endif
    }
    return config;
}

LinkInterface* LinkManager::createConnectedLink(LinkConfiguration* config)
{
    Q_ASSERT(config);
    LinkInterface* pLink = NULL;
    switch(config->type()) {
#ifndef __ios__
        case LinkConfiguration::TypeSerial:
            pLink = new SerialLink(dynamic_cast<SerialConfiguration*>(config));
            break;
#endif
        case LinkConfiguration::TypeUdp:
            pLink = new UDPLink(dynamic_cast<UDPConfiguration*>(config));
            break;
    }
    if(pLink) {
        _addLink(pLink);
        connectLink(pLink);
    }
    return pLink;
}
```
