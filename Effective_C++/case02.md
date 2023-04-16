# 尽量以const,enum,inline代替#define
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
## 请记住
- 对于单纯常量，最好以const或enum代替#define
- 对于宏函数，最好改用inline代替#define，最终达到以编译器代替预处理器