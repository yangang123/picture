介绍Qt中的signal和slot连接方式，同时介绍他们在多线程中的使用

直接连接Qt::DirectConnection
队列连接Qt::QueuedConnection
自动连接Qt::AutoConnection

## 直接连接
1. Widget.cpp源代码内容
```
namespace Ui {
class Widget;
}

class Widget : public QWidget
{
    Q_OBJECT

public:
    explicit Widget(QWidget *parent = nullptr);
    ~Widget();

signals:
    void sendStr();

private:
    Ui::Widget *ui;
    QPushButton* pushButton;

    SubThread subThread;

public slots:
    void on_click_button();

};

Widget::Widget(QWidget *parent) :
    QWidget(parent),
    ui(new Ui::Widget)
{
    ui->setupUi(this);

    qDebug() << "widget thread id:" << QThread::currentThreadId();
    pushButton = new QPushButton(this);
    pushButton->setText(QString("button"));

    connect(pushButton, &QPushButton::clicked, this, &Widget::on_click_button);
    connect(this, &Widget::sendStr, &subThread, &SubThread::showStr, Qt::AutoConnection);
    subThread.start();
}

Widget::~Widget()
{
    delete ui;
}

void Widget::on_click_button()
{
   qDebug() << "send signal thread id:" << QThread::currentThreadId();
   emit sendStr();
}
```
2. SubThread源代码内容如下
```
class SubThread : public QThread
{
public:
    SubThread();
public slots:
    void showStr();
};

SubThread::SubThread()
{

}

void SubThread::showStr()
{
    qDebug() << "slot run id : "  << QThread::currentThreadId() ;
}
```

3. 执行结果
3.1 直接连接
 connect(this, &Widget::sendStr, &subThread, &SubThread::showStr, Qt::DirectConnection);
3.2 队列连接
 connect(this, &Widget::sendStr, &subThread, &SubThread::showStr, Qt::QueuedConnection);
3.3 自动连接
 connect(this, &Widget::sendStr, &subThread, &SubThread::showStr, Qt::AutoConnection); 

```
widget thread id: 0x7f7c30d69780
send signal thread id: 0x7f7c30d69780
slot run id :  0x7f7c30d69780
```

## 更改线程方式
1. SubThread.cp2
把当前对对象放到子线程中
```
SubThread::SubThread()
{
    moveToThread(this);
}
```
2. 执行结果
2.1 直接连接
 connect(this, &Widget::sendStr, &subThread, &SubThread::showStr, Qt::DirectConnection);
```
widget thread id: 0x7f8916374780
send signal thread id: 0x7f8916374780
slot run id :  0x7f8916374780
```
slot是在主线程中执行的
2.2 队列连接
 connect(this, &Widget::sendStr, &subThread, &SubThread::showStr, Qt::QueuedConnection);
2.3 自动连接
 connect(this, &Widget::sendStr, &subThread, &SubThread::showStr, Qt::AutoConnection); 
```
widget thread id: 0x7f8916374780
send signal thread id: 0x7f8916374780
slot run id :  0x7f88fab5e700
```
slot是在子线程中执行的

## 重写run接口
1. 代码片段
```
SubThread::SubThread()
{
    moveToThread(this);
}

void SubThread::showStr()
{
    qDebug() << "slot run id : "  << QThread::currentThreadId() ;
}

void SubThread::run()
{

}
```
2. 执行结果
2.1 直接连接
 connect(this, &Widget::sendStr, &subThread, &SubThread::showStr, Qt::DirectConnection);
```
widget thread id: 0x7f8916374780
send signal thread id: 0x7f8916374780
slot run id :  0x7f8916374780
```
slot是在主线程中执行的
2.2 队列连接
 connect(this, &Widget::sendStr, &subThread, &SubThread::showStr, Qt::QueuedConnection);
2.3 自动连接
 connect(this, &Widget::sendStr, &subThread, &SubThread::showStr, Qt::AutoConnection); 
```
widget thread id: 0x7f8916374780
send signal thread id: 0x7f8916374780
```
slot没有执行

## 移除movetothread
```
SubThread::SubThread()
{
    //moveToThread(this);
}

void SubThread::showStr()
{
    qDebug() << "slot run id : "  << QThread::currentThreadId() ;
}

void SubThread::run()
{
    exec();
}
```
2. 执行结果
2.1 直接连接
 connect(this, &Widget::sendStr, &subThread, &SubThread::showStr, Qt::DirectConnection);
