# C-_QT
QT开发学习内容,请下载PDF观看图片
# QT开发

## QT的简单介绍：

一个跨平台的GUI库，包含了C++的内容以及自己定义的库等等。

## QT CREATOR的常用快捷键

![image-20231102205924371](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231102205924371.png)

![image-20231102205941520](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231102205941520.png)

![image-20231102205958524](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231102205958524.png)

## P1_1.主事件循环

![image-20231102210403019](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231102210403019.png)

这里是层级的继承关系，越往下功能越强大

* QCoreApplication:为非GUI应用提供**主事件循环**

* QGuiApplication:为GUI应用程序提供**主事件循环**

* QApplication:为Qt Widgets模块的应用程序提供**主事件循环**

### QCoreApplication

  QCoreApplication包含**主事件循环**，处理和分发来自操作系统和其他源的所有事件。它还处理用用程序的初始化和终结，以及系统范围和应用程序范围的设置



比如下列代码

~~~c++
#include"mainwindow.h"
#include<QCoreApplication>
int main(int argc, char *argv[])
{
    QApplication a(argc, argv);
    return a.exec();
}
~~~

这里进入主事件循环并等待，直到调用exit()。返回传递给exit的值。



### QGuiApplication

除了继承上面的功能之外，还负责初始化GUI所需的资源；跟踪系统界面属性，保证GUI与系统设置保持一致：提供字符串本地化、剪贴板、鼠标光标处理等功能

~~~C++
#include"mainwindow.h"
#include<QtGui>
#include<QWindow>
int main(int argc, char *argv[])
{
    QApplication a(argc, argv);
	Qwindow w;
	w.show();
    return a.exec();
}
~~~



### QApplication

除了继承上面的功能之外，还负责初始化Qt Widgets模块所需的资源，并提供更多接口

~~~C++
#include"mainwindow.h"
#include<QApplication>
#include<QWidget>
{
    QApplication a(argc, argv);
	QWidget w;
	w.show();
    return a.exec();
}
~~~



## P2_2.moc与QObject

### moc

* Qt对标准C++进行了扩展，引入了新的概念和功能

  * 信号槽机制

  * 属性

  * 内存管理

  * ......
* QObject类是所有使用元对象系统的类的积累

  * 并不是所有Q开头的类都是Object的派生类，例如QString
  * 在一个类的private部分声明Q_OBJECT宏
  * MOC（元对象编译器）为每个QObject的子类提供必要的代码

> 元对象编译器（Meta-Object Compiler,MOC)是一个预处理器，先将Qt的特性程序转换为标准C++程序，再由标准C++编译器进行编译
>
> 使用信号与槽机制，只有添加Q_OBJECT宏，moc才能对类里的信号与槽进行预处理
>
> Qt为C++语言增加的特性在Qt Core模块里实现，由Qt的元对象系统实现。包括：信号与槽机制、属性系统、动态类型转换等。
>
> 元对象系统是一个C++扩展，使该语言更适合真正的组件化GUI编程

![image-20231102213817337](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231102213817337.png)

### QObject

* QObject是Qt对象模型的核心

  * 标准的C++对象模型在某些领域不够灵活，GUI编程是一个既需要效率又需要高度灵活性的领域。QObject提供了灵活性

* QObject不支持拷贝

  * QObject的拷贝函数和赋值运算符是私有的，并且使用了Q_DISABLE_COPY()宏

    如果支持拷贝，需要考虑这么多事情：

    * 可能具有唯一的QObject::objectName()。如果我们复制一个对象，应该给副本起什么名字？
    * 在对象层次结构中具有位置。如果我们复制一个Qt对象，那么副本应该位于哪里？
    * 可以连接到其他Qt对象以向其发射信号或接收其发射的信号。如果复制Qt对象，应该如何将这些连接传输到副本
    * 可以在运行时向其添加未在C++类中声明的新属性。如果我们复制一个Qt对象，那么副本是否应该包括添加到原始对象中的属性？

* QObject在对象树中组织自己

  * 当以另一个对象作为父对象创建QObject时，该对象将自动将自身添加到父对象的子对象列表中。
  * 父对象删除时，它将自动删除其子对象。可以使用findChild()或findChildren()按名称和可选类型查找对象![image-20231104164145780](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231104164145780.png)

  如果只是需要通过QObject自动回收内存，无需moc,也就无需添加QObject,虽然不是必须的，但是强烈在所有的QObject子类中加上Q_OBJECT宏。

如果用一个类去继承QObject类，因为采用了对象树的缘故，所以在析构的时候，先把父结点给释放，再释放子节点。

从QObject继承的子类的拷贝构造函数被隐式声明为delete,因为基类QObject通过一个宏声明为了delete,所以拷贝构造和赋值运算符是无法使用的

## P3_2.2事件与信号

> 所有GUI应用程序都是事件驱动的。事件主要由应用程序的用户生成，但也可以通过其他方式生成，例如Internet连接、窗口管理器或计时器。当调用exec方法时，应用程序进入主循环。主循环获取事件并将其发送到对象。

### 信号与槽

* Qt具有独特的信号和槽机制。这种信号与插槽机制是对C++的扩展

* 信号和槽用于对象之间的通信

* slot是一种普通的C++函数；当与之相连的信号发出时，调用它。

  ![image-20231104164951729](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231104164951729.png)
  有几种连接信号和插槽的方法

* 使用成员函数指针

  * 优点：1.允许编译器检查信号是否与槽的参数兼容
  * 2.编译器可以隐式转换参数
    `connect(sender,&QObject::destoryed,this,&MyObject::objectDestroyed);`

* 使用仿函数或者lambda表达式作为slot

  `connect(sender,&QObject::destoryed,this,[=](){this->m_object.remove(sender);});`

* 使用SIGNAL和SLOT宏

  * 如果参数具有默认值，传递给SIGNAL()宏签名的参数不得少于传递给SLOT()宏的签名的参数、

    `connect(sender,SIGNAL(destroyer(QObject*)),this,SLOT(objectDestroyed(QObject*)));`

    `connect(sender,SIGNAL(destroyer(QObject*)),this,SLOT(objectDestroyed()));`

    `connect(sender,SIGNAL(destroyer()),this,SLOT(objectDestroyed()));`

​		<u>**`connect(sender,SIGNAL(destroyer()),this,SLOT(objectDestroyed(QObject*)));//运行时错误`**</u>

> connect还可以添加Qt::ConnectionType类型的参数，表示信号与槽之间的关联方式
>
> `static QMetaObject::Connection`
>
> `connect(const QObject *sender,const char* *signal,const QObject* receiver,const char*member,Qt::ConnectionTYpe=Qt::AutoConnection)`
>
> 

* Qt::AutoConnection(缺省值):自动确定关联方式
* Qt::DirectConnection:信号被发射时，槽立即执行，槽函数与信号在同一线程
* Qt::QueuedConnection:事件循环回到接收者线程后执行槽，槽与信号在不同线程
* Qt::BlockingQueueConnection:与QT::QueueConnection相似，信号线程会被阻塞直到槽执行完毕。当槽函数与信号在同一线程，会造成死锁。

在槽函数里，使用QObject::sender()可以获取信号发射者的指针

QSpinBox*spinbox=qobject_castr<QSpinBox*>(sender());

信号与槽还有一种自动关联方式，例如on_pushButton_clicked(),uic自动生成的setupUi函数中调用connectSlotByNamea()函数

## P4_2.3自定义信号和槽

信号函数必须无返回值，但可以有输入参数。信号函数无需实现，只需在某些条件下发射信号

~~~c++
class Sender:public QObject{
Q_OBJECT
public:
	explicit Sender(QObject*parent=nullptr);
private:
	int m_age=10;
public:
	void incAge();
	signals://信号函数只需要声明，由moc来做出实现
	void ageChanged(int value);	
};

void Sender::incAge(){m_age++;emit ageChanged(m_age);}//发送这个函数

class Receiver:public QObject{
public:
    explicit Receiver(QObject*parent=nullptr);
    void ageChanged(int age);
}

void Receiver::ageChanged(int age)
{
    qDebug()<<"age:"<<age;
}


#include"sender.h"
#include"receiver.h"
#include <QApplication>

int main(int argc, char *argv[])
{
    Sender senderObj;
    Receiver receiverObj;
    senderObj.incAge();
    QObject::connect(&senderObj,&Sender::ageChanged,&receiverObj,&Receiver::ageChanged);
    senderObj.incAge();
    senderObj.incAge();
    QObject::disconnect(&senderObj,&Sender::ageChanged,&receiverObj,&Receiver::ageChanged);
    senderObj.incAge();
}
~~~

这里，开始连接之前，虽然age增长了，但是由于未连接，所以不会被打印

在断开连接之后，虽然age增长了，但是由于未连接，所以不会被打印

![image-20231105100817758](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231105100817758.png)

或者是调用receiverObj的析构函数也会是一样的结果，使得连接断开

## P5_2.4鼠标键盘事件

~~~c++
#ifndef MAINWINDOW_H
#define MAINWINDOW_H

#include <QMainWindow>

class MainWindow : public QMainWindow
{
    Q_OBJECT

public:
    MainWindow(QWidget *parent = nullptr);
    ~MainWindow();

private:
    Ui::MainWindow *ui;

protected:
    void keyPressEvent(QKeyEvent *event);//键盘触发事件
    void mouseMoveEvent(QMouseEvent *event);//鼠标触发事件
};
#endif // MAINWINDOW_H

~~~



~~~C++
#include "mainwindow.h"
#include "./ui_mainwindow.h"
#include <QtWidgets>
MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
{
    auto*quitbtn=new QPushButton("Quit",this);	
    quitbtn->setGeometry(50,25,100,50);
    connect(quitbtn,&QPushButton::clicked,qApp,&QApplication::quit);//鼠标点击发出信号，连接到退出的槽
}

MainWindow::~MainWindow()
{

}

void MainWindow::keyPressEvent(QKeyEvent *event)
{
    if(event->key()==Qt::Key_Escape)//输入空格就退出
        qApp->quit();
}

void MainWindow::mouseMoveEvent(QMouseEvent *event)
{
    int x=event->pos().x();
    int y=event->pos().y();
    QString text="坐标:"+QString::number(x)+","+QString::number(y);
    this->statusBar()->showMessage(text);
}

~~~

这段代码，设置了一个简单的窗口，可以通过鼠标点击来获取窗口上的坐标位置，可以通过按quit键或是按空格的方式退出程序。

然而一直通过鼠标点击的方式获取坐标十分的不灵活，所以只需要在里面加上一行一直获取鼠标位置信息的代码就行了。

~~~C++
MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
{
    setMouseTracking(true);//时刻获取鼠标的位置信息
    auto*quitbtn=new QPushButton("Quit",this);
    quitbtn->setGeometry(50,25,100,50);
    connect(quitbtn,&QPushButton::clicked,qApp,&QApplication::quit);
}
~~~

## P6_2.5控件与自定义槽

QMainWindow有自己的布局。布局有一个中心区域，可以被任何类型的widget占用

![image-20231105105316624](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231105105316624.png)

~~~C++
#ifndef MAINWINDOW_H
#define MAINWINDOW_H

#include <QMainWindow>
//这里由于没有包含头文件会报错，但是最好的解决方案并不是包含头文件，而是直接声明使得编译通过，因为在C++20之前没有import，只有include,而include实际上就是copy and paste。所以说，包含头文件这样的做法会降低编译速度，不利于性能.
class QPushButton;
class QLabel;
class MainWindow : public QMainWindow
{
    Q_OBJECT

public:
    MainWindow(QWidget *parent = nullptr);
    ~MainWindow();

protected:
   void timerEvent(QTimerEvent *event);
private:
   void onClick();
   void onCheck(int state);//根据是否勾选决定连接与断连
private:
   QPushButton * clickBtn;
   QLabel *lable;
Q_SIGNALS:
   void stateChanged(int);//检测check的状态
};
#endif // MAINWINDOW_H

~~~



~~~C++
#include "mainwindow.h"
#include "./ui_mainwindow.h"
#include<QtWidgets>

MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent){
    QWidget*widget=new QWidget(this);
    setCentralWidget(widget);
    clickBtn=new QPushButton("点击",widget);
    QCheckBox*cb=new QCheckBox("connect",widget);
    cb->setCheckState(Qt::Checked);//默认把√打上
    lable=new QLabel(QTime::currentTime().toString(),widget);//在widget内创建一个文字，显示实时的时间

    QHBoxLayout*hbox=new QHBoxLayout(widget);
    hbox->addWidget(clickBtn);
    hbox->addWidget(cb);
    hbox->addWidget(lable);

    startTimer(1000);
    //连接点击按钮和显示按钮被点击的模块
    connect(clickBtn, &QPushButton::clicked, this, &MainWindow::onClick);
    //连接勾选栏状态和控制连接的模块
    connect(cb, &QCheckBox::stateChanged, this, &MainWindow::onCheck);
}

MainWindow::~MainWindow(){}

void MainWindow::timerEvent(QTimerEvent *event){
    //告知编译器这个event不需要了，消除编译器产生的警告
    Q_UNUSED(event);
    lable->setText(QTime::currentTime().toString());
}

void MainWindow::onClick()
{
    statusBar()->showMessage("按钮被点击");//状态栏显示
}

void MainWindow::onCheck(int state){
    statusBar()->showMessage("");
    if(state == Qt::Checked) {
        connect(clickBtn, &QPushButton::clicked, this, &MainWindow::onClick);
    } else {
        disconnect(clickBtn, &QPushButton::clicked, this, &MainWindow::onClick);
    }
}

}
~~~

## P7_2.6属性系统

* 要声明属性，需要在继承QObject的类中使用Q_PROPERTY()的宏
* Q_PROPERTY宏定义一个返回类型为type,名称为name的属性

![image-20231105145657252](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231105145657252.png)

![image-20231105150257363](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231105150257363.png)

通过在运行时获取类的属性，可以动态地遍历关于这个对象的所有信息

也可以在运行的时候为类定义新的属性，并且可以直接使用。

~~~C++
#ifndef MYCLASS_H
#define MYCLASS_H

#include <QObject>

class MyClass : public QObject
{
    Q_OBJECT
    Q_PROPERTY(Priority priority READ priority WRITE setPriority NOTIFY priorityChanged)
    Q_CLASSINFO("author","Arxibye")
    Q_CLASSINFO("Version","6.6.6")
public:
    enum Priority{High,low,VeryHigh,VeryLow};
    Q_ENUM(Priority)//使枚举值作为字符串，以便在调用setProperty()时使用

    void setPriority(Priority priority){
        m_priority=priority;
        emit priorityChanged(priority);
    }

    Priority priority()const{return m_priority;}
    explicit MyClass(QObject *parent = nullptr);

signals:
    void priorityChanged(MyClass::Priority);
private:
    Priority m_priority;
};

#endif // MYCLASS_H
~~~

