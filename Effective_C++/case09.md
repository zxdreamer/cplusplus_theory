# 绝不在构造和析构函数中调用virtual
本条款前我先阐述观点，不该在构造函数和析构函数中调用virtual函数，因为这样调用不会带来你想要的结果。
## 股票交易系统的日志例子
假如股票交易系统中，存在一个记录日志的类，用于记录交易订单。
```c++
class Transaction {
public:
    Transaction() {
        ...
        logTransaction();
    }
    virtual void logTransaction() const = 0;
    ...
};
```
买卖股票中的操作类分别是BuyTransaction和SellTransaction类
```c++
class BuyTransaction: public Transaction {
public:
    virtual void logTransaction() const;
    ...
};
class SellTransaction: public Transaction {
public:
    virtual void logTransaction() const;
    ...
};
```
当执行以下程序会发生什么事情呢？
```c++
BuyTransaction b;
```
构造b对象时会先执行base-class的构造函数Transaction()，当执行到最后一行logTransaction()时，调用的是Transaction版本，并没有调用BuyTransaction版本，因为base-class构造期间，virtual函数不会下沉到derived-class中。
- 原因
  1. 简单原因，base-class的执行早于derived-class的构造，当base-class构造函数执行时，derived-class的成员变量尚未初始化，此时若将virtual函数下降到derived-class,必然会调用成员变量，会出现很多未定义的行为。
  2. 根本原因，derived-class的base-class期间，对象的类型是base-class而非derived-class。virtual函数会被编译器解析成base-class，使用运行类型信息如dynamic_cast或typeid也会把对象视为base-class。因为derived-class为初始化完全之前，最安全的做法是任务其不存在。
- 析构函数也类似，由于继承体系中，析构函数与构造函数执行顺序相反，会先执行derived-class的析构函数，再执行base-class的析构函数，所以在base-class的析构函数执行时，derived-class的专属对象已经被销毁，所以也不能将virtual下降到子类中。
## 构造函数调用其他函数间接引起virtual函数被调用
```c++
class Transaction {
public:
    Transaction() {
        init();
    }
    virtual void logTransaction() const = 0;
    ...
private:
    void init() {
        ...
        logTransaction();
    }
};
```
这段代码与稍早版本相同，但比较隐藏却暗中为害，因为virtual函数logTransaction()封装到init()中，间接被构造函数调用了，我们排查问题时一定要注意这种间接调用的情况发生。
## 如何确保每一次Transaction继承体系中的对象被创建，就会有适当版本的logTransaction()被调用？
一种可行的方案是，将logTransaction改成non-virtual，然后要求derived-class构造函数传递必要的信息。
```c++
class Transaction {
public:
    explicit Transaction(const string& logInfo) {
        logTransaction(logInfo);
    }
    void logTransaction(const string& logInfo) const;
};
```
```c++
class BuyTransaction: public Transaction {
public:
    BuyTransaction(param) : Transaction(param) {};
    ...
};
```
## 请记住
- 构造函数和析构函数中不要调用virtual函数，因为这类调用不会下降至derived-class中

