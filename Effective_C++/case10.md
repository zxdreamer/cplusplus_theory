# 令operator= 返回一个reference to *this
关于赋值，有趣的是可以写成连锁形式：
```c++
int x, y, z;
x = y = z = 5;
```
同样有趣的是，赋值采用右结合律，上述连锁赋值被解析成：
```c++
x = (y = (z = 15));
```
为了实现"连锁赋值"，赋值操作符必须返回一个reference指向左侧实参，这是为class实现赋值操作符应遵循的协议:
```c++
class Widgt {
public:
    Widgt& operator=(const Widgt& rhs) {
        ...
        return *this;
    }
};
```
此协议不仅适用于上述标准赋值，也适用于所有赋值相关的运算，例如：
```c++
class Widgt {
public:
    Widget& operator+=(const Widget& rhs) {
        ...
        return *this;
    }
    Widget& operator-=(const Widget& rhs) {
        ...
        return *this;
    } 
    ...   
};
```
注意这只是一个协议，并无强制性。如果不遵循它，代码一样可以通过编译，然而这个协议，所有内置类型和标准库提供的STL类型都共同遵守，因此除非你有个标新立异的好理由，否则还是随众吧!
## 请记住
- 令赋值操作符返回reference to *this