~~~C++
#include "mainwindow.h"
#include<iostream>
#include <QApplication>
#include"myclass.h"
#include "qmetaobject.h"
int main(int argc, char *argv[])
{
    QApplication a(argc, argv);
    MyClass*myinstance=new MyClass;
    QObject*object1=myinstance;

    myinstance->setPriority(MyClass::VeryLow);
    object1->setProperty("priority","VeryHigh");
    object1->setProperty("isGood",false);

    qInfo()<<myinstance->priority();
    qInfo()<<myinstance->property("isGood");
    qInfo()<<myinstance->metaObject()->classInfo(0).name();
    qInfo()<<myinstance->metaObject()->classInfo(1).value();
    qInfo()<<myinstance->metaObject()->classInfo(1).name();
    qInfo()<<myinstance->metaObject()->classInfo(1).value();
    return a.exec();
}
~~~

![image-20231105153456832](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231105153456832.png)

### 使用智能指针完善程序

* QSharedPointer是C++中的一个自动共享指针。它的行为与普通指针完全相同，包括const约束的使用。

* 如果没有其他的QSharedPointer对象引用它，当它超出范围时，QSharedPointer将删除它所持有的指针（即回收指向的内存)

* QSharedPointer对象可以从普通指针、另一个QSharedPointer对象或通过QWeakPointer对象提升为强引用创建

* 在我看来，QSharedPointer其实就是Qt对于C++11中的std::shared_ptr的Qt适配版本，就像std::string和QString,std::thread和QThread的关系一样。因为我发现QSharedPointer和shared_ptr可以发生互相的转换，并且没有发生任何异常现象

  * QSharedPointer转shared_ptr

  ~~~c++
  QSharedPointer<Image> image;
  image* image2=image.data();//先获取原对象指针
  shared_ptr<Image> image3(image2);//再创建新指针
  ~~~

  * shared_ptr转QSharedPtr  

~~~c++
QSharedPointer<Image> image;
image* image2=image.data();//先获取原对象指针
shared_ptr<Image> image3(image2);//再创建新指针

~~~



#### 创建智能指针

`QSharedPointer<MyClass>myinstance=QSharedPointer<MyCLass>(new MyClass);`

`QObject*object=myinstance.get();`//通过get获取裸指针，不过delete这个左边object指针的时候会存在引用计数的安全隐患

#### 创建一个按'Q'退出的线程

~~~c++
#include<QThread>
#include<iostream>
class Thread:public QThread{
    void run(){
        while(1){
            std::string isQ;
            std::cin>>isQ;
            if(isQ.compare("Q")==0)break;
        }
    }
};
~~~

#### 使用Connect关联

~~~c++
Thread mythread;
mythread.start();
QObject::connect(&mythread,&Thread::finished,&a,QCoreApplication::quit);
~~~

主函数完整代码

~~~c++
#include "mainwindow.h"
#include<iostream>
#include <QApplication>
#include"myclass.h"
#include "qmetaobject.h"
#include<QThread>
#include<iostream>

class Thread:public QThread{
    void run(){
        while(1){
            std::string isQ;
            std::cin>>isQ;
            if(isQ.compare("Q")==0)break;
        }
    }
};

int main(int argc, char *argv[])
{
    QApplication a(argc, argv);
    QSharedPointer<MyClass>myinstance=QSharedPointer<MyClass>(new MyClass);
    QObject*object1=myinstance.get();

    myinstance->setPriority(MyClass::VeryLow);
    object1->setProperty("priority","VeryHigh");
    object1->setProperty("isGood",false);

    qInfo()<<myinstance->priority();
    qInfo()<<myinstance->property("isGood");
    qInfo()<<myinstance->metaObject()->classInfo(0).name();
    qInfo()<<myinstance->metaObject()->classInfo(0).value();
    qInfo()<<myinstance->metaObject()->classInfo(1).name();

    qInfo()<<myinstance->metaObject()->classInfo(1).value();
    
    Thread mythread;
    mythread.start();
    QObject::connect(&mythread,&Thread::finished,&a,QCoreApplication::quit);

    return a.exec();
}

~~~

当然，这里完全可以不用线程，可以利用代码块的机制使得智能指针离开作用域而自动释放，使其自动回收。

~~~C++
#include "mainwindow.h"
#include <QApplication>
#include"myclass.h"
#include "qmetaobject.h"

int main(int argc, char *argv[])
{
    QApplication a(argc, argv);
  {
    QSharedPointer<MyClass>myinstance=QSharedPointer<MyClass>(new MyClass);
    QObject*object1=myinstance.get();

    myinstance->setPriority(MyClass::VeryLow);
    object1->setProperty("priority","VeryHigh");
    object1->setProperty("isGood",false);

    qInfo()<<myinstance->priority();
    qInfo()<<myinstance->property("isGood");
    qInfo()<<myinstance->metaObject()->classInfo(0).name();
    qInfo()<<myinstance->metaObject()->classInfo(0).value();
    qInfo()<<myinstance->metaObject()->classInfo(1).name();

    qInfo()<<myinstance->metaObject()->classInfo(1).value();
   }

    return a.exec();
}

~~~

这里其实不需要共享，也可以换成QScopedPointer来替换，和之前的类似，这个指针其实和C++11引入的std::unique_ptr是一样的

这是我对于unique_ptr的理解部分:[Cpp11-14-/智能指针unique_ptr.cpp at main · caomengxuan666/Cpp11-14- (github.com)](https://github.com/caomengxuan666/Cpp11-14-/blob/main/智能指针unique_ptr.cpp)

这是我对shared_ptr的理解部分:[Cpp11-14-/智能指针shared_ptr.cpp at main · caomengxuan666/Cpp11-14- (github.com)](https://github.com/caomengxuan666/Cpp11-14-/blob/main/智能指针shared_ptr.cpp)

## P8_3.1QString

### QString遥遥领先之处

#### QString的特性

* QString存储16位QChar(Unicode)字符串

* QString使用隐式共享(copy-on-write)来提高性能

  举个例子，有一个str1,有一个str2，都是"abc"。那么如果都给str1,str2赋值为"abc"就很影响性能了。如果str1和str2都要使用，那么它们会被指向同一个"abc"。只有当其中一个字符串的值要发生改变的时候，才会拷贝一个"abc"下来并且对其进行修改。这就是copy-on-write，它并不是由QT提出的，而是一种提升性能的常见的方案。

#### 从QString的优化来看C++中拷贝的优化问题

C++中各种各样的地方都会隐式触发调用拷贝构造和赋值函数发生不必要的拷贝操作，而不必要的拷贝对于性能是一种很大的浪费。就比如值传递在大部分情况下就是不如地址传递快，因为传值也会发生拷贝，产生副本，造成资源浪费。

比如

~~~C++
for(auto e:nums)
~~~

这里用一个range-for遍历nums,这种写法很糟糕，因为每次遍历实质上都需要进行以下步骤

~~~c++
type e=nums[i];
~~~

最正确合理的写法应该是

~~~c++
for(auto&e:nums)
~~~

通过引用的方式大大减少拷贝带来的成本



  日常开发中又不能什么都写个explicit或者直接把缺省的拷贝构造delete掉，因为这本身就是编译器的一种优化，什么优化都禁用程序的性能只会更差，所以平时要注意可能会发生隐式拷贝构造的地方，这样才能提高性能。

  总结一下，以下还有一些可能在初始化的时候就发生拷贝构造的地方

   ***1.将一个对象作为实参给一个非引用参数类型的形参***

   ***2.从一个返回类型为非引用类型的函数返回一个形参***

   ***3.用花括号列表初始化一个数组中的元素活一个聚合类中的成员***

  ***4.代码中由隐式转换产生的临时变量***

  ***5.STL中的非临时变量引发的拷贝构造，比如vector的扩容，vector的赋值，std::sort，不过一般这种情况用std::move就可以很好的解决，因为只改变了指针指向***

  ***6.交换两个对象，会调用拷贝构造函数和赋值函数，这种情况可以在类内重载swap来改进，因为拷贝构造和赋值函数会引入new,memcpy,strlen,delete等等操作***

> Unicode是一种国际标准，支持当今使用的大多数书写系统。它是US-ASCII和Latin-1的超集(与子集相同字符编码相同)

在Qt开发中，一般不再使用STL中的std::string了，因为相对来说QString更加强大

std::string中一个元素就是char,一个char就是八位，所以能存放的状态很有限，放中文的话就没有这么多状态来表示了，只有2的八次方个状态，但是常用中文就有好几千字了，只有200种状态肯定是不够用的。QString才有的QString就可以实现国际化了，可以用来表示大部分语言了。

![image-20231106160058035](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231106160058035.png)

![image-20231106155737371](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231106155737371.png)

在Qt中QString提供的函数实在太多太多，std::string有的QString有，std::string没有的,QString也有。对比之下确实可以说是遥遥领先了。(我参考了cppreference上面的std::string的文档以及QT CREATOR中自带的文档得出的结论)

### QString的使用

#### 1.可以头插尾插、大小写转换

~~~C++
#include <QCoreApplication>

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);
    QString str{"look down at "};
    str.append("lng");
    str.prepend("i ");
    qDebug()<<str;
    qDebug()<<"The str has"<<str.count()<<"characters";//count返回一个字符数，其实等效于length
    //toUpper和toLower并不会直接改变str,改变的只是副本
    qDebug()<<str.toUpper();
    qDebug()<<str.toLower();
    return a.exec();
}
~~~

![image-20231106160805821](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231106160805821.png)

#### 2.支持直接赋值，拷贝构造，大括号聚合初始化，支持由std::string和char*类型转化后进行赋值和拷贝赋值

~~~c++
#include <QCoreApplication>

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);
    QString str1="Korean Tarzan";
    QString str2("Spy zika");
    QString str3={"Patriotic scout"};
    std::string s1=("Stupid hang");
    QString str4=s1.c_str();//如果要直接利用一个std::string进行对其进行赋值，必须先转化成c风格字符串
    std::string s2="Onlyman gala";
    QString str5=QString::fromLatin1(s2.data(),s2.size());
    char s3[]="A LCK coach";
    QString str6(s3);
    qDebug()<<str1<<"\n"<<str2<<"\n"<<str3<<"\n"<<str4<<"\n"<<str5<<"\nA"<<str6;
    return a.exec();
}

~~~



![image-20231106162019152](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231106162019152.png)

#### 3.支持对于各种符号的重载，如中括号

这是我在QT文档中查询到的QString支持的重载符号大全

![image-20231106162551766](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231106162551766.png)

~~~C++
#include <QCoreApplication>

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);
    QString str{"cmx666"};
    for(int i=0;i<str.length();i++){
        qDebug()<<str[i];
    }
    return a.exec();
}

~~~

![image-20231106162918538](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231106162918538.png)

其实从古老C语言来看，以前是没有中括号这种东西的，比方说有一个叫str的数组，要访问它就直接是利用*(str+index)的方式来访问的，是为了方便，才在后面的标准里面提出了数组[]这种语法糖，这也是数组从0开始计数的原因，因为str地址上存放的本身就是第一个元素。std::string延续了这种用中括号访问数组元素的风格，QString也将其完美继承了下来。而关于at函数，其实就和中括号的实现是一样的,str[0]就是str.at(0)，写法的区别而已

这是我对这部分的个人理解：4.82 m@Q.kP HVl:/ 08/24 复制打开抖音，看看【暮光之眼的作品】经典语法糖之数组访问# 计算机 # 编程 # 教程... https://v.douyin.com/iRN6y5uS/

* 这里的operator有两个重载版本![image-20231106163626950](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231106163626950.png)

其中一个版本是可以修改的QChar&，一个是不可修改的const QChar

* at返回就比较限定了
  ![image-20231106163844570](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231106163844570.png)

只有一个const QChar版本，无法被修改

<u>**但是at版本是有一个好处的，因为它是只读，所以比[]更加高效**</u>



#### 4.可以使用实际值替换特定控制字符来灵活构建字符串

~~~C++
#include <QCoreApplication>

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);
    QString s1="这里有%1个半岛铁盒";
    int n=519;
    qDebug()<<s1.arg(n);
    QString s2="缓缓飘落的枫叶从%1米处落下";
    double h=5.56;
    qDebug()<<s2.arg(h);
    QString s3="手牵手%1步%2步%3步望着天";
    int q=1,w=2,e=3;
    qDebug()<<s3.arg(q).arg(w).arg(e);//有点链式编程的味道了
    return a.exec();
}

~~~

![image-20231106164658567](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231106164658567.png)

这里面的arg就是argument,是实参的意思

这里的%后面的数字就决定了执行的顺序

#### 5.可以灵活截取子字符串

~~~c++
#include <QCoreApplication>

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);
    QString str={"而我独缺你一生的了解"};
    qDebug()<<str.left(4);
    qDebug()<<str.mid(4,3);
    qDebug()<<str.right(5);
    return a.exec();
}

~~~

![image-20231106165323642](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231106165323642.png)

其实这里的成员函数就很像std::substr，但是substr需要一个起始位置，也就是说需要一个从左往右边的一个序号，并不能直接从尾部开始往左截取(不能通过倒数第几个来进行截取)。当然要这么实现也不是不行，得调用个rfind(）从右边往左边查找想找的子串，然后利用这个返回值——这个子串的第一个字符的索引，把这个索引作为substr的第一个参数，把子串的size作为第二个参数（要截取的字符串大小）传进去，也能实现上面的right()的功能，不过这就多了一个rfind()的开销，所以说QString遥遥领先。QString的成员函数是很丰富的，不需要再自己考虑这么多东西，像mid()这样获取左右边界进行截取也是非常的方便。![image-20231106195248125](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231106195248125.png)

#### 6.也有迭代器，支持range-for，支持std::string所支持的所有遍历操作

~~~C++
#include <QCoreApplication>
#include <QTextStream>
int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);
    QTextStream out(stdout);//使之成为标准输出流
    out.setEncoding(QStringConverter::System);//设置为系统里面的编码，主要是这样就可以支持中文了
    QString str{"我们意念合一！"};
    for(QChar &qc:str)out<<qc<<"";
    out<<Qt::endl;

    for(QChar*it=str.begin();it!=str.end();++it){
        out<<*it<<"";
    }
    out<<Qt::endl;

    for(int i=0;i<str.size();++i)out<<str.at(i)<<"";
    out<<Qt::endl;
    return a.exec();
}

~~~

![image-20231106200900107](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231106200900107.png)

这里，可以注意到QString也支持采用range-for和迭代器访问的方式，和std::string一样，也可以正常采用at的方式来遍历，也就是说std::string的遍历方式QString都是完美支持的，因为有迭代器的缘故所以<algorithm>中的for_each也是可以正常使用的

#### 7.支持字符串的比较

据我观测，QString的compare应该是和std::string::compare差不多的

![image-20231106202021878](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231106202021878.png)

![image-20231106201938033](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231106201938033.png)

