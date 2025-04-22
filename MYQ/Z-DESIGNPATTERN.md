#  c++单例模式

> 定义：单例模式是创建型设计模式，指的是在系统的生命周期中只能产生一个实例(对象)，确保该类的唯一性。
>
> 一般遇到的写进程池类、日志类、内存池（用来缓存数据的结构，在一处写多出读或者多处写多处读）的话都会用到单例模式

**实现方法：**全局只有一个实例也就意味着不能用new调用构造函数来创建对象，因此构造函数必须是虚有的。但是由于不能new出对象，所以类的内部必须提供一个函数来获取对象，而且由于不能外部构造对象，因此这个函数不能是通过对象调出来，换句话说这个函数应该是属于对象的，很自然我们就想到了用static。由于静态成员函数属于整个类，在类实例化对象之前就已经分配了空间，而类的非静态成员函数必须在类实例化后才能有内存空间。

单例模式的要点总结：

1. 全局只有一个实例，用static特性实现，构造函数设为私有
2. 通过公有接口获得实例
3. 线程安全
4. 禁止拷贝和赋值

单例模式可以**分为懒汉式和饿汉式**，两者之间的区别在于创建实例的时间不同：懒汉式指系统运行中，实例并不存在，只有当需要使用该实例时，才会去创建并使用实例(这种方式要考虑线程安全)。饿汉式指系统一运行，就初始化创建实例，当需要时，直接调用即可。（本身就线程安全，没有多线程的问题）

## 懒汉式

- 普通懒汉式会让线程不安全

  因为不加锁的话当线程并发时会产生多个实例，导致线程不安全

  ```c++
  ///  普通懒汉式实现 -- 线程不安全 //
  #include <iostream> // std::cout
  #include <mutex>    // std::mutex
  #include <pthread.h> // pthread_create
  
  class SingleInstance
  {
  public:
      // 获取单例对象
      static SingleInstance *GetInstance();
      // 释放单例，进程退出时调用
      static void deleteInstance();
  	// 打印单例地址
      void Print();
  private:
  	// 将其构造和析构成为私有的, 禁止外部构造和析构
      SingleInstance();
      ~SingleInstance();
      // 将其拷贝构造和赋值构造成为私有函数, 禁止外部拷贝和赋值
      SingleInstance(const SingleInstance &signal);
      const SingleInstance &operator=(const SingleInstance &signal);
  private:
      // 唯一单例对象指针
      static SingleInstance *m_SingleInstance;
  };
  
  //初始化静态成员变量
  SingleInstance *SingleInstance::m_SingleInstance = NULL;
  
  SingleInstance* SingleInstance::GetInstance()
  {
  	if (m_SingleInstance == NULL)
  	{
  		m_SingleInstance = new (std::nothrow) SingleInstance;  // 没有加锁是线程不安全的，当线程并发时会创建多个实例
  	}
      return m_SingleInstance;
  }
  
  void SingleInstance::deleteInstance()
  {
      if (m_SingleInstance)
      {
          delete m_SingleInstance;
          m_SingleInstance = NULL;
      }
  }
  
  void SingleInstance::Print()
  {
  	std::cout << "我的实例内存地址是:" << this << std::endl;
  }
  
  SingleInstance::SingleInstance()
  {
      std::cout << "构造函数" << std::endl;
  }
  
  SingleInstance::~SingleInstance()
  {
      std::cout << "析构函数" << std::endl;
  }
  ///  普通懒汉式实现 -- 线程不安全  //
  ```

