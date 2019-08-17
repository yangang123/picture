
C++中static_cast和dynamic_cast强制类型转换

> 在C++标准中，提供了关于类型层次转换中的两个关键字static_cast和dynamic_cast。

# 一. 明确static_cast和static_cast区别
```cpp
ParentClass *obj = new ParentClass;
Child *objChild1 = static_cast<ParentClass *>(obj);
Child *objChild2 = dynamic_cast<ParentClass *>(obj);
```
## 1.1 static_cast
static_cast转换在编译时不会报错，但是这样是不安全的，在运行时可能会有问题，因为子类中包含父类中没有的数据和函数成员，如果访问子类的成员和函数，就会出现访问越界的错误，导致程序崩溃。

## 1.2 dynamic_cast
dynamic_cast由于具有运行时类型检查功能，它能检查obj的类型，由于上述转换是不合理的，所以它返回NULL。

# 二. 实例
大名鼎鼎的px4的qGroundControl在serialLink.h 和serialLink.cc的设计

## 声明
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

## 定义
> 由于SerialLink类的构造函数需要传递SerialConfiguration类型的对象，然而config的类型确实
> LinkConfiguration是SerialConfiguration的派生类，所以需要dynamic_cast进行动态检查config的指定
> 对象是SerialConfiguration的子类对象还是LinkConfiguration的父类对象，我们通过代码确定
> LinkConfiguration在创建的时候每次可以通type进行实例化对象. 通过分析, qgroundcontrol的设计安全的
```cpp

on_typeCombo_currentIndexChanged
    -> changeLinkType(type);
       -> _config = LinkConfiguration::createSettings(type, name);
          ->  config = new SerialConfiguration(name);

 pLink = new SerialLink(dynamic_cast<SerialConfiguration*>(config));


//combox多选框内容改变
void QGCCommConfiguration::on_typeCombo_currentIndexChanged(int index)
{
    int type = _ui->typeCombo->itemData(index).toInt();
    _changeLinkType(type);
}

//创建1个link配置选项
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