因为C++14之后重载版本也变多了，C++14/17/20都给出了新的重载实现。所以在这个成员函数方面QString就不是特别遥遥领先了

~~~C++
#include <QCoreApplication>
#include<QTextStream>
int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);
    QString str1{"Cmx"};
    QString str2{"cmx"};
    QString str3{"cmx\n"};
    if(QString::compare(str1,str2)==0){
        qDebug()<<"str1与str2相等";
    }
    else{qDebug()<<"str1与str2不相等";}
    qDebug()<<"大小写不敏感的比较结果";
    //这个Qt::CaseInsensitive我个人感觉是一个谓词，从上面那个文档中也能看出来，缺省情况下就是大小写敏感的。但是如果说有验证码这种场景可能就设置成大小写不敏感好一点。
    if(QString::compare(str1,str2,Qt::CaseInsensitive)==0){qDebug()<<"str1与str2相等";}
    else qDebug()<<"str1与str2不相等";

    if(QString::compare(str2,str3)==0)qDebug()<<"str2与str3相等";
    else qDebug()<<"str2与str3不相等";
    str3.chop(1);//删掉最后一个字符，这个chop就是尾删
    qDebug()<<"删掉换行符之后:";
    if(QString::compare(str2,str3)==0)qDebug()<<"str2与str3相等";
    else qDebug()<<"str2与str3不想等";
    return a.exec();
}

~~~

![image-20231106203845138](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231106203845138.png)

这里面的compare是返回int的，如果是0就是相等，负数就是小于，正数就是大于，不过正常情况下很少有人在意大于和小于的情况，可以理解为0就是相等，不是0就不相等。但是，由于if语句简写是写成这个样子的

`if(!Qstring::compare(str1,str2))`

这样乍一看这一句，没有读过文档的人可能会产生误解：这是不是说如果str1和str2不相等的意思啊。所以这里要么完整写成函数返回值==0，要么，就写一个宏，使其等于0

比方说`const int STR_EQUAL=0;`这样子看起来就很醒目，一眼就能看出来是字符串相等的意思

可以看到上面还提供了很多种的扩展，比如说CaseSensitivity这种东西，就可以很好的在各个场景应用。比如说qq登录，大小写肯定要区分啊。但是网站的验证码登录，上面大小写切来切去不太方便，一般大写小写都算对了，所以说QString直接把功能都封装好了，不需要写个别的实现或者是继承一个std::string自己加扩展，QString遥遥领先。

#### 8.对于字符类型的分类

QString对于字符的分类仍然是比较传统的，分为字母，标点符号，数字和空格

~~~C++
#include <QCoreApplication>
#include<QTextStream>
int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);
    int digits,letters,spaces,puncts;
    digits=letters=spaces=puncts=0;
    QString str{"man,what can I say? Manba out.--42 kobe"};

    for(QChar s:str){
        if(s.isDigit())digits++;
        else if(s.isLetter())letters++;
        else if(s.isSpace())spaces++;
        else if(s.isPunct())puncts++;
    }
    qDebug()<<QString("There are %1 characters").arg(str.count());
    qDebug()<<QString("There are %1 letters").arg(letters);
    qDebug()<<QString("There are %1 digits").arg(digits);
    qDebug()<<QString("There are %1 spaces").arg(spaces);
    qDebug()<<QString("There are %1 punctuation characters").arg(spaces);
    return a.exec();
}

~~~



![image-20231106205734575](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231106205734575.png)

这几个检测字符类型的成员函数也是继承下来的，甚至是C的标准库里面都有，不作为成员函数,而作为一个标准函数出现的，如<ctype.h>

#### 9.很好地支持字符串的类型转换

~~~C++
#include <QCoreApplication>
#include<QTextStream>
int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);
    QString s1{"12"};
    QString s2{"28"};
    QString s3,s4;
    qDebug()<<s1.toInt()+s2.toInt();
    int n1=1;
    int n2=23;
    qDebug()<<s3.setNum(n1)+s4.setNum(n2);
    return a.exec();
}

~~~

![image-20231106210426989](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231106210426989.png)

我个人感觉做的和std::string很像，只是命名上稍微有点区别

![image-20231106210630973](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231106210630973.png)

![image-20231106210824733](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231106210824733.png)

很明显QString提供的转换函数要多得多

当然，把别的类型转换成QString，是需要利用QString提供的类型实现，可能这就稍微显得奇怪一点。因为大部分情况下，类型转换都是直接这么写

~~~C++
a.c_str();
a.tostring
~~~

然而这里需要这么写: `str.setNum(x);`

这里其实是有一个隐含的问题的，QString的构造函数没法直接用数字来构造，必须还先用to_string转换把int类型转换为string，然后再把这个string类型用c_str转换，因为string类型也是无法直接被用于构造的，这里的两步会产生极大的没有必要的开销。因为我是保持着RAII原则的，在C++里面是很需要遵循这个规则的，否则就很容易发生未定义事件。当然在这里，仅仅声明一个QString类型的对象而不做赋值是没有问题的，因为它采用了默认构造，实际上是赋值了，做了初始化的。所以说，如果想要用一个数字来赋值一个QString字符串，只要声明一个QString字符串并且紧接着调用这个setNum函数就好了。

#### 10.字符串的修改

* 某些方法，比如说toLower,它返回的是字符串修改之后的副本，不影响字符串本身
* 其他的方法**都**会修改原本的字符串，比如说remove,replace,clear等等操作

~~~C++
#include <QCoreApplication>
#include<QTextStream>
int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);
    QString str{"Jay"};
    str.append(" chou");
    qDebug()<<str<<Qt::endl;

    str.remove(3,5);
    qDebug()<<str<<Qt::endl;
    return a.exec();
}

~~~

![image-20231106212438655](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231106212438655.png)

#### 11.字符串的对齐

~~~C++
#include <QCoreApplication>
#include<QTextStream>
int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);
    QString field1{"Name:"};
    QString field2{"Occupation:"};
    QString field3{"Residence:"};
    QString field4{"Marital status:"};
    int width=field4.size();
    qDebug()<<field1.rightJustified(width,' ')<<"Cmx";
    qDebug()<<field2.rightJustified(width,' ')<<"Programmer";
    qDebug()<<field3.rightJustified(width,' ')<<"Huang Shan";
    qDebug()<<field4.rightJustified(width,' ')<<"Single";

    return a.exec();
}

~~~

![image-20231106213315546](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231106213315546.png)

可以利用leftJustfied和rightJustfied来对齐字符串

#### 12.格式转换

~~~C++
#include <QCoreApplication>
#include<QTextStream>
int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);
    QString allText="<hello world>";
    qDebug()<<allText.toHtmlEscaped();
    return a.exec();
}

~~~

![image-20231106213738860](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231106213738860.png)

这个toHtmlEscaped()就将纯文本字符串转化为具有HTML元字符的HTML字符串

## P9_4.1QTL容器

### 我对于STL和QTL的看法

* Qt中有以下常用的容器:QList,QSet,QMap
* 有两种容器：顺序容器和关联容器
* 顺序(sequence)容器一个接一个的存储项
  * QList,QVector（在QT6中,QVector是QList的别名)
* 而关联容器存储键值对
  * QSET,QMap
* 我个人理解的这些容器还是基于STL中的那些容器的升级适配版，除了多几个成员函数之外大差不差。对比STL中的容器，Qt中的容器是尽可能隐藏了底层细节，在某些实现方式方面和STL标准库有略微的不同。可能比较适合初学者，QT的容器也称之为QTL。我所知的QTL和STL最大的区别就是，QTL采用了写时复制技术，但是缺点就是不支持用户自定义的allocator

这是知乎上面的对比介绍

> - **QLinkedList —— std::list** 两者都是双向链表，两者可以直接互转。
> - **QVector —— std::vector** 两者都是动态数组，都是根据sizeof(T)进行连续分配，保证成员内存连续，能够用data()直接取出指针作为c数组使用，两者可以直接互转。
> - **QMap —— std::map** 两者都是红黑树算法，但不能互转，因为数据成员实现方式不同。std::map的数据成员用的是std::pair，而QMap用的是自己封装的Node，当然还是键值对.
> - **QMultiMap —— std::multimap** 同上。
> - **QList —— 暂无**。QList其实不是链表，是优化过的vector，官方的形容是array list。它的存储方式是分配连续的node，每个node的数据成员不大于一个指针大小，所以对于int、char等基础类型，它是直接存储，对于Class、Struct等类型，它是存储对象指针。
>
> 　　std::deque很相似，但有少许区别。据有的知友提出，QList更像是boost::deque。
>
> - **QBitArray —— std::bitset** 功能相同，实现相似，都是构造一个array，用位操作来存取数据。不同的是，QBitArray数据的基础元素是unsigned char，而bitset是unsigned long。所以QBitArray可能在空间消耗上会省一点。至于效率上么，二者查询都是一次寻址提取加一次移位操作，算法层面应该没有区别。
>
> 　　不过二者最大的差别是，std::bitset是定长，数据元素分配在栈上。QBitArray是变长，数据元素分配在堆上。这个肯定有性能差别。
>
> - **QHash —— std::unordered_map**都是各自实现了自己的hashTable，然后查询上都是用node->next的方式逐一对比，不支持互转，性能上更多的应该是看hash算法。QHash为常用的qt数据类型都提供好了qHash()函数，用户自定类型也能通过自己实现qHash()来存入QHash容器。
>
> - **QSet —— std::unordered_set**二者不能互转，实现方式本质相同，都是改造过的QHash，用key来存数据，value置空。另外STL提供了使用红黑树的std::set，可以看作是std::map的改造版。std::unordered_set效率上一般应该是和QSet没区别，std::set效率较低，但不用担心撞车。
>
> - **QVarLengthArray —— std::array** std::array是用class封装后的定长数组，数据分配在栈上。QVarLengthArray类似，默认也是定长数组，栈分配。但用户依旧可以添加超出大小的内容，此时它会退化为Vector，改用堆分配。
>
> - 性能上的对比：
>
> - QTL和STL性能，同样算法的容器基本在同一数量级。
>
>   Bitset容器可以看出，堆分配比栈分配有性能损失。
>
>   Set和Hash两者存在实现差异，所以benchmark结果差距较大。红黑树和hashtable的效率差距太大了……（后来得知STL有使用hashtable的std::unordered_set，不过没继续测了……）。
>
>   vector的insert效率太低，不管是Qt还是STL，时间开销都在内存移动上，而且不太可能通过除了内存池之外的办法进行优化，所以就没测了。
>
>   QList在拥有vector的操作性能的同时，通过前向扩展空间，模拟出了LinkedList的双向添加O(1)的特点。但存储int这种基础类型时，由于存在Node的构造开销，效率不如vector。
>
>   MSVC的Debug真凶残，不造加了多少overhead进去，这运行时间长的……

在容器这方面只能说是各有千秋，还是得看具体的应用场景选择合适的容器来进行开发。

我查询文档，发现主要区别如下

#### API上的差别

- QTL有Qt风格和STL风格，两种风格的API，比如 `append()` 和`push_back()`，`count()` 和 `size()`，`isEmpty()` 和`empty()`。
- QTL不支持allocator；STL支持指定allocator以及自定义allocator。
- Qt除了我们常用的类似于STL的迭代器，也提供了JAVA风格的迭代器。关联容器，Qt STL风格迭代器通过`iterator.key()`和`iterator.value()`获取键值，STL通过`std::pair<key,value>`结构获取键值。

#### 实现原理上面的差别

- QTL采用隐式共享，赋值操作执行的都是浅拷贝。STL，所有STL容器赋值操作执行的都是深拷贝；
- QTL线程安全，STL线程不安全；
- `QQueue`用`QList`作为底层容器，`QStack`用`QVector`作为底层容器；`std::queue`和`std::stack`都默认采用 `std::deque`作为底层容器，可以手动指定容器。

#### QTL自身的特性

- 除了`QVarLengthArray`，所有QTL数据都存放在堆空间，支持隐式共享。
- `QVarLengthArray`：数据存储在对象内，连续存储结构，无隐式共享功能；
- `QVector`：在堆空间存储数据，连续存储结构；
- `QLinkedList`：双向链表，在堆空间存储数据，链式存储结构；
- `QList`：后面详细说明；
- `QMap`：用红黑树管理键值对数据，key不可重复；
- `QMultiMap`：用红黑树管理键值对数据，key可重复；
- `QHash`：用哈希表管理键值对数据，key不可重复；
- `QMultiHash`：用哈希表管理键值对数据，key可重复；
- `QSet`：用哈希表存储值类型的数据，值可重复；
- `QQueue`：队列结构，先进先出；
- `QStack`：栈结构，先进后出；
  由于QTL和QList相似度太高，我不会做过于复杂的解读，以下是我的个人理解：
- [caomengxuan666/STL_AND_ALGORITHM_CPP98: 对C++98版本中的STL和算法库中常用的自带算法做出了详细的解释教学和代码实现 (github.com)](https://github.com/caomengxuan666/STL_AND_ALGORITHM_CPP98)


##### 特殊的QList

`QList`名称中有list，但不以链表结构存储数据，不等同于`std::list`；`QList`两端插入数据较快，但它不完全以连续内存结构存储数据，不等同于`std::deque`。`QList`是优化后的`QVector`，以两种方式存储数据。

- 当类型`T`同时满足两个条件：1.`sizeof（T）<= sizeof（void*）`；2.`T`是`Q_PRIMITIVE_TYPE`或者`Q_MOVABLE_TYPE`类型。`T`会被存储在一段连续的内存中。

![image-20231107185330288](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231107185330288.png)

这是对这`Q_PRIMITIVE_TYPE`和`Q_MOVABLE_TYPE`类型的官方说明文档：[ - Global Qt Declarations | Qt Core 5.15.15](https://doc.qt.io/qt-5/qtglobal.html#Q_DECLARE_TYPEINFO)

* `T`不满足上述两个条件的时候，`QList`会把指向`T`的指针存放在一段连续的内存空间，而这些指针指向的数据`T`则被存放在堆中。

![image-20231107185622197](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231107185622197.png)

优势：进行大对象插入的时候，`QList`比`QVector`，`std::deque`更快，因为它只需要移动指针。

劣势：`QList`更加浪费内存空间。

由于QList性能比QVector各方面看起来都好，这就导致QVector这个容器的地位很尴尬，所以Qt官方就在QT6版本的时候，直接暴力地让QVector成为了QList的一个别名，以前的QVector容器的实现就直接删除作废了。。。

#### QList的成员函数的使用

~~~C++
#include <QCoreApplication>
#include <QList>
int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);
    QList<int>val={1,2,3,4,5};
    qDebug()<<"容器大小"<<val.size();
        qDebug()<<"第一个元素是"<<val.first();
        qDebug()<<"最后一个元素是"<<val.last();
        val.append(6);
        val.prepend(0);
        qDebug()<<"Elements:";
        for(auto &v:val){
            qDebug()<<v;
        }
        return a.exec();
}