- 线程安全、内存安全的懒汉式

  上述代码出现的问题：

  1. GetInstance()可能会引发竞态条件，第一个线程在if中判断 `m_instance_ptr`是空的，于是开始实例化单例;同时第2个线程也尝试获取单例，这个时候判断`m_instance_ptr`还是空的，于是也开始实例化单例;这样就会实例化出两个对象,这就是线程安全问题的由来

     解决办法：①加锁。②局部变量实例

  2. 类中只负责new出对象，却没有负责delete对象，因此只有构造函数被调用，析构函数却没有被调用;因此会导致内存泄漏。

     解决办法：使用共享指针

  > c++11标准中有一个特性：如果当变量在初始化的时候，并发同时进入声明语句，并发线程将会阻塞等待初始化结束。这样保证了并发线程在获取静态局部变量的时候一定是初始化过的，所以具有线程安全性。因此这种懒汉式是最推荐的，因为：
  >
  > 1. 通过局部静态变量的特性保证了线程安全 (C++11, GCC > 4.3, VS2015支持该特性);
  > 2. 不需要使用共享指针和锁
  > 3. get_instance()函数要返回引用而尽量不要返回指针，

  ```c++
  ///  内部静态变量的懒汉实现  //
  class Singleton
  {
  public:
      ~Singleton(){
          std::cout<<"destructor called!"<<std::endl;
      }
      //或者放到private中
      Singleton(const Singleton&)=delete;
      Singleton& operator=(const Singleton&)=delete;
      static Singleton& get_instance(){
          //关键点！
          static Singleton instance;
          return instance;
      }
      //不推荐，返回指针的方式
      /*static Singleton* get_instance(){
          static Singleton instance;
          return &instance;
  	}*/
  private:
      Singleton(){
          std::cout<<"constructor called!"<<std::endl;
      }
  };
  
  ```

  > 使用锁、共享指针实现的懒汉式单例模式
  >
  > - 基于 shared_ptr, 用了C++比较倡导的 RAII思想，用对象管理资源,当 shared_ptr 析构的时候，new 出来的对象也会被 delete掉。以此避免内存泄漏。
  > - 加了锁，使用互斥量来达到线程安全。这里使用了两个 if判断语句的技术称为**双检锁**；好处是，只有判断指针为空的时候才加锁，避免每次调用 get_instance的方法都加锁，锁的开销毕竟还是有点大的。
  >
  > 不足之处在于： 使用智能指针会要求用户也得使用智能指针，非必要不应该提出这种约束; 使用锁也有开销; 同时代码量也增多了，实现上我们希望越简单越好。

  ```c++
  #include <iostream>
  #include <memory> // shared_ptr
  #include <mutex>  // mutex
  
  // version 2:
  // with problems below fixed:
  // 1. thread is safe now
  // 2. memory doesn't leak
  
  class Singleton {
  public:
      typedef std::shared_ptr<Singleton> Ptr;
      ~Singleton() {
          std::cout << "destructor called!" << std::endl;
      }
      Singleton(Singleton&) = delete;
      Singleton& operator=(const Singleton&) = delete;
      static Ptr get_instance() {
  
          // "double checked lock"
          if (m_instance_ptr == nullptr) {
              std::lock_guard<std::mutex> lk(m_mutex);
              if (m_instance_ptr == nullptr) {
                  m_instance_ptr = std::shared_ptr<Singleton>(new Singleton);
              }
          }
          return m_instance_ptr;
      }
  
  
  private:
      Singleton() {
          std::cout << "constructor called!" << std::endl;
      }
      static Ptr m_instance_ptr;
      static std::mutex m_mutex;
  };
  
  // initialization static variables out of class
  Singleton::Ptr Singleton::m_instance_ptr = nullptr;
  std::mutex Singleton::m_mutex;
  ```

  

## 饿汉式

```c++

// 饿汉实现 /
class Singleton
{
    
public:
    // 获取单实例
    static Singleton* GetInstance();
    // 释放单实例，进程退出时调用
    static void deleteInstance();
    // 打印实例地址
    void Print();

private:
    // 将其构造和析构成为私有的, 禁止外部构造和析构
    Singleton();
    ~Singleton();

    // 将其拷贝构造和赋值构造成为私有函数, 禁止外部拷贝和赋值
    Singleton(const Singleton &signal);
    const Singleton &operator=(const Singleton &signal);

private:
    // 唯一单实例对象指针
    static Singleton *g_pSingleton;
};

// 代码一运行就初始化创建实例 ，本身就线程安全
Singleton* Singleton::g_pSingleton = new (std::nothrow) Singleton;

Singleton* Singleton::GetInstance()
{
    return g_pSingleton;
}

void Singleton::deleteInstance()
{
    if (g_pSingleton)
    {
    
        delete g_pSingleton;
        g_pSingleton = NULL;
    }
}

void Singleton::Print()
{
    std::cout << "我的实例内存地址是:" << this << std::endl;
}

Singleton::Singleton()
{
    std::cout << "构造函数" << std::endl;
}

Singleton::~Singleton()
{
    std::cout << "析构函数" << std::endl;
}
// 饿汉实现 /
```

