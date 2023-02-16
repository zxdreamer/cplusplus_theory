# 令operator=返回一个reference to *this
- 关于赋值，可以写成连锁形式
```c++
int x, y, z;
x = y = z = 15;
// 赋值采用右结合律, 以上赋值结果被解析为 
x = (y = (z = 15)); 
```
- 实现"连锁赋值"，operator=()需要返回一个reference to *this
```c++
class Widget {
public:
    ...
    Widget& operator=(const Widget& rhs) {
        ...
        return *this;
    }
};
```
- 这个协议不仅适合于标准赋值形式，也适用于所有赋值相关运算符
```c++
class Widget {
public:
    ...
    Widget& operator+=(const Widget& rhs) {
        ...
        return *this;
    }
};
```
### 注意: 这只是一个协议，并无强制性，然而这份协议被所有stl容器类遵从，如果没有一个标新立异的好理由，就请随众吧!
## 令赋值操作符返回一个reference to *this