~~~



![image-20231107190522567](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231107190522567.png)



![image-20231107190858846](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231107190858846.png)

可以从命名风格上看出来，QT不仅有自己的一套风格，比如说first,last,也保留了std中的命名风格，front和back仍然是有的，而且实现的功能也是一样的。在我的理解之下，QT就是C++的一个超集，就好像C++对C的包含关系一样一样。在代码构建工具上也是这样，不过QT6之后QT官方放弃了对qmake的更新，确实也是因为CMake现在做的很好，上限也很高了，不知道未来是否会对QTL还有QT的各种各样的函数做出合并操作。

~~~c++
#include <QCoreApplication>
#include <QList>
#include <QLocale>
#include <QCollator>
#include <algorithm>
int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);
    QList<QString>name={"七里香","龙卷风","珊瑚海"};
    for(auto&i:name)qDebug()<<i;
    name<<"夜曲"<<"枫";
        qDebug()<<"***************";
    QLocale cn(QLocale::Chinese);
        QCollator collator(cn);
        std::sort(name.begin(),name.end(),collator);
        qDebug()<<"按照拼音排序";
        for(QString&n:name){
            qDebug()<<n;
        }
        return a.exec();
}

~~~

![image-20231109204219963](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231109204219963.png)

其实这里面的QCollator就是qt提供的一个仿函数了，是专门用于对于不同的语言实现特定排序的，比如说我填入的是中文，那么，他就会按照中文的方式对容器进行排序，这个仿函数就是填入sort的谓词位置的，这样就可以按照中文拼音的方式排序了，std标准库里面提供的仿函数基本上都是降序之类的基本的仿函数，大部分要靠用户来自己写，没有qt里面提供的多种多样。

~~~C++
#include <QCoreApplication>
#include <QList>
#include <QLocale>
#include <QCollator>
#include <algorithm>
int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);
    QString string="放在糖果旁的是我，很想回忆的甜,然而过滤了你和我，沦落而成美";
    QStringList items=string.split(",");
    QStringListIterator it(items);
    while(it.hasNext())qDebug()<<it.next().trimmed();
        return a.exec();
}

~~~

![image-20231109205317362](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231109205317362.png)

我用split来切割string，通过判断是否有","将其分割。所以，中文的“，”不会被分割，而英文的“,”则会被分割。

这里可以看出，QStringListIterator为QString提供了java风格的const迭代器，迭代器指向的都是中间的位置

![image-20231109205844355](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231109205844355.png)

这里的hasNext的逻辑也很简单，就是判断链表是否为空，相当于下图

![image-20231109205704805](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231109205704805.png)

![image-20231109205712265](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231109205712265.png)

#### QSet容器

##### 特性

* 提供具有快速查找功能的单值集
* 不支持排序，values方法返回一个QList,其中包含QSet中的元素
* unite方法执行两个集合的并集
* 实际上就是对应std::unordeded set,自身是乱序的，不支持排序，而且不能插入重复元素

如果实在想要排序，就直接把QSet的值全部放到QList中去，有多种构造函数，比如用QSet的迭代器或者是调用QSet的values方法，或者干脆使用QMap

#### QMap容器

特性

* 提供了QMapIterator这一QMap的JAVA风格迭代器,与QSet类似
* next():返回下一项并且将迭代器前进一个位置

value的迭代器:cbegin(),cend()

key的迭代器:KeyBegin(),KeyEnd()

#### 容器的排序

和STL一样的规则，只需要提供一个全局的bool函数或者一个仿函数或者一个lambda函数来填上std::sort的谓词的位置，都是可以的。

## P10_5.1日期和时间

### Qt中提供的日期和时间类

QT6有QDate，QTime,QDateTime类来处理日期和时间

* QDate	用于处理日期的类
* QTime    用于时钟时间的类
* QDateTime    将QDate和QTime对象组合到一个对象中的类

我用的比较熟悉的是C++11提供的chorono库，当然在qt中还是qt提供的时间库更加方便一些

这是我对chorono库的学习部分

[Cpp11-14-/时间操作chrono库.cpp at main · caomengxuan666/Cpp11-14- (github.com)](https://github.com/caomengxuan666/Cpp11-14-/blob/main/时间操作chrono库.cpp)

~~~C++
#include <QCoreApplication>
#include <QDate>
#include <QTime>
int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);
    QDate dt1{2023,11,10};
    qDebug()<<"The date is"<<dt1.toString();
    QDate dt2;
    dt2.setDate(2023,11,11);
    qDebug()<<"The date is"<<dt2.toString();

    QTime tm1{17,30,12,55};
    qDebug()<<"The time is"<<tm1.toString("hh:mm:ss.zzz");

    QTime tm2;
    tm2.setHMS(13,52,45,155);
    qDebug()<<"The time is"<<tm2.toString("hh:mm:ss A");
    return a.exec();
}

~~~

![image-20231110161426933](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231110161426933.png)

这里的两个函数原型是

`bool QDate::setDate(int year,int month,int day);`

`bool QDate::setHMS(int h,int m,int s,int ms=0);`

这是toString中参数的含义对照表

![image-20231110162111081](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231110162111081.png)

#### 显示当前日期和时间

~~~C++
#include <QCoreApplication>
#include <QDate>
#include <QTime>
int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);
    QDate cd=QDate::currentDate();
    QTime ct=QTime::currentTime();
    qDebug()<<"Current date is:"<<cd.toString();
    qDebug()<<"Current time is:"<<ct.toString();
    return a.exec();
}

~~~



![image-20231110162558130](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231110162558130.png)

不得不说获取当前时间确实比chorono库要方便很多

这是我用chorono库实现的对应的代码

~~~C++
void test02() {
    using namespace chrono;
    //1.静态成员函数chrono::system_clock::now()用于获取系统时间（C++时间）
    auto now = system_clock::now();

    //2.静态成员函数chrono::system_clock::to_time_t()把系统时间转换成time_t(UTC时间)
    auto t_now = system_clock::to_time_t(now);

    //可以使用时间偏移
    //t_now=t_now+24*60*60  //把当前时间加1天

    //3.std::localtime()函数把time_t转换成本地时间(北京时间)
    //localtime()不是线程安全的，VS用localtime_s代替，Linux用localtime_r代替
    auto tm_now = std::localtime(&t_now);

    //4.格式化输出本地时间
    cout << put_time(tm_now, "%Y-%m-%d %H:%M:%S") << endl;
    cout << put_time(tm_now, "%Y-%m-%d") << endl;
    cout << put_time(tm_now, "%H:%M:%S") << endl;
    cout << put_time(tm_now, "%Y%m%d%H%M%S") << endl;

    stringstream ss;    //创建stringstream对象ss
    ss << put_time(tm_now, "%Y-%m-%d %H:%M:%S");   //把时间输出到对象ss中
    string timestr = ss.str();
    cout << timestr << endl;
}
~~~

可以发现，chrono库中获取的系统时间是C++时间，是需要额外调用函数把系统时间转化成UTC时间，然后如果需要获取本地时间，还需要一个额外的函数把这个UTC时间转化为北京时间，然后输出的时候，还要把本地时间手动格式化才行，对比起来Qt的时间库至少在易用性上面遥遥领先。

#### 比较日期

和chrono库类似，chrono的模板类duration重载了大量的运算符，而Qt的时间库也实现了这些运算符的重载，比如说QTime和QDateTime对象可以比较日期

~~~C++
#include <QCoreApplication>
#include <QDate>
#include <QTime>
int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);
    QDate dt1={2023,1,23};
    QDate dt2={2019,10,24};
    if(dt1<dt2)qDebug()<<dt1.toString()<<"comes before"<<dt2.toString();
    else qDebug()<<dt1.toString()<<"comes after"<<dt2.toString();
    return a.exec();
}

~~~

![image-20231110163449250](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231110163449250.png)

#### 判断闰年

Qt考虑的太过周到了，连判断是否是闰年都独立封装成了一个函数

~~~C++
#include <QCoreApplication>
#include <QDate>
#include <QTime>
int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);
    QList<int>years{2000,2004,2007,2013,2016,2018,2019,2022,2023};
    for(int&year:years)
        if(QDate::isLeapYear(year))
        qDebug()<<year<<"is a leap year";
    else qDebug()<<year<<"is not a leap year";
    return a.exec();
}

~~~

![image-20231110163843882](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231110163843882.png)

### 日期格式

#### 预定义的日期格式

~~~C++
#include <QCoreApplication>
#include <QDate>
#include <QTime>
int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);
    QDate cd=QDate::currentDate();
    qDebug()<<"Today is"<<cd.toString();
    qDebug()<<"Today is"<<cd.toString(Qt::TextDate);
    qDebug()<<"Today is"<<cd.toString(Qt::ISODate);
    qDebug()<<"Today is"<<cd.toString(Qt::RFC2822Date);
    return a.exec();
}

~~~

![image-20231111121607460](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231111121607460.png)

tostring预定义的日期格式缺省就是TextDate。

预定义的写上就会按照这种标准进行打印

#### 自定义的日期格式

~~~C++
#include <QCoreApplication>
#include <QDate>
#include <QTime>
int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);
    QDate cd=QDate::currentDate();
    qDebug()<<"Today is"<<cd.toString("yyyy-MM-dd");
    qDebug()<<"Today is"<<cd.toString("yy/M/dd");
    qDebug()<<"Today is"<<cd.toString("d.M.yyyy");
    qDebug()<<"Today is"<<cd.toString("d-MMMM-yyyy");
    return a.exec();
}

~~~

![image-20231111121938800](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231111121938800.png)

具体参数含义见下表

![image-20231111122118853](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231111122118853.png)

### 处理周工作日

~~~C++
#include <QCoreApplication>
#include <QDate>
#include <QTime>
int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);
    QDate cd=QDate::currentDate();
    int wd=cd.dayOfWeek();

    QLocale locale(QLocale("zh_CN"));
    qDebug()<<"今天是"<<locale.dayName(wd);
    qDebug()<<"今天是"<<locale.dayName(wd,QLocale::ShortFormat);

    QLocale usa(QLocale("en_US"));
    qDebug()<<"Today is"<<usa.dayName(wd);
    qDebug()<<"Today is"<<usa.dayName(wd,QLocale::ShortFormat);

    return a.exec();
}

~~~

![image-20231111122839061](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231111122839061.png)

* dayOfWeek返回一个1-7的数字
* 1表示周一，7表示周日

用了dayOfWeek成员函数可以让打印的dayName变成周几这种形式

### 计算天数

* 可以使用daysInMonth计算特定月份的天数

* 使用daysInYear方法计算一年中的天数

我个人感觉这样比较鸡肋，因为一年的天数是不是365只看是不是闰年，而且除了二月，所有月份的日期都是固定的，这里就不详细展开了。

### 检查日期有效性

~~~C++
#include <QCoreApplication>
#include <QDate>
#include <QTime>
int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);
    QList<QDate>dates{QDate(2020,1,23),QDate(2021,12,12),QDate(2023,2,29)};
    for(int i=0;i<dates.size();i++){
        if(dates.at(i).isValid())qDebug()<<"Date"<<i+i<<"is a valid date";
        else qDebug()<<"Date"<<i+1<<"is not a valid date";
    }
    return a.exec();
}

~~~

![image-20231111125056685](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231111125056685.png)

### 计算日期

* addDays:从一个特定日期算起n天后的日期
* daysTo:返回到所选日期的天数

~~~C++
#include <QCoreApplication>
#include <QDate>
#include <QTime>
int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);
    QDate dt=QDate::currentDate();
    QDate nd=dt.addDays(55);
    QDate bir={2024,1,23};
    qDebug()<<"55 days from"<<dt.toString()<<"is"<<nd.toString();
    qDebug()<<"There are"<<QDate::currentDate().daysTo(bir)<<"days till my birthday";
    return a.exec();
}

~~~

![image-20231111125545264](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231111125545264.png)

### QDateTime类

QDateTime对象包含日历日期和时钟时间，它是QDate和QTime的组合

~~~C++
#include <QCoreApplication>
#include <QDateTime>
int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);
    QDateTime cdt=QDateTime::currentDateTime();
    qDebug()<<"The current datetime is"<<cdt.toString();
    qDebug()<<"The current date is"<<cdt.date().toString();
    qDebug()<<"The current time is"<<cdt.time().toString();
    return a.exec();
}

~~~



![image-20231111125917752](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231111125917752.png)

### UNIX新纪元

计算机的时间都以这个UNIX新纪元的时间开始计算

* time(0)返回UNIX新纪元以来的秒数
* 参数只填0的话显示的UNIX新纪元的时间

~~~C++
#include <QCoreApplication>
#include <QDateTime>
#include <ctime>
int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);
    QDateTime dt;
    dt.setSecsSinceEpoch(0);
    qDebug()<<dt.toString();

    dt.setSecsSinceEpoch(time(0));
    qDebug()<<dt.toString();

    QDateTime cd=QDateTime::currentDateTime();
    qDebug()<<cd.toString();

    qDebug()<<time(0);
    qDebug()<<cd.toSecsSinceEpoch();
    return a.exec();
}

~~~

![image-20231111130314918](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231111130314918.png)

从新纪元到现在的时间和直接获取现在的时间是一样的

## P11_6.1QFile

* Qt处理文件与目录的基本类
  * QFile:用于读取和写入文件的接口
  * QDir:对目录结构及其内容的访问
  * QFileInfo:独立于系统的文件信息，包括文件的名称和在文件系统中的位置、访问时间和修改时间、权限或文件所有权

![image-20231111142957917](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231111142957917.png)

QFile是一种输入/输出设备，用于读取和写入文本、二进制文件和资源。QFile可以单独使用，也可以死与QTextStream或QDataStream一起使用

### 获取文件名称

这里只需要改变项目的Command line arguments就行了，比如我在工作目录下写了一个test.txt

~~~C++
#include <QCoreApplication>
#include <QFile>
int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);
    if(argc!=2){
        qWarning("请给出文件名称");
        return 1;
    }
    QString filename=argv[1];
    QFile f(filename);
    if(!f.exists()){
        qWarning("指定文件不存在");
        return 1;
    }
    qDebug()<<f.fileName();
    return a.exec();
}

~~~

![image-20231111143641242](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231111143641242.png)

### 读取文件

QTextStream可以在QIODevice、QByteArray或QString上操作。可以方便地读写单词、行和数字。支持格式设置

为了读取文件的内容

* 打开文件进行读取
* 创建输入文件流
* 从该流中读取数据

~~~C++
#include <QCoreApplication>
#include <QFile>
int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);
    QFile f("test.txt");
    if(!f.open(QIODevice::ReadOnly)){
        qWarning("Cannot open file for reading");
        return 1;
    }
    QTextStream in(&f);
    while(!in.atEnd()){
        QString line=in.readLine();//不是末尾读一行出来并输出
        qDebug()<<line;
    }
    return a.exec();
}