## 面试题

- 懒汉模式和恶汉模式的实现（判空！！！加锁！！！），并且要能说明原因（为什么判空两次？）
- 构造函数的设计（为什么私有？除了私有还可以怎么实现（进阶）？）
- 对外接口的设计（为什么这么设计？）
- 单例对象的设计（为什么是static？如何初始化？如何销毁？（进阶））
- 对于C++编码者，需尤其注意C++11以后的单例模式的实现（为什么这么简化？怎么保证的（进阶））


# Observe模式
## 定义

又叫做观察者模式，被观察者叫做subjec，观察者叫做observer

**观察者模式(Observer Pattern)**： 定义对象间一种一对多的依赖关系，使得当每一个对象改变状态，则所有依赖于它的对象都会得到通知并自动更新。

观察者模式所做的工作其实就是在解耦，让耦合的双方都依赖于抽象而不是具体，从而使得各自的变化都不会影响另一边的变化。当一个对象的改变需要改变其他对象的时候，而且它不知道具体有多少对象有待改变的时候，应该考虑使用观察者模式。一旦观察目标的状态发生改变，所有的观察者都将得到通知。具体来说就是被观察者需要用一个容器比如vector存放所有观察者对象，以便状态发生变化时给观察着发通知。观察者内部需要实例化被观察者对象的实例（需要前向声明）

<img src="https://s2.loli.net/2022/07/09/mXhfzvFaTi6BApy.webp" alt="img" style="zoom:80%;float:left" />

**观察者模式中主要角色--2个接口,2个类**

1. 抽象主题（Subject）角色(接口)：主题角色将所有对观察者对象的引用保存在一个集合中，每个主题可以有任意多个观察者。 抽象主题提供了增加和删除观察者对象的接口。
2. 抽象观察者（Observer）角色(接口)：为所有的具体观察者定义一个接口，在观察的主题发生改变时更新自己。
3. 具体主题（ConcreteSubject）角色(1个)：存储相关状态到具体观察者对象，当具体主题的内部状态改变时，给所有登记过的观察者发出通知。具体主题角色通常用一个具体子类实现。
4. 具体观察者（ConcretedObserver）角色(多个)：存储一个具体主题对象，存储相关状态，实现抽象观察者角色所要求的更新接口，以使得其自身状态和主题的状态保持一致。

```cpp
// 观察者接口
class Observer {
public:
    virtual void update(int newState) = 0;
    virtual ~Observer() {}
};

// 被观察者类
class Subject {
private:
    std::vector<std::shared_ptr<Observer>> observers;
    int state;
public:
    void attach(std::shared_ptr<Observer> observer) {
        observers.push_back(observer);
    }

    void detach(std::shared_ptr<Observer> observer) {
        observers.erase(std::remove(observers.begin(), observers.end(), observer), observers.end());
    }

    void notify() {
        for (auto& observer : observers) {
            observer->update(state);
        }
    }

    void setState(int newState) {
        state = newState;
        notify();  // 状态变化时通知所有观察者
    }
};

// 具体观察者
class ConcreteObserver : public Observer {
private:
    std::string name;
public:
    ConcreteObserver(std::string n) : name(n) {}
    void update(int newState) override {
        std::cout << "Observer " << name << " received new state: " << newState << std::endl;
    }
};

// 测试
int main() {
    auto subject = std::make_shared<Subject>();

    auto observer1 = std::make_shared<ConcreteObserver>("A");
    auto observer2 = std::make_shared<ConcreteObserver>("B");

    subject->attach(observer1);
    subject->attach(observer2);

    subject->setState(10);  // 通知所有观察者
    subject->setState(20);  // 观察者再次收到通知

    return 0;
}
```
### **执行结果**
```
Observer A received new state: 10
Observer B received new state: 10
Observer A received new state: 20
Observer B received new state: 20
```

---

## **4. 观察者模式的应用场景**
观察者模式常用于需要**实时更新**的场景，比如：
- **GUI框架**（按钮的点击事件通知多个监听者）
- **发布-订阅模式**（消息队列、事件系统）
- **股票行情系统**（投资者关注股票价格变化）
- **游戏引擎**（游戏对象状态变化通知 UI 组件）

---

