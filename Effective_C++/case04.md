# 确定对象被使用前已被初始化
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
## 请记住
- 为内置类型手动初始化，C++不保证初始化他们
- 构造函数最好使用初始化列表，初始化列表列出的成员变量名，其排列顺序应该和在class中声明次