~~~



![image-20231111153426679](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231111153426679.png)

### 写文件

为了写入文件

* 以写入模式打开文件
* 创建指向该文件的输出流
* 使用写入操作符写入该流

~~~C++
#include <QCoreApplication>
#include <QFile>
int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);
    QTextStream out{stdout};
    out.setEncoding(QStringConverter::System);
    QString filename="test.txt";
    QFile f(filename);

    if(f.open(QIODevice::WriteOnly)){
     
        QTextStream out(&f);
        out<<"Name:cmx"<<Qt::endl
            <<"Age:19"<<Qt::endl
            <<"Language:C PLUS PLUS"<<Qt::endl
            <<"IDE:CLION Visual_Studio VSCode Vim Qt_Creator"<<Qt::endl;
    }
    else qWarning("could not open this file");
    return a.exec();
}

~~~

![image-20231111155313175](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231111155313175.png)

### 拷贝文件

复制文件时，会使用不同的名称或不同的位置创建新副本

~~~C++
#include <QCoreApplication>
#include <QFile>
int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);
    QTextStream out{stdout};
    if(argc!=3){
        qWarning("Usage:copyfile source destination");
        return 1;
    }
    QString src=argv[1];
    if(!QFile{src}.exists()){
        qWarning("The source file does not exist");
        return 1;
    }
    QString dest(argv[2]);
    QFile::copy(src,dest);//把src拷贝给dest
    return a.exec();
}

~~~



![image-20231111161220963](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231111161220963.png)

## P12_6.2QDir

### 获取目录中文件的信息

~~~C++
#include <QCoreApplication>
#include <QFile>
#include <QDebug>
#include <QDir>
int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);
    QDir dir;
    dir.setFilter(QDir::Files|QDir::Hidden|QDir::NoSymLinks);
    dir.setSorting(QDir::Size|QDir::Reversed);
    //返回目录中所有文件和目录的QFileInfo对象列表
    QFileInfoList list=dir.entryInfoList();
    qDebug()<<" Bytes Filename";
    for(auto&fileinfo:list){
        qDebug()<<QString("%1,%2").arg(fileinfo.size(),10).arg(fileinfo.fileName());
        //正fieldWidth生成右对齐的文本
        //负fieldWidth生成左对齐文本
        //默认为0
    }
    return a.exec();
}

~~~

![image-20231111222000673](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231111222000673.png)

#### 我对于QDir成员函数中出现的"管道运算符"的深入探索

注意这里

![image-20231111222256946](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231111222256946.png)

什么？这不是C++20的管道运算符，这是一个位运算符，我们这个位运算符体积小方便携带，就是看起来有点像管道运算符。

没错，至今为止CSDN和知乎上还有很多固执的人居然说它是“或”，或可是“||”啊，而且“或”，不是多者满足一者即可吗？所以说这里面的“|"应该是什么呢。

我看的时候觉得也很懵逼，明明qt6还不支持C++20的功能特性呢，怎么会有管道运算符这种超越C++标准委员会官方设定的东西（虽然说1970年UNIX上面就有管道机制了，但是在具体语言层面QT怎么可能实现的比C++的ISO 标准还早）而且更要命的是，管道运算符也是用来实现过滤器的这样一个功能的：通过一个命令的输出过滤并且传递给另一个文件进行处理，而且这里的逻辑也是这样的，层层过滤的。

那这里究竟是怎么实现的呢？

我们发现一点，这里面的Files,Hidden,NoSymLinks都是绿色的，默认情况下绿色就是宏定义或者一个枚举值。所以我们直接转到这几个绿绿的东西的实现上面

![image-20231111223158031](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231111223158031.png)

哎，破案了，这里面的东西都是16进制的枚举值

再来看上面这个'|',虽然说他实现了和管道一样的过滤功能，真的是好巧不巧。这里其实它就是个按位或运算符，按位或运算符用于将两个整数的对应位进行逻辑或操作。具体地说，如果两个对应位中至少有一个为1，则结果位为1，否则为0。

就比如说，这里我们设置的三个条件dir.setFilter(QDir::Files|QDir::Hidden|QDir::NoSymLinks);

他们的十六进制分别是0x002,0x100,0x008

按照这种运算就是0x002|0x100|0x008

 也就是  000000010 | 100000000 |      1000 -------------   100000110

把他们转换成二进制进行运算后得到的结果应该是0x10A,那么在实现文件中，我们就可以找到一个对应0x10A的结果：包括文件、包括隐藏文件、不包括符号链接

我试图通过它的实现文件来找到具体的实现逻辑，但是我发现一个很大的问题："qdir.h"没有对应实现的"qdir.cpp"。我按了一下F4，切换出来的源文件居然是qflags.h，再切换了一下发现居然又跳到了qtypeinfo.h。好嘛，到最后才发现实现也给Qt写到那个qflags.h和qtypeinfo.h文件里面了（不好评价这种开发风格，要么就.h/.cpp严格分开，要么用.hpp一起实现，哪有人用.h把模板和具体实现一起写在里面的）。这层层的包含关系，加上大量嵌套的模板类，使得我也无法理解具体实现原理了。反正基本的逻辑就是：通过|这个位运算符获取了一个结果，作为参数，来达到和管道运算符一样的过滤器的作用就对了。

然后官方的文档的介绍也是潦草到只介绍了应用场景，压根没想让人知道底层是怎么实现的，所以说QT喜欢隐藏底层实现的细节，但是仔细看一下他们的底层实现就真的和屎山一样，所以他们才想藏起来。即使现在口碑还不错的Qt6还是不能避免它建立在前任版本的屎山之上。

![image-20231111225130062](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231111225130062.png)

我突然又想到了另一个东西：C++自己的文件系统库。哎，里面也是有一个“|”这样的东西的。

~~~
outfile2.open("D:\\afile.dat", ios::ate|ios::in);
~~~

所以，破案了，Qt采用的是和标准库中<fstream>一样的实现方式来编写的文件系统代码

我这里找到了标准库中给出的实现方式

（话说这个constexpr也是C++11引入的标准，说明这个标准库还是更新过的）

~~~C++
static constexpr _Openmode in = (_Openmode)0x01;
	static constexpr _Openmode out = (_Openmode)0x02;
	static constexpr _Openmode ate = (_Openmode)0x04;
	static constexpr _Openmode app = (_Openmode)0x08;
	static constexpr _Openmode trunc = (_Openmode)0x10;
~~~

~~~C++
	enum _Openmode
		{	// constants for file opening options
		_Openmask = 0xff};
~~~

~~~C++
	void open(const char *_Filename,
		ios_base::openmode _Mode = ios_base::out,
		int _Prot = (int)ios_base::_Openprot)
~~~

这里面的openmode不是别的，就是int，是给的int一个openmode的别名

![image-20231111231420699](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231111231420699.png)

所以说，本质上这个参数位置是一个十六进制数，通过位运算得到一个结果，然后函数再根据不同的数字产生不同的结果(具体实现逻辑不详)，我怀疑已经上升到了调用系统原生API的层面了，那么不同的编译器很可能给出不同的实现了，再往下面探寻就没有意义了，要知道MSVC/G++/CLANG这几种编译器的底层实现有很多很多地方是完全不一样的，比如说我感觉这里它就用了WINAPI的函数了，比如说Kernel32.dll和NTDLL.dll中的函数。。。

总的来说呢，这个setFilter函数就是用来设置访问文件的内容的，这里的setSorting是用来给搜索到的文件排序的。这里面是按照大小来排序的，这个Reversed就是表示按升序排列了。

### 创建目录

~~~C++
#include <QCoreApplication>
#include <QFile>
#include <QDebug>
#include <QDir>
int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);
    QDir dir;
    if(dir.mkdir("mydir"))qDebug()<<"mydir successfully created";
    dir.mkdir("mydir2");
    if(dir.exists("mydir2"))dir.rename("mydir2","newdir");
    dir.mkpath("temp/newdir");//创建一个temp的路径，里面再创建一个newdir 
    return a.exec();
}

~~~

![image-20231111233455371](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231111233455371.png)

![image-20231111233730960](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231111233730960.png)

![image-20231111233737992](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231111233737992.png)

#### 当前目录和其他特殊路径

QDir的许多静态函数提供了一些对常见目录的访问:

![image-20231111233846864](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231111233846864.png)

我个人感觉就有点像vscode里面的${workspaceFolder} ${fileDirname}这种东西，也有点像visual studio设置里面路径参数的宏定义，反正作用都差不多。

* setCurrent():设置应用程序的工作目录
* QCoreApplication::applicationDirPath():查找包含应用可执行文件的目录
* drives():提供根目录列表

## P13_6.3QFileInfo

### 读取、修改记录

文件存储有关上次（最后一次）读取或修改的信息，为了获得这些信息，需要使用QFileInfo类

~~~C++
#include <QCoreApplication>
#include <QFile>
#include <QFileInfo>
#include <QDateTime>
int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);
    if(argc!=2){
        qWarning("请给出文件名称");
        return 1;
    }
    QString filename=argv[1];
    QFileInfo fileinfo{filename};
    QDateTime last_rea=fileinfo.lastRead();
    QDateTime last_mod=fileinfo.lastModified();
    qDebug()<<"Last read:"<<last_rea.toString();
    qDebug()<<"Last modified:"<<last_mod.toString();
        return a.exec();
}

~~~



![image-20231111235407915](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231111235407915.png)

### 列出目录内容

~~~C++
#include <QCoreApplication>
#include <QFile>
#include <QFileInfo>
#include <QDir>
#include <QDirIterator>
int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);
    QDir dir{QDir::current()};
    dir.setFilter(QDir::Files|QDir::AllDirs);
    //setSorting无法更改QDirIterator中元素的顺序
    dir.setSorting(QDir::Size|QDir::Reversed);

    qDebug()<<QString("Filename").leftJustified(10).append("Bytes");
    QDirIterator it(dir);
    while(it.hasNext()){
        QFileInfo fileInfo=it.nextFileInfo();
        QString str=fileInfo.fileName().leftJustified(10);
        str.append(QString("%1").arg(fileInfo.size()));
        qDebug()<<str;
    }
        return a.exec();
}

~~~



![image-20231112000143920](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231112000143920.png)

如果加上QDir::NoDotAndDotDot的作用：去除“.” “..”，也就是表示当前目录和上一级目录的东西

这里可以重复设置setFilter,保留上一次结果的同时新的筛选条件也会加上，因为根据我们上面得出的结论，最终得到的参数都是十六进制数，所以你怎么排它，再加上它不影响最终结果的。

这里发现排序没有用，是因为这个JAVA风格的迭代器QDirIterator它不支持setSorting排序

## P14_7.1GUI初识

### Qt设计界面的三种方式:

* 纯代码

  * 优点:灵活
  * 缺点：太复杂了，不直观

* 使用Qt Designer的页面编辑器

  优缺点正好反过来

* 代码和Qt Designer相结合

  结合其优点，减少其缺点

####  手写代码

  ~~~C++
#include "widget.h"

#include <QApplication>

int main(int argc, char *argv[])
{
    QApplication a(argc, argv);
    Widget w;
    w.resize(250,250);
    w.setWindowTitle("快把鼠标放进来");
    w.setToolTip("vocal还真浪费你几秒钟啊");
    //.setWindowIcon(QIcon("这里可以加上路径作为图标来显示"));
    w.show();
    return a.exec();
}

  ~~~

  这里给他加上鼠标样式，三个不同的区域对应上不同的鼠标样式

  ~~~C++
#include "widget.h"
#include "./ui_widget.h"
#include <QGridLayout>
#include <QFrame>
Widget::Widget(QWidget *parent)
    : QWidget(parent)
{
    //这里的this都代表Widget类创建的对象，也就是这个窗口，也就是parent
    auto*frame1=new QFrame(this);
    frame1->setFrameStyle(QFrame::Box);
    frame1->setCursor(Qt::SizeAllCursor);
    auto*frame2=new QFrame(this);
    frame2->setFrameStyle(QFrame::Box);
    frame2->setCursor(Qt::WaitCursor);
    auto*frame3=new QFrame(this);
    frame3->setFrameStyle(QFrame::Box);
    frame3->setCursor(Qt::PointingHandCursor);
    auto*grid=new QGridLayout(this);//grid就是创建格子，排版,如果这里不加this，会导致窗口都堆在一块去
    grid->addWidget(frame1,0,0);
    grid->addWidget(frame2,0,1);
    grid->addWidget(frame3,0,2);
    setLayout(grid);
}

Widget::~Widget()
{

}


  ~~~

  通过dumpObjectTree可以看到结构如下

  ![image-20231122200419707](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231122200419707.png)

由于Qt的存在一个内存回收机制：父对象被销毁，子对象也会一起被销毁，所以说这里就不用关心内存回收问题了。写上this指针是一个比较好的习惯，因为不写又不加上setLayout，那就不会出现这种树状结构的，会导致内存泄漏。

这里再给这个代码加上一个按钮，为了加上这个按钮，可以采用一个全新的布局grid2,我这里把它设置在了frame2区域，然后给这个布局加上了一个退出按钮，然后利用信号槽，把点击信号传给了控制退出的槽。

~~~c++
#include "widget.h"
#include "./ui_widget.h"
#include <QGridLayout>
#include <QFrame>
#include<QApplication>
#include <QPushButton>
Widget::Widget(QWidget *parent)
    : QWidget(parent)
{
    //这里的this都代表Widget类创建的对象，也就是这个窗口，也就是parent
    auto*frame1=new QFrame(this);
    frame1->setFrameStyle(QFrame::Box);
    frame1->setCursor(Qt::SizeAllCursor);
    auto*frame2=new QFrame(this);
    frame2->setFrameStyle(QFrame::Box);
    frame2->setCursor(Qt::WaitCursor);
    auto*frame3=new QFrame(this);
    frame3->setFrameStyle(QFrame::Box);
    frame3->setCursor(Qt::PointingHandCursor);
    auto*grid=new QGridLayout(this);//grid就是创建格子，排版,如果这里不加this，会导致窗口都堆在一块去
    grid->addWidget(frame1,0,0);
    grid->addWidget(frame2,0,1);
    grid->addWidget(frame3,0,2);
    setLayout(grid);
    auto grid2=new QGridLayout(frame2);
    QPushButton*butt=new QPushButton("quit",this);
    grid2->addWidget(butt);
    connect(butt,&QPushButton::clicked,qApp,&QApplication::quit);
}

Widget::~Widget()
{

}


~~~

![image-20231122201102364](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231122201102364.png)

## P15_7.2使用样式表自定义界面

Qt样式表(style sheet)与HTML的CSS类似，是纯文本的格式定义，Qt支持css2定义的所有选择器，然后Qt自己也有一个qss,有更多的样式表，并且是以引入资源文件的方式加载使用。基本上可以认为qss就是抄袭(bushi)，又或者说是借鉴css的产物，盒子模型qss也有。反正qss/css都能用。

