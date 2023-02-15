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
    - 初始化列表较长，???
- 跨编译单元的初始化
  - 编译单元是指，单一源文件以及包含的所有头文件。C++对跨编译单元的变量（non-local static变量）的初始化顺序不固定。
  ```c++
    // FileSystem.cpp
    class FileSystem {
    public:
        ...
        std::size_t numDisks() const;
        ...
    };
    extern FileSystem tfs;
    // Directory
    class Directory {
    public:
        Directory(params) {
            auto disks = tfs.numDisks(); // 可能存在问题，某个编译单元的non-local static变量的初始化用到了另一个编译单元的non-local static变量，他所用到的变量可能未被初始化
        }
    }
  ```
  - 解决办法是，将non-local static搬到函数内部，转化成local static对象，并返回其引用。原因是，C++保证local static对象会在该函数调用期间，首次遇上对象定义时初始化。
  ```c++
    class FileSystem { };
    FileSystem& tfs() { // Disign Patterns的迷哥迷妹们，看起来会像Singleton模式
        static FileSystem fs;
        return fs;
    }
    class Directory {
    public:
        Directory(params) {
            auto disks = tfs().numDisks();
        }
    };    
  ```
### 请记住
- 为内置类型手动初始化，C++不保证初始化他们
- 构造函数最好使用初始化列表，初始化列表列出的成员变量名，其排列顺序应该和在class中声明次序相同
- 免除夸编译单元的初始化次序问题，请以local static代替non-local static对象
## 若不想使用编译器自动生成的函数，就该明确拒绝
### 了解C++默默编写哪些函数
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
### 如何禁止编译器默认生成的函数
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
### 请记住
- 如果不声明，C++编译器暗自生成构造函数，拷贝构造函数，拷贝运算符以及析构函数，但是如果生成的代码存在错误，则编译器选择不生成。
- 为驳回编译器暗自提供的机能，选择将成员函数声明为private并不去实现，使用Uncopyable也是不错的选择。
## 为多态基类声明为virtual析构函数