# 绝不在构造函数或析构函数中调用virtual函数
- 在base class构造期间，virtual不是virtual函数
```c++
class Transaction {
public:
    Transaction();
    virtual void logTransaction() const = 0; // 做出一份因类型不同而不同的日志
    ...
};
Transaction::Transaction() {
    ...
    logTransaction();                       // 最后的动作是记录日志
}
class BuyTransaction : public Transaction {
public:
    virtual void logTransaction() const;
    ...
};
class SellTransaction : public Transaction {
public:
    virtual void logTransaction() const;
    ...
};
// 此时如下调用会发生什么事情:
BuyTransaction b;
// 此时会先调用base-class的构造函数，再调用derived-class的构造函数，此时base-class的logTransaction()不会下沉到子类中。
// 因为，在base-class构造函数执行期间，derived-class的成员变量尚未完成初始化，此时将virtual下沉到子类，调用derived-class的成员变量存在未定义的错误。
```
- 在析构函数中，virtual不是virtual函数
在析构过程中，先调用derived-class的析构函数，此时完成自身对象的释放，在base-class中调用virtual函数同样存在未定义的对象，也不会下沉到derived-class中。
- base-class中想要用derived-class的内容怎么实现？
```c++
class Transaction {
public:
    explicit Transaction(const std::string& logInfo);
    void logTransaction(const std::string& logInfo) const; // 做出一份因类型不同而不同的日志
    ...
};
Transaction::Transaction(const std::string& logInfo) {
    ...
    logTransaction(logInfo);                              // 最后的动作是记录日志
}
class BuyTransaction : public Transaction {
public:
    BuyTransaction(parameters):Transaction(createLogString(parameters))
    ...
private:
    static std::string createLogString(params);
};
// 在base-class中去掉virtual函数，在构造期间，改用向base-class传参的形式实现base-class使用derived-class的效果，从而在base-class中统一实现某些功能
```
## 请记住
- 在构造函数和析构函数中不要调用virtual函数，这类调用不会下沉至derived-class中