可以打开一个css文件进行操作如下

~~~C++
QFile stylefile("myStyle.css");
stylefile.open(QIODevice::ReadOnly);
QString stylesheet=QString::fromLatin1(stylefile.readAll());
a.setStyleSheet(stylesheet);
w.show();
~~~



这里的QWidget为选择器，表面后面花括号里的样式声明应用于QWidget

~~~
QWidget{
	background-color:rgb(79,79,79);
	color:rgb(235,235,235);
	font:12pt"新宋体";
}
~~~

这是声明部分，每个样式法则由属性和值构成



##### Qt样式表中的选择器类型

![image-20231122204534991](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231122204534991.png)

勘误：上图中第非子类选择器的.QPushButton的用途是“不包含子类”



~~~C++
#include "widget.h"
#include <QFile>
#include <QApplication>

int main(int argc, char *argv[])
{
    QApplication a(argc, argv);
    Widget w;
    w.resize(250,250);
    w.setWindowTitle("快把鼠标放进来");
    w.setToolTip("vocal还真浪费你几秒钟啊");
    //.setWindowIcon(QIcon("这里可以加上路径作为图标来显示"));
    QFile stylefile("myStyle.css");
    stylefile.open(QIODevice::ReadOnly);
    QString stylesheet=QString::fromLatin1(stylefile.readAll());
    a.setStyleSheet(stylesheet);
    w.show();
    return a.exec();
}

~~~

![image-20231122205515774](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231122205515774.png)

现在就是这样的背景颜色了

##### 子控件列表

我们可以改变子控件的图片，比如说默认的一个子控件是这样的，我们通过按上和下就可以改变这个数字

~~~C++
  QSpinBox*spinbox=new QSpinBox(this);
    grid2->addWidget(spinbox);
~~~



![image-20231122205949991](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231122205949991.png)

~~~CSS
QSpinBox::down-button{image:url(:/images/down.bmp);}
QSpinBox::up-button{image:url(:/images/up.bmp);}
~~~

把这个代码写到css文件里面，这样就可以改变样式的图片了

##### 状态

一个控件可以有很多种状态，在选择后面用分号(：)隔开

* 可以对状态取反(!)
* 状态还可以有子状态
* 子控件也可以使用状态

如:

~~~css
QPushButton:hover{
	background-color:black;
	color:yellow;
}
~~~

如果取否，也就是默认的情况下是这种状态，而不是鼠标掠过是这种状态

![image-20231122211908510](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231122211908510.png)

![image-20231122211945683](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231122211945683.png)

也可以在Qt Designer中编辑样式

QPushButton{

​	min-width:60px;					//主要内容区域的宽

​	min-height:60px;					//主要内容区域的高度

​	padding:0px 10px 0px 10px;	//填充区域

​	border:2px groove red;	//边框凹陷

​	border-radius:30px;		//边框半径

}

![image-20231122211203577](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231122211203577.png)

![image-20231122212256518](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231122212256518.png)

## P16_7.3通过按钮实现加减QLable数字

~~~C++
#ifndef MINUSPLUS_H
#define MINUSPLUS_H

#include <QWidget>
#include <QLabel>

class MinusPlus : public QWidget
{
    Q_OBJECT

public:
    MinusPlus(QWidget *parent = nullptr);
    ~MinusPlus();
public slots:
    void OnPlus();
    void OnMinus();

private:
    QLabel*lbl;
};
#endif // MINUSPLUS_H

~~~



~~~C++
#include "minusplus.h"
#include <QPushButton>
#include <QGridLayout>
MinusPlus::MinusPlus(QWidget *parent)
    : QWidget(parent)

{
    auto *plusBtn=new QPushButton("+",this);
    auto *minusBtn=new QPushButton("-",this);
    lbl=new QLabel("0",this);

    auto*grid=new QGridLayout(this);
    grid->addWidget(plusBtn,0,0,1,1);
    grid->addWidget(minusBtn,0,1,1,1);
    grid->addWidget(lbl,1,1,1,1);
    setLayout(grid);

    connect(plusBtn,&QPushButton::clicked,this,&MinusPlus::OnPlus);
    connect(minusBtn,&QPushButton::clicked,this,&MinusPlus::OnMinus);

}

MinusPlus::~MinusPlus()
{
    
}

void MinusPlus::OnPlus()
{
    int val=lbl->text().toInt();
    lbl->setText(QString::number(++val));
}

void MinusPlus::OnMinus()
{
    int val=lbl->text().toInt();
    lbl->setText(QString::number(--val));
}

~~~

这里很简单，只需要设置两个槽，然后对这两个槽做出具体实现：从label中读取字符串并转化成整数型，然后通过label指针再改变label上面text的值即可

## P17_7.4无边框透明背景

在Qt Designer中，可以以这种简单的图形化方式来修改Lable为一个图片，就是搜索pixmap并且以资源文件的形式进行替换

![image-20231123203040217](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231123203040217.png)

因为没有找到合适尺寸的照片外加照片都有水印，这里我就不设置icon图标了

通过拉三个label和两个QPushButton，然后采用栅格布局，反复调整之后我们就得到了这样的窗口

![image-20231123210028371](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231123210028371.png)

最后运行起来是这样的

![image-20231123210758040](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231123210758040.png)

这里的确认我把它连接到了退出上，只要按下确认就会退出。

用这个连接，就是用connect连接

![image-20231123211000029](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231123211000029.png)

如果使用转到槽，那么会使用名称连接

![image-20231123211034462](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231123211034462.png)

设置无边框透明背景，只需要把下列代码加入Widget的构造函数中中即可

~~~C++
setWindowFlag(Qt::FramelessWindowHint);//设置窗口无边框，设置后窗口无法移动
setAttribute(Qt::WA_TranslucentBackground,true);

border-image:url(:/images/button.png);//按钮样式
~~~

为了改变按钮的样式，其实也可以在designer中选中这个QPushButton，然后选择stylesheet属性，把上面的按钮样式写在里面

![image-20231124180937133](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231124180937133.png)

然后就变成这样了，因为我这里找的素材不好所以比例不是特别协调

![image-20231124181100158](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231124181100158.png)

然后活学活用，利用之前学到的CSS的样式表把字体颜色也改的醒目一点

![image-20231124181312366](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231124181312366.png)

然后他就和我的桌面背景融合在一起了

![image-20231124181440280](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231124181440280.png)

然后再给再个程序添加三个事件，通过给QWidget添加虚函数找到这三个鼠标事件并且添加定义，已达到拖拽这个窗口的目的



~~~C++
#include "widget.h"
#include "./ui_widget.h"
#include <QMouseEvent>

QPoint winPos;//鼠标窗口坐标

Widget::Widget(QWidget *parent):QWidget(parent),ui(new Ui::Widget)
{
    ui->setupUi(this);
    setWindowFlag(Qt::FramelessWindowHint);//设置窗口无边框，设置后窗口无法移动
    setAttribute(Qt::WA_TranslucentBackground,true);
}

Widget::~Widget()
{
    delete ui;
}


void Widget::on_pushbuttonquit_clicked()
{
    qApp->quit();
}

void Widget::mousePressEvent(QMouseEvent *event)
{
    QPoint mousePos=event->globalPosition().toPoint();
    QPoint topleft=this->geometry().topLeft();//窗口左上角相对于屏幕的坐标
    //将鼠标相对于窗口的坐标保存
    winPos=mousePos-topleft;
}

void Widget::mouseReleaseEvent(QMouseEvent *event)
{
    winPos=QPoint();
}

void Widget::mouseMoveEvent(QMouseEvent *event)
{
    QPoint mousePos=event->globalPosition().toPoint();//鼠标相对于屏幕的坐标
    QPoint endPos=mousePos-winPos;//窗口相对于桌面的坐标
    this->move(endPos);//设置窗口相对于桌面的目标(x),实现窗口移动
}


~~~

![image-20231124183639365](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231124183639365.png)

这样子的话，如果这个窗口有一个图标，那就可以拽着图片拖动窗口了，这样就解决了设置无边框窗口后无法拖动的问题了

## P18_8.1垂直布局与水平布局

### 布局管理

典型的应用程序由各种widget控件组成，这些可以放置在布局中。我们有两种选择

* 绝对定位
  * 如果调整窗口大小，widget的大小和位置不会改变
  * 不同平台上的应用程序看起来不同(一般很糟糕)
  * 更改应用程序中的字体可能会破坏布局
  * 如果决定改变布局，必须完全重做布局，既耗时又乏力
* 布局管理器
  * QHBoxLayout
  * QVBoxLayout
  * QFormLayout
  * GridLayout

由于绝对定位太不好用了，这里就不写了，没有哪个开发者愿意写绝对定位的QT程序，又费力又不好用

#### 垂直布局

~~~C++
#include "widget.h"
#include "./ui_widget.h"
#include <QPushButton>
#include<QVBoxLayout>

Widget::Widget(QWidget *parent):QWidget(parent),ui(new Ui::Widget)
{
    ui->setupUi(this);
    auto*add=new QPushButton("增",this);
    auto*del=new QPushButton("删",this);
    auto*mod=new QPushButton("改",this);
    auto*fin=new QPushButton("查",this);
    auto*vbox=new QVBoxLayout(this);
    add->setSizePolicy(QSizePolicy::Expanding,QSizePolicy::Expanding);
    del->setSizePolicy(QSizePolicy::Expanding,QSizePolicy::Expanding);
    mod->setSizePolicy(QSizePolicy::Expanding,QSizePolicy::Expanding);
    fin->setSizePolicy(QSizePolicy::Expanding,QSizePolicy::Expanding);
    vbox->addWidget(add);
    vbox->addWidget(del);
    vbox->addWidget(mod);
    vbox->addWidget(fin);

}

Widget::~Widget()
{
    delete ui;
}



~~~

![image-20231124185134982](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231124185134982.png)

设置成Expanding之后，不论上下左右怎么拉都不会有空位了，都会被填满

#### 水平布局

~~~C++
#include "widget.h"
#include "./ui_widget.h"
#include <QPushButton>
#include <QVBoxLayout>


Widget::Widget(QWidget *parent):QWidget(parent),ui(new Ui::Widget)
{
    ui->setupUi(this);
    auto vbox=new QVBoxLayout(this);
    auto hbox=new QHBoxLayout();

    auto okBtn=new QPushButton("OK",this);
    auto applyBtn=new QPushButton("Apply",this);

    //格子本身占据一个因子的空间，控件放在格子的最右边
    hbox->addWidget(okBtn,1,Qt::AlignRight);
    //格子本身占据0个因子的空间，控件填满整个格子
    hbox->addWidget(applyBtn,0);
    vbox->addStretch(1);
    vbox->addLayout(hbox);
    setLayout(vbox);
}

Widget::~Widget()
{
    delete ui;
}

~~~

![image-20231124190102527](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231124190102527.png)

这样不管怎么拉，这两个键都会出现在右下角不会再膨胀了

注意这里拉伸因子越高的widget和box增长的越快

#### 混合布局

即使采用混合布局，也要选择一个主布局，这里的主布局就是水平布局。谁是主布局，就把别的布局加到主布局里面

~~~C++
#include "widget.h"
#include "./ui_widget.h"
#include <QPushButton>
#include <QVBoxLayout>
#include <QListWidget>

Widget::Widget(QWidget *parent):QWidget(parent),ui(new Ui::Widget)
{
    ui->setupUi(this);
    auto vbox=new QVBoxLayout();
    auto hbox=new QHBoxLayout(this);
    auto*lw=new QListWidget(this);

    this->setWindowTitle("生死簿");

    lw->addItem("蔡徐坤");
    lw->addItem("陈立农");
    lw->addItem("范丞丞");
    lw->addItem("黄明昊");
    lw->addItem("林彦俊");
    lw->addItem("朱正廷");
    lw->addItem("王子异");
    lw->addItem("王琳凯");
    lw->addItem("尤长靖");

    auto*add=new QPushButton("新人入队",this);
    auto*rename=new QPushButton("更名改姓",this);
    auto*remove=new QPushButton("无常索命",this);
    auto*removeall=new QPushButton("轮回绝境",this);

    vbox->setSpacing(3);
    vbox->addStretch(1);

    vbox->addWidget(add);
    vbox->addWidget(rename);
    vbox->addWidget(remove);
    vbox->addWidget(removeall);
    hbox->addWidget(lw);

    hbox->addLayout(vbox);
    setLayout(hbox);
}

Widget::~Widget()
{
    delete ui;
}

~~~

![image-20231124192118049](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231124192118049.png)

## P19_8.2表格布局与网格布局

### QFormLayout

QFormLayout以两列形式显示。左栏由标签组成，右栏由QLineEdit或QSpinBox等输入widget组成

~~~C++
#include "widget.h"
#include "./ui_widget.h"
#include <QtWidgets>
Widget::Widget(QWidget *parent):QWidget(parent),ui(new Ui::Widget)
{
    ui->setupUi(this);
    auto*nameEdit=new QLineEdit(this);
    auto*addrEdit=new QLineEdit(this);
    auto*occpEdit=new QLineEdit(this);
    auto*formLayout=new QFormLayout(this);

    formLayout->setLabelAlignment(Qt::AlignRight|Qt::AlignCenter);
    formLayout->addRow("Name:",nameEdit);
    formLayout->addRow("Email:",addrEdit);
    formLayout->addRow("Age:",occpEdit);

    setLayout(formLayout);
}

Widget::~Widget()
{
    delete ui;
}


~~~



![image-20231124192955802](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231124192955802.png)

### QGridLayout

QGridLayout将其widget放置在网格中

~~~C++
#include "widget.h"
#include "./ui_widget.h"
#include <QtWidgets>
#include <QVector>
#include <QString>
Widget::Widget(QWidget *parent):QWidget(parent),ui(new Ui::Widget)
{
    ui->setupUi(this);
    auto*grid=new QGridLayout(this);
    grid->setSpacing(2);
    QVector<QString>values({"7","8","9","/",
                            "4","5","6","*",
                            "1","2","3","-",
                            "0",".","=","+"});
    int pos=0;
    for(int i=0;i<4;i++)
        for(int j=0;j<4;j++){
            auto*btn=new QPushButton(values[pos],this);
            btn->setSizePolicy(QSizePolicy::Expanding,QSizePolicy::Expanding);
            grid->addWidget(btn,i,j);
            pos++;
        }

    setLayout(grid);
}

Widget::~Widget()
{
    delete ui;
}



~~~

![image-20231124194141006](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231124194141006.png)

如果想要固定大小而不是随着拉伸，那可以把setSizePolicy换成setFixedSize



可以利用QGridLayout达到类似于QFormLayout的效果

而这需要使用到addWidget

addWidget方法有五个参数

* 第一个参数是子widget
* 接下来的两个参数是网格中放置的widget的行和列
* 最后的参数是rowspan和colspan。指定当前widget将跨越的行数。

