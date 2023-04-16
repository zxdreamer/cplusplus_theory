# 为多态基类声明为virtual析构函数
```c++
class TimeKeeper {
    TimeKeeper();
    ~TimeKeeper();
    ...
};
class AtomicClock: public TimeKeeper {}
class WaterClock: public TimeKeeper {}
class WristWatch: public TimeKeeprt {}
TimeKeeper* getTimeKeeper(); // factory 函数，返回一个具体的时钟

// 用户
TimeKeeper* ptk = getTimeKeeper();
// ...
delete ptk;
// 由于base-class的析构函数是non-virtual，所以此时只能释放derived-class的内存，base-class的内存泄漏
```
## 是不是需要为所有base-class的析构函数都加上virtual？
```c++
class Point {
public:
    Point(int x, int y) = default;
    ~Point() = default;
private:
    int x, y;
};
// 这样内存紧凑的类，占用8个字节，运行期间可以直接装入寄存器或缓存中，加入虚函数会在类中增加虚函数指针vptr，占用4-8字节，内存翻倍，同时会引入一系列维护操作，增加运行速度的缓存操作更无从谈起。可见，无端的为类的析构函数声明为virtual，就像没用virtual一样有害。
```
## stl的一个小插曲
```c++
class SpecialString : public std::string {  // 馊主意！std::string的析构函数是non-virtual

};
string* ps = new SpecialString("Hello world!");
delete ps;  // 内存泄漏
// 绝大部分stl的容器的析构函数都是non-virtual，不要去继承一个stl容器
```
## 带多态性的base-class的析构函数声明为virtual
### 当类中至少有一个函数是虚函数，才为他声明virtual析构函数
## 请记住
- 带有多态性质的类应该为他的析构函数声明为virtual，class的设计目的不作为base class使用，就不要为他声明虚析构函数。