## **5. 观察者模式的优缺点**
### **✅ 优点**
1. **解耦**：观察者和被观察者解耦，便于维护和扩展。
2. **灵活性**：可以动态添加/删除观察者，而无需修改被观察者的代码。
3. **符合开闭原则**：新增观察者不影响现有代码。

### **❌ 缺点**
1. **性能开销**：当观察者数量较多时，通知操作可能会影响性能。
2. **可能导致循环依赖**：如果观察者和被观察者之间存在复杂的引用关系，可能会导致循环引用问题。

---

## **6. 进阶优化**
### **(1) 异步通知**
- 可以使用 **事件队列** 或 **消息队列（如 Kafka、RabbitMQ）** 来异步通知，提高性能。

### **(2) 结合智能指针**
- 例如 `std::weak_ptr` 避免循环引用，防止内存泄漏。

### **(3) 过滤通知**
- 让观察者根据不同的事件类型决定是否接收通知，减少不必要的调用。

---

## **7. 总结**
观察者模式是一种**解耦、灵活**的设计模式，适用于需要**实时更新、订阅发布**的场景，但在高并发和大规模订阅情况下需要优化性能（如异步处理）。


# 工厂模式
**工厂模式**是一种**创建型设计模式**，它提供了一种**封装对象创建逻辑**的方法，而不直接在代码中使用 `new` 关键字创建对象，从而提高代码的**可扩展性**和**解耦性**。

---

## **1. 为什么要使用工厂模式？**
在面向对象编程中，直接使用 `new` 创建对象会导致**代码高度耦合**，不利于后续的扩展和维护。例如：
```cpp
class Product {
public:
    Product() { std::cout << "Product created\n"; }
};
int main() {
    Product* p = new Product();  // 直接使用 new，代码耦合
    delete p;
}
```
如果以后 `Product` 类发生变化，比如新增子类或修改构造逻辑，所有 `new Product()` 的地方都要修改，维护成本高。**工厂模式**的核心思想就是**把对象创建逻辑封装起来**，外部代码不直接 `new`，而是通过工厂方法获取实例。

---

## **2. 工厂模式的分类**
工厂模式主要有 **三种类型**：
1. **简单工厂模式（Simple Factory）** 🏭
2. **工厂方法模式（Factory Method）** 🏭🏭
3. **抽象工厂模式（Abstract Factory）** 🏭🏭🏭

---

## **3. 工厂模式的实现**
### **(1) 简单工厂模式（Simple Factory Pattern）**
**特点**：
- 由一个**工厂类**负责创建不同类型的对象。
- 适用于**对象种类较少**的情况。

**示例**：
```cpp
#include <iostream>

// 产品基类
class Product {
public:
    virtual void show() = 0;
    virtual ~Product() {}
};

// 具体产品 A
class ProductA : public Product {
public:
    void show() override { std::cout << "Product A\n"; }
};

// 具体产品 B
class ProductB : public Product {
public:
    void show() override { std::cout << "Product B\n"; }
};

// 简单工厂类
class SimpleFactory {
public:
    static Product* createProduct(char type) {
        if (type == 'A') return new ProductA();
        if (type == 'B') return new ProductB();
        return nullptr;
    }
};

// 测试
int main() {
    Product* p1 = SimpleFactory::createProduct('A');
    p1->show();
    delete p1;

    Product* p2 = SimpleFactory::createProduct('B');
    p2->show();
    delete p2;

    return 0;
}
```
### **优缺点**
✅ **优点**：
- 工厂类封装了创建逻辑，客户端只需调用 `createProduct` 方法即可。
- 代码简洁，适用于产品种类较少的情况。

❌ **缺点**：
- **违反开放封闭原则（OCP）**：如果需要添加新的产品，需要修改 `SimpleFactory`，不符合**“对扩展开放，对修改封闭”**的原则。
- 不能很好地支持继承和扩展。

---

### **(2) 工厂方法模式（Factory Method Pattern）**
**特点**：
- 每个具体产品类都有自己的工厂类，符合**开闭原则（OCP）**。
- 客户端调用具体工厂来创建实例，而不是直接使用 `new`。

