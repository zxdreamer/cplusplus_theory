# 尽可能使用const
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
void f1(const Widget* p);   // 可读性上有很大歧义
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
## 请记住
- const可施加于任何作用域内的任何对象(参数，返回值，函数本体，局部或全局变量)
- 提升C++性能的重要做法之一：param-by-reference to const
- const和non-const有实质等价的实现，可以用non-const调用const版本，减少重复