2.2 队列连接
 connect(this, &Widget::sendStr, &subThread, &SubThread::showStr, Qt::QueuedConnection);
2.3 自动连接
 connect(this, &Widget::sendStr, &subThread, &SubThread::showStr, Qt::AutoConnection); 

```
widget thread id: 0x7f7c30d69780
send signal thread id: 0x7f7c30d69780
slot run id :  0x7f7c30d69780
```

## 好的设计1
1. 代码片段
```
namespace Ui {
class Widget;
}

class SubThreadRecevier;

class Widget : public QWidget
{
    Q_OBJECT

public:
    explicit Widget(QWidget *parent = nullptr);
    ~Widget();

signals:
    void sendStr();

private:
    Ui::Widget *ui;
    QPushButton* pushButton;
    QThread* qthread;
    SubThread subThread;

public slots:
    void on_click_button();

};

#include "widget.h"
#include "ui_widget.h"

Widget::Widget(QWidget *parent) :
    QWidget(parent),
    ui(new Ui::Widget)
{
    ui->setupUi(this);

    qDebug() << "widget thread id:" << QThread::currentThreadId();
    pushButton = new QPushButton(this);
    pushButton->setText(QString("button"));
    connect(pushButton, &QPushButton::clicked, this, &Widget::on_click_button);
    
    qthread = new QThread();
    subThread.moveToThread(qthread);
    connect(this, &Widget::sendStr, &subThread, &SubThread::showStr, Qt::AutoConnection);
    qthread->start();
}

Widget::~Widget()
{
    delete ui;
}

void Widget::on_click_button()
{
   qDebug() << "send signal thread id:" << QThread::currentThreadId();
   emit sendStr();
}

```
2. 执行结果
2.1 直接连接
 connect(this, &Widget::sendStr, &subThread, &SubThread::showStr, Qt::DirectConnection);
```
widget thread id: 0x7f8916374780
send signal thread id: 0x7f8916374780
slot run id :  0x7f8916374780
```
slot是在主线程中执行的
2.2 队列连接
 connect(this, &Widget::sendStr, &subThread, &SubThread::showStr, Qt::QueuedConnection);
2.3 自动连接
 connect(this, &Widget::sendStr, &subThread, &SubThread::showStr, Qt::AutoConnection); 
```
widget thread id: 0x7f8916374780
send signal thread id: 0x7f8916374780
slot run id :  0x7f88fab5e700
```
slot是在子线程中执行的

## 好的设计2
```
class SubThread2 : public QThread
{
    Q_OBJECT
public:
    SubThread2();
    void run();

public slots:
    void showStr();
};

#include "subthread2.h"

SubThread2::SubThread2()
{

}

// can`t executed in subthread
void SubThread2::showStr()
{
    qDebug() << "slot run id : "  << QThread::currentThreadId() ;
}

void SubThread2::run()
{

}


Widget::Widget(QWidget *parent) :
    QWidget(parent),
    ui(new Ui::Widget)
{
    ui->setupUi(this);

    qDebug() << "widget thread id:" << QThread::currentThreadId();
    pushButton = new QPushButton(this);
    pushButton->setText(QString("button"));
    connect(pushButton, &QPushButton::clicked, this, &Widget::on_click_button);

    connect(this, &Widget::sendStr, &subThread2, &SubThread2::showStr, Qt::QueuedConnection);
    subThread2.start();
}

```
2. 执行结果
2.1 直接连接
 connect(this, &Widget::sendStr, &subThread, &SubThread::showStr, Qt::DirectConnection);
2.2 队列连接
 connect(this, &Widget::sendStr, &subThread, &SubThread::showStr, Qt::QueuedConnection);
2.3 自动连接
 connect(this, &Widget::sendStr, &subThread, &SubThread::showStr, Qt::AutoConnection); 

```
widget thread id: 0x7f7c30d69780
send signal thread id: 0x7f7c30d69780
slot run id :  0x7f7c30d69780
```

# 总结

subThread::QThread,然后把subThread通过movetothread(this), 只要subThread重写了run方法，就需早run中调用exec才可以执行solt.
推荐使用设计1,自由度比较大，slot槽函数可以在线程中执行，设计2,slot只能在主线程中执行，slot函数中简短，同时要和子线程做好线程安全处理。
