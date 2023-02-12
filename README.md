# 写出高质量C++程序
工作后越来越胖，就像我们的C++，经历版本迭代，功能开始愈发浮肿，使我忍不住思考，如何在工作中写出高质量的C++代码，又不引入晦涩的语法呢？恰看到侯捷先生出版的系列书籍，深受启发，决定整理出几则精简且实用的C++指南。
## 尽量以const,enum,inline代替#define
```c++
#define ASPECT_RATIO 1.653;
#define CALL_WITH_MAX(a, b) f((a) > (b) ? (a) : (b))
```
记号名ASPECT_RATIO在预编译阶段被替换成1.653，会造成四点问题：
- 未被编译器看到，未进入程序的符号表内，问题定位困难
- 目标代码会出现多份1.653，容易造成代码膨胀
- 不能为class创建专属宏定义
- 宏函数展开形式与参数关联严重，容易造成未知错误
```c++
const double AspectRatio 1.653;
```
```c++
class GamePlayer{
private:
    //1. const int NumTurns = 5;    // 为常量指定作用域
    static const int NumTurns;      // 常量声明(不可以取地址)，保证常量只有一份实体

    int scores[NumTurns];
}
const int GamePlayer::NumTurns = 5; // 常量定义
```
```c++
class GamePlayer{
private:
    enum {NumTurns = 5};        // enum hack，由于有些编译器不支持static类内常量
                                // 两者都不可以取地址，适用于某变量不想让pointer或reference引用；
    int scores[NumTurns];       // 不会带来额外的内存开销
}
```
宏函数可以避免函数调用带来的开销，但是需要记住为所有参数加上小括号，但还是会出现不可思议的问题：
```c++
int a = 5, b = 6;
CALL_WITH_MAX(++a, b);      
CALL_WITH_MAX(++a, b+10);   // a的递增次数，取决于和谁比较
```
幸运的是我们可以避免为这种事提供温床，可以获得宏带来的效率以及一般函数的所有可预测行为(private,类型检查)——template inline函数。
```c++
template<typename T>
inline void callWithMax(const T& a, constT& b) {
    if(a > b ? a : b);
}
```
### 请记住
- 对于单纯常量，最好以const或enum代替#define
- 对于宏函数，最好改用inline代替#define，最终达到以编译器代替预处理器
## 尽可能实用const
关键字const多才多艺。
- 定义常量
- const最权威的用法是面对函数声明时的作用，在一个函数声明式内，const可以修饰返回值，参数，函数自身。
```c++ 
// 修饰返回值
class Rational {...}
const Rational operator*(const Rational& lhs, const Rational& rhs); // 避免(a * b) = c;
                                                                    // 即可以避免对函数的错误赋值或误判
```
```c++
// 修饰参数
void f1(Widget* p);         // 一般认为param-by-pointer, 用于改变参数传出函数外
void f1(const Widget* p);   
void f1(const Widget& r);   // param-by-reference to const 用于传值,建议大家函数不改变参数值时，尽量加上const，因为他是提升C++运行效率很重要的方式之一
```
```c++
// 修饰成员函数
// 1. 增强class接口的可读性
// 得知哪个函数可以改变对象内容，哪些不行很重要

// 2. 操作const对象成为可能
// param-by-reference to const 传递对象时，必须知道可以调用哪些函数
// const对象只能调用const修饰的成员函数
// non-const对象可以调用任何publish成员函数

// 3. const支持函数重载
class TextBlock {
public:
    const char& operator[](std::size_t pos) const{
        return text[pos];
    }
    char& operator[](std::size_t pos) {
        return text[pos];
    }
private:
    std::string text;
};

void print(const TextBlock& ctb) {  
    std::cout << ctb[0]; // 调用const TextBlock::operator[]
}

TextBlock tb("Hello");
TextBlock ctb("World");
std::cout << tb[0];
tb[0] = 'c';
std::cout << ctb[0];
ctb[0] = 'c';          // 编译错误

// 4. const 和 non-const中避免重复代码
// TextBlock类的成员函数operator[]，中会有参数边界检查，标记位置位，校验数据完整性等，const和non-const中都写一遍存在代码重复，解决办法是non-const调用const成员函数
class TextBlock {
public:
    const char& operator[](std::size_t pos) const{
        // ...
        return text[pos];
    }
    char& operator[](std::size_t pos) {
        return (const_cast<char&>)(static_cast<const TextBlock&>(*this));
    }
private:
    std::string text;
};

// 5. const成员函数中可以修改成员变量
// const成员函数中确实可以修改成员变量，但是需要对变量声明为mutable
```
### 请记住
- const可施加于任何作用域内的任何对象(参数，返回值，函数本体，局部或全局变量)
- 提升C++性能的重要做法之一：param-by-reference to const
- const和non-const有实质等价的实现，可以用non-const调用const版本，减少重复
## 确定对象被使用前已被初始化
- 一件很奇怪的事，c++似乎并不清楚，什么时候要帮我们初始化变量。
```c++
int x;              // x不被初始化
int nums[5];        // nums不被初始化
vector<int> vecs(5);   // vecs被初始化为0
// c part of C++ 不会被自动初始化, non-c part of c++(stl, object-oriented c++，template)会被初始化为空。
```
- 初始化落在了构造函数上，确保每个构造函数都讲对象的每一个成员初始化
- 不要混淆初始化和赋值
  - 赋值
    ```c++
    class PhoneNumber {...}
    class ABEntry {
    public:
        ABEntry(cosnt string& name, const string& address, const list<PhoneNumber>& phones){
            theName = name;
            theAddress = address;
            thePhones = phones;
            numTimesConsulted = 0;
        }   
    private:
        string theName;
        string theAddress;
        list<PhoneNumber> thePhones;
        int numTimesConsulted;
    };
    ```
  - 初始化列表
    ```c++
    class PhoneNumber {...}
    class ABEntry {
    public:
        ABEntry(cosnt string& name, const string& address, const list<PhoneNumber>& phones):theName(name),theAddress(address),thePhones(phones),numTimesConsulted(0){ } 
    private:
        string theName;
        string theAddress;
        list<PhoneNumber> thePhones;
        int numTimesConsulted;
    };
    ```
    - 赋值的过程：默认构造函数->拷贝构造函数
    - 初始化列表：拷贝构造函数
    - 初始化列表更高效，调用时间更早
    - 对于内置类型赋值和初始化开销相同，但为了保证一致性，内置类型也用初始化列表
    - 有些内置类型必须初始化，static，const，reference
- 跨编译单元的初始化