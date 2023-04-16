# 别让异常逃离析构函数
C++的析构函数有意或无意会造成抛出异常的事，此时程序会过早的结束，由于析构函数未执行完，会造成内存泄漏。所以C++不喜欢析构函数吐出异常。
## 如果一个函数必须执行一个动作，这个动作可能抛出异常，该如何解决？
```c++
class DBConnection {
public:
    ...
    static DBConnection create();

    void close();   // 关闭数据库，失败有可能抛出异常
}；
```
为保证用户不忘记调用close，我们创建一个管理DBConnection的class,并在其析构函数中调用close.
```c++
class DBConn {
public:
    ...
    ~DBConn() {
        db.close();
    }
private:
    DBConnection db;
};
```
客户代码
```c++
{
    DBConn dbc(DBConnection::create()); // 离开作用域，会自动调用close()
}
```
如果调用close成功一切都好，但如果调用导致异常，析构函数会传播异常，会造成不可意料的麻烦。
- 终止程序，std::abort()
```c++
DBConn::~DBConn() {
    try {db.close()}
    catch (...) {
        // log中记录异常信息
        std::abort();
    }
}
```
析构函数中出现异常就终止程序是一个合理的行为，毕竟终止程序比出现未意料的行为要好。
- 吞下调用close的异常
```c++
DBConn::~DBConn() {
    try {db.close()}
    catch (...) {
        // 制作运行记录，记录下对close的调用失败
    }
}
```
虽然由析构函数吞下异常是个坏主意，他压制了"某些动作失败"的重要信息，但也提供了一种不想草率的结束程序的一种方案。
- 将可能出现异常的操作交给用户调用
```c++
class DBConn {
public:
    ...
    void close() {
        db.close();
        closed = true;
    }
    ~DBConn() {
        if (!closed) {
            try {
                db.close();
            }
            catch {...} {
                //制作运行记录，记录close调用失败；
            }
        }
    }
private:
    DBConnection db;
    bool closed = false;
};
```
我们应该赋予客户一个权力，有机会处理函数调用的异常，而不是我们暗地里将异常处理完，用户会对抛出的异常感到困惑。
## 请记住
- 析构函数绝不要吐出异常，如果一个函数调用的函数可能抛出异常，析构函数应该吞下异常或终止程序
- 对于可能抛出异常的函数，应该一共一个普通函数让用户对异常做出回应，而非在析构函数中执行改操作
