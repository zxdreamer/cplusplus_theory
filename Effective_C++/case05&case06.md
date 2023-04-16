# 若不想使用编译器自动生成的函数，就该明确拒绝
## 了解C++默默编写哪些函数
```c++
class Empty{ };
// 这就好像你写了如下代码
class Empty {
public:
    Empty()
    Empty(const Empty& empty)
    ~Empty()    // 思考，这里为什么不是virtual?

    Empty& operator=(const Empty& empty)
};
// 这些自动生成的代码只有在调用时才被生成出来
```
## 如何禁止编译器默认生成的函数
- delete 关键字，适用于c++11以上版本
```c++
class HomeForSale {
public:
    HomeForSale(const HomeForSale&) = delete;
    HomeForSale& operator=(const HomeFoeSale&) = delete;
    // default是与delete对应的关键字，代表明确使用编译器生成的函数
};
```
- 将成员函数声明为private而且故意不去实现
```c++
class HomeForSale {
public:
    ...
private:
    HomeForSale(const HomeForSale&);
    HomeForSale& operator=(const HomeFoeSale&);
};
// 外部函数调用报private的编译错误，成员函数或friend函数调用报链接错误
// 错误应该被更早的侦测出来，链接错误能否移到编译中呢？
```
- 继承方式Uncopyable类
```c++
class Uncopyable {
protected:
    Uncopyable() {}
    ~Uncopyable() {}
private:
    Uncopyable(const Uncopyable&);
    Uncopyable& operator=(const Uncopyable&);
};
// 其他类直接继承就好，由于拷贝构造函数要先拷贝父类对象，父类的copy构造函数是私有成员，所以编译器不会默认生成。
class HomeForSale : private Uncopyable {

};
// Uncopyable的实现和运用颇为微妙，请大家仔细体会。
```
## 请记住
- 如果不声明，C++编译器暗自生成构造函数，拷贝构造函数，拷贝运算符以及析构函数，但是如果生成的代码存在错误，则编译器选择不生成。
- 为驳回编译器暗自提供的机能，选择将成员函数声明为private并不去实现，使用Uncopyable也是不错的选择。