**示例**：
```cpp
#include <iostream>
#include <memory>

// 抽象产品
class Product {
public:
    virtual void show() = 0;
    virtual ~Product() {}
};

// 具体产品 A
class ProductA : public Product {
public:
    void show() override { std::cout << "Product A\n"; }
};

// 具体产品 B
class ProductB : public Product {
public:
    void show() override { std::cout << "Product B\n"; }
};

// 抽象工厂
class Factory {
public:
    virtual Product* createProduct() = 0;
    virtual ~Factory() {}
};

// 具体工厂 A
class FactoryA : public Factory {
public:
    Product* createProduct() override { return new ProductA(); }
};

// 具体工厂 B
class FactoryB : public Factory {
public:
    Product* createProduct() override { return new ProductB(); }
};

// 测试
int main() {
    std::unique_ptr<Factory> factoryA = std::make_unique<FactoryA>();
    std::unique_ptr<Product> productA(factoryA->createProduct());
    productA->show();

    std::unique_ptr<Factory> factoryB = std::make_unique<FactoryB>();
    std::unique_ptr<Product> productB(factoryB->createProduct());
    productB->show();

    return 0;
}
```
### **优缺点**
✅ **优点**：
- **符合开闭原则**：新增产品时，只需增加新的**工厂类**，不修改现有代码。
- 代码更加**灵活可扩展**，不同工厂可以定制创建逻辑。

❌ **缺点**：
- 需要为每种产品创建一个**工厂类**，可能导致类数量增多。

---

### **(3) 抽象工厂模式（Abstract Factory Pattern）**
**特点**：
- 适用于**创建多个相关产品**的情况（如 GUI 库需要同时创建按钮和文本框）。
- 工厂不仅能创建一种产品，而是可以创建一**系列**的产品。

**示例**：
```cpp
#include <iostream>

// 产品抽象基类
class Button {
public:
    virtual void click() = 0;
    virtual ~Button() {}
};

class TextBox {
public:
    virtual void type() = 0;
    virtual ~TextBox() {}
};

// 具体产品：Windows 风格按钮和文本框
class WindowsButton : public Button {
public:
    void click() override { std::cout << "Windows Button Clicked\n"; }
};

class WindowsTextBox : public TextBox {
public:
    void type() override { std::cout << "Windows TextBox Typing\n"; }
};

// 具体产品：Mac 风格按钮和文本框
class MacButton : public Button {
public:
    void click() override { std::cout << "Mac Button Clicked\n"; }
};

class MacTextBox : public TextBox {
public:
    void type() override { std::cout << "Mac TextBox Typing\n"; }
};

// 抽象工厂
class GUIFactory {
public:
    virtual Button* createButton() = 0;
    virtual TextBox* createTextBox() = 0;
    virtual ~GUIFactory() {}
};

// Windows 具体工厂
class WindowsFactory : public GUIFactory {
public:
    Button* createButton() override { return new WindowsButton(); }
    TextBox* createTextBox() override { return new WindowsTextBox(); }
};

// Mac 具体工厂
class MacFactory : public GUIFactory {
public:
    Button* createButton() override { return new MacButton(); }
    TextBox* createTextBox() override { return new MacTextBox(); }
};

// 测试
int main() {
    std::unique_ptr<GUIFactory> factory = std::make_unique<WindowsFactory>();
    std::unique_ptr<Button> button(factory->createButton());
    std::unique_ptr<TextBox> textBox(factory->createTextBox());

    button->click();
    textBox->type();

    return 0;
}
```
### **优缺点**
✅ **优点**：
- 适用于创建**多个相关产品**的情况。
- 代码结构**清晰，便于扩展**。

❌ **缺点**：
- 代码复杂度较高，**适用于复杂项目**。

---

## **4. 总结**
| 模式 | 适用场景 | 是否符合开闭原则 | 代码复杂度 |
|------|---------|-------------|---------|
| **简单工厂模式** | 产品种类较少 | ❌ 不符合 | ⭐ |
| **工厂方法模式** | 产品种类较多 | ✅ 符合 | ⭐⭐ |
| **抽象工厂模式** | 多种相关产品 | ✅ 符合 | ⭐⭐⭐ |

👉 **推荐**：
- **简单工厂**：适用于小型项目，代码简洁。
- **工厂方法**：适用于需要扩展的产品类型。
- **抽象工厂**：适用于创建多个**相关产品**的情况，如 UI 组件或数据库连接工厂。