~~~C++
#include "widget.h"
#include "./ui_widget.h"
#include <QtWidgets>
#include <QVector>
#include <QString>
Widget::Widget(QWidget *parent):QWidget(parent),ui(new Ui::Widget)
{
    ui->setupUi(this);

    auto*grid=new QGridLayout(this);
    grid->setVerticalSpacing(15);
    grid->setHorizontalSpacing(10);

    auto*title=new QLabel("Title:",this);
    grid->addWidget(title,0,0,1,1);
    title->setAlignment(Qt::AlignRight|Qt::AlignVCenter);

    auto*edit1=new QLineEdit(this);
    grid->addWidget(edit1,0,1,1,1);

    auto*author=new QLabel("Author:",this);
    grid->addWidget(author,1,0,1,1);
    author->setAlignment(Qt::AlignRight|Qt::AlignCenter);

    auto*edit2=new QLineEdit(this);
    grid->addWidget(edit2,1,1,1,1);

    auto*review=new QLabel("Review:",this);
    grid->addWidget(review,2,0,1,1);
    review->setAlignment(Qt::AlignRight|Qt::AlignCenter);

    auto*te=new QTextEdit(this);
    grid->addWidget(te,2,1,3,1);

    setLayout(grid);

}

Widget::~Widget()
{
    delete ui;
}
~~~

![image-20231124195610966](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231124195610966.png)

总的来说QGridLayout比QFormLayout更加强大

这里可以用te->sizeHint()获取到默认合适的高度

如果想忽略这个sizeHint的影响，可以用te->setSizePolicy，后面高度的参数写成QSizePolicy::Ignored或者是QSizePolicy::Maximum。前者忽略这个合适的大小，后者是把这个合适的宽度设置成了最大宽度，只能缩小不能放大。同理使用Minimum，那么就只能放大不能缩小了

## P20_9.1菜单栏与工具栏

* 菜单栏：对在应用程序中使用的action进行分组
* 工具栏：快速访问最常用的action

action是一种行动，我们如果触发了它，它就会响应

### 菜单栏

基类为QMainWindow,它有自己的布局，布局有一个中心区域，可以被任何类型的widget占用

![image-20231124200818519](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231124200818519.png)

~~~C++
#include "mainwindow.h"
#include <QtWidgets>
#include <QApplication>
MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
{
    auto*quit=new QAction("&Quit",this);
    QMenu*file=menuBar()->addMenu("&File");

    file->addAction(quit);
    connect(quit,&QAction::triggered,qApp,QApplication::quit);
}

MainWindow::~MainWindow()
{
}
~~~

这里可以按住ALT利用快捷键，按下F打开菜单栏，然后直接按Q退出即可。

这里的QAction的构造函数是有重载版本的，第一个参数可以填图标名，按照这种方式使用

~~~C++
QPixmap quitpix("../image/quit.png");
auto*quit=new QAction(quitpix,"&Quit",this);
~~~

这样子的话，菜单栏在名字面前会多一个图标

我这里没有分辨率合适的无水印素材，就不展示了



加入更多的快捷键和按键

~~~C++
#include "mainwindow.h"
#include <QtWidgets>
#include <QApplication>
MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
{
    QMenu*file=menuBar()->addMenu("&File");

    auto*quit=new QAction("&Quit",this);
    file->addAction(quit);
    quit->setShortcut(tr("CTRL+Q"));

    file->addSeparator(); 
        
    auto*newa=new QAction("&New",this);
    file->addAction(newa);

    auto*open=new QAction("&Open",this);
    file->addAction(open);



    qApp->setAttribute(Qt::AA_DontShowIconsInMenus,false);
//这里就是把菜单里面不显示图标的设置给关掉，就是让其显示
    connect(quit,&QAction::triggered,qApp,QApplication::quit);
}

MainWindow::~MainWindow()
{
}

~~~

这个可以直接用CTRL+Q退出程序

这个addSeparator()的作用就是分组，在中间加一个分割线

![image-20231124203230614](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231124203230614.png)

#### 可勾选菜单

~~~c++
#include "mainwindow.h"
#include <QtWidgets>
#include <QApplication>
MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
{
    QMenu*file=menuBar()->addMenu("&File");

    auto*quit=new QAction("&Quit",this);
    file->addAction(quit);
    quit->setShortcut(tr("CTRL+Q"));

    file->addSeparator();

    auto*newa=new QAction("&New",this);
    file->addAction(newa);

    auto*open=new QAction("&Open",this);
    file->addAction(open);


    qApp->setAttribute(Qt::AA_DontShowIconsInMenus,false);

    connect(quit,&QAction::triggered,qApp,QApplication::quit);

    status=new QAction("&View statusbar",this);
    status->setCheckable(true);//可以勾选
    status->setChecked(true);   //默认就是勾上了的
    file->addAction(status);

    statusBar();//激活状态栏
    status->setStatusTip("芝士状态栏");
    connect(status,&QAction::triggered,this,&MainWindow::toggleStatusbar);
}



MainWindow::~MainWindow()
{
}

void MainWindow::toggleStatusbar()
{
    if(status->isChecked())statusBar()->show();
    else statusBar()->hide();
}


~~~

![image-20231124205419494](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231124205419494.png)

这里把鼠标放到上面，并且在勾选激活状态就会显示状态栏，如果是非勾选状态就隐藏

### 工具栏

在QMainWindow里面可以通过addToolBar可以创建一个ToolBar对象，用一个指针接受它返回的对象，构造函数里面可以直接设置标题，然后将这个对象插入到窗口的顶部，也就是工具栏的区域

~~~c++
#include "mainwindow.h"
#include <QtWidgets>
#include <QApplication>
MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
{
   //......
        
    QToolBar*toolbar=addToolBar("main toolbar");
    toolbar->addAction(newa);
    toolbar->addAction(open);
    toolbar->addSeparator();
    toolbar->addAction(quit);
        
   //......
}

~~~

![image-20231124210058374](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231124210058374.png)

这个工具栏也是可以拖拽到各个区域的

![image-20231124210306436](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231124210306436.png)

通过加入这些代码，可以在widget构造的时候将状态栏显示为已经就绪，并且可以在里面输入文字

~~~C++
statusBar()->showMessage("已经就绪");
    auto *edit=new QTextEdit(this);
    setCentralWidget(edit);//把中心区域设置为文字编辑
~~~



![image-20231124210547959](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231124210547959.png)

## P21_10.1ModelView结构

类似Model-View(MVC)设计模式

* 数据:如数据库的一个数据表或SQL查询结果，内存中的一个StringList，或磁盘文件结构等

* Model:与数据通信，并为视图组件提供数据接口

* View:是屏幕上的界面组件，视图从数据模型获得每个数据项的索引模型(model index)，通过模型索引获取数据

* 代理：定制数据的界面显示和编辑方式。在标准的视图组件中，代理功能显示一个数据，当数据被编辑时，提供一个编辑器，一般是QLineEdit

  模型、视图和代理之间使用信号和槽通信

![image-20231124211233065](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231124211233065.png)

当然，上面这个图里面，其实数据也可以直接写到view里面展示

如果说是借助Model的，就像一个显示器需要数据线一样，那就是Model-Based

如果是直接显示的，就像黑板一样，就是Item-Based

他们各有各存在的价值，当然，肯定还是Model-Based强大，显示器肯定比黑板要好用。

不存在一个Model同时支持数据库，表格，链表，文件这样的数据，所以有各种各样的Model类型

View也不可能支持所有的数据格式，也会有多种的View,但是它和Model之间并不是一一对应的关系

### 数据模型

![image-20231124212606439](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231124212606439.png)

![image-20231124213243508](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231124213243508.png)

如果没有想要的数据类型只能自己写Model实现了，可以自己选一个合适的基类继承下来自己写，绿色的是纯虚的，红色的是可以直接使用的

#### 视图组件

显示数据时，只需要调用视图类的setModel()函数

![image-20231124213443706](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231124213443706.png)

红色的都是Model-Based,蓝色的都是Item-Based

#### 代理

* 代理就是在视图组件上为编辑数据提供编辑器
  * 如在表格组件中编辑一个单元格的数据时，缺省是使用一个QLineEdit编辑框。代理负责从数据模型中获取相应的数据，然后显示在编辑器里，修改数据后，又将其保存到数据模型中
* QAbstractItemDelegate是所有代理类的基类

#### Model/View结构的一些概念

数据模型中存储数据的基本单元都是项(item)，每个项有一个行号、一个列号，还有一个父项。

![image-20231126000931686](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231126000931686.png)

<u>QModelIndex表示模型索引的类。模型索引提供数据存储的一个临时指针。</u>

<u>持久索引模型由QPersistentModelIndex类提供</u>

~~~C++
//Table Model
QModelIndex indexA=model->index(0,0,QModelIndex());
QModelIndex indexC=model->index(2,1,QModelIndex());

//Tree Model
QModelIndex indexB=model->index(1,0,index);
~~~

项的角色：在为数据模型的一个项设置数据时，可以赋予其不同项的角色的数据。、

~~~C++
void QStandardItem::setData(const QVariant &value,int role=Qt::UserRole+1);

QVariant QStandardItem::data(int role=Qt::UserRole+1)const;
~~~

![image-20231126001705189](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231126001705189.png)

## P22_10.2QFileSystemModel

如同Windows的资源管理器。使用QFIleSystemModel提供的接口函数，可以创建、删除、重命名目录，可以获得文件名称、目录名称、文件大小等参数，还可以获得文件的详细信息。

![image-20231126172227554](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231126172227554.png)

~~~C++
#include "mainwindow.h"
#include "./ui_mainwindow.h"

MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
    , ui(new Ui::MainWindow)
{
    setWindowIcon(QIcon("C:/Users/Lenovo/Desktop/曹梦轩/shen.jpg"));
    ui->setupUi(this);
    model=new QFileSystemModel;//这里的model在.h中声明为QFileSystemModel对象
    model->setRootPath(QDir::currentPath());
    ui->treeView->setModel(model);
    ui->listView->setModel(model);
}

MainWindow::~MainWindow()
{
    delete ui;
}


~~~

![image-20231127193913822](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231127193913822.png)

打开之后就是这种效果，但是只有左边的treeView可以动，所以还要把treeView和listView根节点connect起来一起变化

![image-20231127194426646](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231127194426646.png)

现在点开左边的treeView,右边的listView也会随之改变

这个时候再加上一个点击文件显示大小的功能

~~~C++
void MainWindow::on_treeView_clicked(const QModelIndex &index)
{
    QString status=model->filePath(index);

    if(!model->isDir(index)){
        status+=" size:";
        int size=model->size(index);

        if(size<1024)status+=QString("%1KB").arg(size);
        else status+=QString("%.2MB").arg(size/1024.0);
    }
    statusBar()->showMessage(status);
}


~~~

这里的判断逻辑很简单，话说Linux下面有一句话叫“一切皆文件”，但是在windows里面，它表现出来的并不是这样，而是区分成文件和文件夹的形式。所以说，只要不是文件夹那它就是一个文件，如果是文件就直接在状态栏展示它的大小。

可以直接在designer里面选中treeview提升到槽自动生成两个代码框架，然后自己写个代码的具体实现就行了

![image-20231127200803326](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231127200803326.png)

## P23_10.3QStringListModel

* setStringList()函数可以初始化数据模型的字符串列表的内容
* 提供编辑和修改字符串列表数据的函数，如insertRows(),removeRows()，setData()等

![image-20231127202659395](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231127202659395.png)

~~~C++
MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
    , ui(new Ui::MainWindow)
{
    setFixedSize(300,200);
    setWindowIcon(QIcon("C:/Users/Lenovo/Desktop/曹梦轩/shen.jpg"));
    ui->setupUi(this);
    setWindowTitle("三个战犯和一个消愁");
    model=new QStringListModel;
    QStringList members={"曹梦轩","程宇政","刘伟博","胡瀚文"};
    model->setStringList(members);
    ui->listView->setModel(model);
    ui->listView->setEditTriggers(QAbstractItemView::NoEditTriggers);
}

~~~



![image-20231127212658701](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231127212658701.png)

然后接下来就是把增删改查分别绑定就行了，最好的方式是从ui里面右键转到槽然后写实现代码

~~~C++
void MainWindow::on_add_clicked()
{
    model->insertRow(model->rowCount());
    QModelIndex index=model->index(model->rowCount()-1);
    model->setData(index,"new person",Qt::DisplayRole);
    ui->listView->edit(index);
}
~~~

~~~C++
void MainWindow::on_del_clicked()
{
    QModelIndex index=ui->listView->currentIndex();
    model->removeRow(index.row());
}

~~~

~~~C++
void MainWindow::on_reset_clicked()
{
    model->modelReset()
}

~~~

~~~C++
void MainWindow::on_clear_clicked()
{
    model->removeRows(0,model->rowCount());
}
~~~



## P24_10.4QStandarditemModel

* 以item数据为基础的标准数据模型类，通常与QTableView组合成Model/View结构，实现通用的二维数据的管理功能
* QDataWidgetMapper类允许在widget集合中查看和编辑从模型中获得的信息，从QabstractitemModel派生的任何模型都可以用作数据源

![image-20231217172624172](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231217172624172.png)

~~~c++
#include "mainwindow.h"
#include "./ui_mainwindow.h"

MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
    , ui(new Ui::MainWindow)
{
    setFixedSize(365,200);
    ui->setupUi(this);
    setWindowTitle("WBG-T1 ROUND3");
    setupModel();
    ui->tableView->setModel(model);//把view和数据关联起来
}


MainWindow::~MainWindow()
{
    delete ui;
}


void MainWindow::setupModel(){
    model=new QStandardItemModel(5,3,this);

    QStringList names;
    names<<"惹晒"<<"喂喂"<<"销户"<<"莱特"<<"青松";

        QStringList KD;
    KD<<"0-6-3"<<"2-4-3"<<"0-3-0"<<"2-2-1"<<"1-4-4";

    QStringList rank;
    rank<<"5"<<"3"<<"1"<<"2"<<"4";

    for(int row=0;row<5;++row){
        QStandardItem*item=new QStandardItem(names[row]);
        model->setItem(row,0,item);

        item=new QStandardItem(KD[row]);
        model->setItem(row,1,item);

        item=new QStandardItem(rank[row]);
        model->setItem(row,2,item);
    }
}

~~~

初步运行的效果就是这样的

![image-20231217181120421](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231217181120421.png)

但是这里有一个问题，上面没有显示每一行的数据分别对应着什么，而且布局也不美观，所以还需要再进行修改。

~~~c++
ui->tableView->horizontalHeader()->setSectionResizeMode(QHeaderView::ResizeToContents);
~~~

这里的作用是通过内容来改变大小

~~~c++
S
~~~

通过这个来设置标题

~~~c++
    mapper=new QDataWidgetMapper(this);
    mapper->setModel(model);

    mapper->addMapping(ui->nameEdit,0);
    mapper->addMapping(ui->kdEdit,1);
    mapper->addMapping(ui->rankEdit,2);
~~~

然后再用这个把mapper和model绑定起来，这样就可以自己更改数据了。这里其实是在widget和model之中添加映射。如果方向为水平（默认），则第二个参数为模型中的列，否则就是行了。

再指定先从Model中拿出第一个的数据

~~~C++
    mapper->toFirst();//让它指定到第一个，去model中拿出第一行数据
~~~

现在的效果就是这样了

![image-20231217183135709](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231217183135709.png)

最后就是加一个实时更新的功能

~~~c++
void MainWindow::updateButtons(int row)
{
    //先判断一下这个按钮还是不是有效的
    ui->previousButton->setEnabled(row>0);
    ui->nextButton->setEnabled(row<model->rowCount()-1);
    
    //设置当前被选中的一个格子
    QModelIndex index=model->index(row,0);
    //在tableview中同步更新成模型的index
    ui->tableView->setCurrentIndex(index);
}
~~~

再把它绑定起来就好了

~~~C++
connect(mapper,&QDataWidgetMapper::currentIndexChanged,this,&MainWindow::updateButtons);
~~~

这个时候还有一个问题，就是点右边按钮的时候无法与左边产生联动，只能通过上一个和下一个改变选定目标

这里实现看似应该会非常复杂，但其实很简单，只要把右边的这个tableview加一个槽函数就行了，通过点击这个tableview的某个位置获取其当前的行索引，然后同步更新到mapper里面去

~~~c++
void MainWindow::on_tableView_clicked(const QModelIndex &index)
{
    mapper->setCurrentIndex(index.row());
}
~~~

其实还有更简单的办法，根本不需要额外增加一个槽，这个办法就是再绑定一个lambda函数，lambda的函数体就是上面的这一行。

~~~C++
    connect(ui->tableView,&QTableView::clicked,this,
            [this](const QModelIndex&index){mapper->setCurrentIndex(index.row());});
~~~

这里通过捕获this,来获取this里面的mapper.

完整的实现源码

~~~c++
#include "mainwindow.h"
#include "./ui_mainwindow.h"

MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
    , ui(new Ui::MainWindow)
{
    setFixedSize(365,200);
    ui->setupUi(this);
    setWindowTitle("WBG-T1 ROUND3");
    setupModel();
    ui->tableView->setModel(model);//把view和数据关联起来
    ui->tableView->horizontalHeader()->setSectionResizeMode(QHeaderView::ResizeToContents);
    QStringList headers={"姓名","战绩","排名"};
    model->setHorizontalHeaderLabels(headers);
    mapper=new QDataWidgetMapper(this);
    mapper->setModel(model);
    mapper->addMapping(ui->nameEdit,0);
    mapper->addMapping(ui->kdEdit,1);
    mapper->addMapping(ui->rankEdit,2);


    connect(ui->previousButton,&QAbstractButton::clicked,mapper,&QDataWidgetMapper::toPrevious);
    connect(ui->nextButton,&QAbstractButton::clicked,mapper,&QDataWidgetMapper::toNext);
    connect(mapper,&QDataWidgetMapper::currentIndexChanged,this,&MainWindow::updateButtons);
    connect(ui->tableView,&QTableView::clicked,this,
            [this](const QModelIndex&index){mapper->setCurrentIndex(index.row());});
    mapper->toFirst();//让它指定到第一个，去model中拿出第一行数据
}


MainWindow::~MainWindow()
{
    delete ui;
}


void MainWindow::setupModel(){
    model=new QStandardItemModel(5,3,this);

    QStringList names;
    names<<"惹晒"<<"喂喂"<<"销户"<<"莱特"<<"青松";

        QStringList KD;
    KD<<"0-6-3"<<"2-4-3"<<"0-3-0"<<"2-2-1"<<"1-4-4";

    QStringList rank;
    rank<<"5"<<"3"<<"1"<<"2"<<"4";

    for(int row=0;row<5;++row){
        QStandardItem*item=new QStandardItem(names[row]);
        model->setItem(row,0,item);

        item=new QStandardItem(KD[row]);
        model->setItem(row,1,item);

        item=new QStandardItem(rank[row]);
        model->setItem(row,2,item);
    }
}

void MainWindow::updateButtons(int row)
{
    ui->previousButton->setEnabled(row>0);
    ui->nextButton->setEnabled(row<model->rowCount()-1);

    QModelIndex index=model->index(row,0);
    //这里是选中某一项，其实不是很合理，应该选中某一行
    //ui->tableView->setCurrentIndex(index);
    ui->tableView->selectRow(index.row());
}
~~~

## P25_10.5QStyledItemDelegate

### 自定义代理

![image-20231217190651039](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231217190651039.png)

在标准的视图组件中，代理功能显示一个数据，当数据被编辑的时候，提供一个编辑器，一般来说是QLineEdit

QAbstractItemDelegate是**所有代理类的**抽象基类。

QStyledItemDelegate是视图组件使用的缺省的代理类，QItemDelegate也是类似功能的类。区别在于，QStyleItemDelegate可以使用当前的样式表来绘制组件，所以用QStyleItemDelegate明显要更好一点

![image-20231217190936673](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231217190936673.png)

必须实现以下几个函数：

* createEdit():创建用于编辑模型数据的widget组件，如QSpinBox,QcomboBox
* setEditorData():从数据模型获取数据，供widget组件进行编辑
* setModelData():将widget上的数据更新到数据模型
* updateEditorGeometry():用于给widget组件设置一个合适的大小

比如上面一个的案例，我想把里面的一个lineEdit换成SpinBox

~~~c++
class IntSpinDelegate : public QStyledItemDelegate
{
    Q_OBJECT
public:
    explicit IntSpinDelegate(QObject *parent = nullptr);

    // QAbstractItemDelegate interface
public:
    QWidget *createEditor(QWidget *parent, const QStyleOptionViewItem &option, const QModelIndex &index) const override;
    void setEditorData(QWidget *editor, const QModelIndex &index) const override;
    void setModelData(QWidget *editor, QAbstractItemModel *model, const QModelIndex &index) const override;
    void updateEditorGeometry(QWidget *editor, const QStyleOptionViewItem &option, const QModelIndex &index) const override;
};
~~~

这时候只需要继承这个QStyledItemDelegate写一个自己的子类就行了，然后右键这个父类加四个虚函数上去然后自己写实现

这是完整的实现代码部分

~~~c++
#include "intspindelegate.h"
#include <QSpinBox>

IntSpinDelegate::IntSpinDelegate(QObject *parent)
    : QStyledItemDelegate{parent}
{

}

QWidget *IntSpinDelegate::createEditor(QWidget *parent, const QStyleOptionViewItem &option, const QModelIndex &index) const
{
    QSpinBox*spinbox=new QSpinBox(parent);
    spinbox->setMinimum(0);
    spinbox->setMaximum(200);
    return spinbox;//返回此编辑器
}

void IntSpinDelegate::setEditorData(QWidget *editor, const QModelIndex &index) const
{
    //从数据模型中获取数据，显示到代理组件中
    //获取数据模型的模型索引指向的单元的数据
    int value=index.model()->data(index,Qt::EditRole).toInt();
    //把QWidget转换成QSpinBox
    QSpinBox*spinBox=static_cast<QSpinBox*>(editor);
    spinBox->setValue(value);   //设置编辑器的数值
}

void IntSpinDelegate::setModelData(QWidget *editor, QAbstractItemModel *model, const QModelIndex &index) const
{
    //将代理组件的数据保存到数据模型中
    QSpinBox*spinBox=static_cast<QSpinBox*>(editor);
    spinBox->interpretText();   //解释数据，如果数据被修改之后就会触发信号
    int value=spinBox->value();//获取到spinBox的值
    model->setData(index,value,Qt::EditRole);   //更新到数据模型中区

}

void IntSpinDelegate::updateEditorGeometry(QWidget *editor, const QStyleOptionViewItem &option, const QModelIndex &index) const
{
    //就是说index不会被使用，编译器你别警告了
    Q_UNUSED(index);
    //设置组件大小
    editor->setGeometry(option.rect);   //option里面包含了几何大小
}

~~~

对应的mainwindow也要加上这两行

~~~c++
    IntSpinDelegate *intSpinDelegate=new IntSpinDelegate(this);
    ui->tableView->setItemDelegateForColumn(2,intSpinDelegate);//这里的2就是index为2的列，也就是排名那一列,也就是说我们把这一列的缺省值给自己自定义更换成了自己实现的一个类型，只要我们实现了这四个函数就可以进行替换了。
~~~

## P26_11.1Qt Designer

在Qt Designer中，我们可以获得所见即所得的界面编辑，用GUI界面编辑来实现GUI界面

* 项目组织文件CMakeLists.txt存储项目设置
* 主程序入口文件main.cpp实现main函数
* 界面文件xxxx.ui,使用XML格式描述
* xxxx.h是所设计的窗体类的头文件，，xxxx.cpp是实现文件，任何窗体或者页面组件都是用类封装的

![image-20231219163402734](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231219163402734.png)

UIC(User Interface Compiler)读取这个XML格式用户界面定义(ui)文件，并且创建相应的C++文件，最后调用的时候呢是用文件里面的setupUi函数来初始化这个界面的，在mainwindow里面用Ui::MainWindow*ui这样的指针来进行调用的。

![image-20231219170137575](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231219170137575.png)

![image-20231219170204213](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231219170204213.png)

![image-20231219170226424](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231219170226424.png)

比如说我上次设计ui界面所对应的XML格式文件就是这样的

![image-20231219163803378](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231219163803378.png)

![image-20231219163719525](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231219163719525.png)

QtDesigner的图形化带来了很多方便。

### 1.类似蓝图的方式绑定信号槽

比如你有一个按钮，一个窗口，你想通过这个按钮来绑定某个窗口的一个事件，你可以点击上面的![image-20231219164228848](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231219164228848.png)

然后点击这个按钮，把它拖向这个窗口，就可以直接自动匹配，选择信号槽直接完成绑定

我个人感觉有点像UE5的蓝图了(以前也尝试过UE5方向有所了解)，那么我个人所理解的QT，也应该是像虚幻引擎那样，可以用蓝图和代码混合的方式，以高效的开发效率打造完美的产品。

### 2.绑定伙伴Buddy

比如有一个TextLable,一个LineEdit，可以点击上面的进行绑定

![image-20231219164822131](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231219164822131.png)

![image-20231219164759185](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231219164759185.png)

然后他们两个就会被绑定到一起了，拖动的时候也是一起拖

这样的好处就是在布局的时候可以体现它们的一体性了

### 3.布局

布局的时候拖动几个控件然后点布局就行了，需要注意的是布局是有技巧的，需要给desinger足够的提示，比如说通过设置一个弹簧来改变目前布局的样式使其美观



### 4.编辑TAB顺序

![image-20231219165103042](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231219165103042.png)

每个控件都有对应的顺序，可以改变控件的TAB ORDER顺序

## P27_动态加载UI文件

动态加载ui文件是指运行的时候才去加载，而不是编译的时候去确定然后加载

* 使用QtuTools模块中的QuiLoader类，基于ui文件，在运行时动态创建用户界面
* Header:#include<QUiLoader>
* CMake:
* find_package(Qt6 REQUIRED COMPONENTS UiTools) 
* target_link_libraries(mytarget PRIVATE QT6::UiTools)
* qmake(QT+=Uitools)
* Inherits:QObject

我个人觉得qmake虽然方便，但是QT6之后停止更新了，因为就连QT官方都认为，现在的CMake更适合做Qt开发，因为CMake它的上限更高，对新的标准的支持也是很好，所以我个人的代码构建工具还是使用CMake而不会转而去使用qmake

动态加载ui文件，就是增加一个.qrc的资源文件，然后加入到CMake中，然后加载现有文件，把.ui文件放到这个qrc里面去。

> 我写QUiLoader的时候QT并没有给我自动补全提示，这不是因为没有这个头文件，而是因为我还没有加入这个模块，一般来说这种情况需要自己手动去Qt的帮助里面去查文档，像这样，连CMake怎么去写都给提示了
>
> ![image-20231220175729733](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231220175729733.png)

### 在窗口类中添加需要控制的ui控件和slot函数

~~~c++
#ifndef WIDGET_H
#define WIDGET_H

#include <QWidget>
#include <QUiLoader>
#include <QFile>
#include <QSpinBox>
#include <QLabel>
#include <QVBoxLayout>

QT_BEGIN_NAMESPACE
namespace Ui { class Widget; }
QT_END_NAMESPACE

class Widget : public QWidget
{
    Q_OBJECT

public:
    Widget(QWidget *parent = nullptr);
    ~Widget();

private slots:
    void on_inputSpinBox1_valueChanged(int value);
    void on_inputSpinBox2_valueChanegd(int value);
private:
    QSpinBox*ui_inputSpinBox1;
    QSpinBox*ui_inputSpinBox2;
    QLabel*ui_outputWidget;

};
#endif // WIDGET_H

~~~

~~~C++
#include "widget.h"


Widget::Widget(QWidget *parent)
    : QWidget(parent)
{
    QUiLoader loader;
    QFile file("D:/qt/axb_window/mainwindow.ui");
    file.open(QFile::ReadOnly);
    QWidget*formWidget=loader.load(&file,this);
    file.close();
        //findChild是Widget的函数，widget归根到底就是一个QObject,这个函数就是到当前的树下面找一个对象
        //比如这里就是找inputSpinBox1的对象
    ui_inputSpinBox1=findChild<QSpinBox*>("inputSpinBox1");
    ui_inputSpinBox2=findChild<QSpinBox*>("inputSpinBox2");
    ui_outputWidget=findChild<QLabel*>("outputWidget");
    QMetaObject::connectSlotsByName(this);
    QVBoxLayout*layout=new QVBoxLayout;
    layout->addWidget(formWidget);
    setLayout(layout);
    setWindowTitle(tr("Caculator Builder"));
}

void Widget::on_inputSpinBox1_valueChanged(int value){
    
}

void Widget::on_inputSpinBox2_valueChanegd(int value)
{
    ui_outputWidget->setText(QString::number(value+ui_inputSpinBox2->value());
}
~~~

## P28_QWidget和QWindow

* QWidget:是桌面环境中典型的用户界面元素，在Windows、Linux和macOS上提供了本机外观。用于创建大型桌面应用程序
* QWindow:表示底层窗口系统中的窗口

* 应用程序通常将QWidget用于UI，而不是直接使用QWindow
* 尽管如此，当想要将依赖关系保持在最小或者想要直接使用OPENGL时，仍然可以使用QBackingStore或者是QOpenGLContext直接渲染到QWindow

### QWidget和QWindow的差别

![image-20231220190228987](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231220190228987.png)

总的来说QWidget比QWindow强大，但是OPENGL方面还是QWindow更轻量级

![image-20231220190749424](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20231220190749424.png)
