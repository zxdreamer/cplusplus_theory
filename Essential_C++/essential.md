1. binary function adapter 转化成 unary function object 使用 binder adapter ：bind1st 和 bind2st.
2. 两种赋值方式：先申请内存再填充内存空间；只申请空容器，以扩展方式插入新元素，称为iterator insert，有三个函数：back_inserter，inserter，front_inserter。
3. 文件的输入输出流可以转化成迭代器：istream_iterator和ostream_iterator。
4. ：： 是作用域运算符，放到类名后。
5. member object的初始化必须放到成员初始化列表中。
6. 拷贝构造函数需要成员变量依次复制。
7. const函数与非const函数调用规则。const对象不可直接调用非const成员函数，需要const_cast转化，非const对象不可直接调用const方法，需要强制转化。const支持重载。mutable。
8. this指针，编译器会自动将this指针加入到每个member function，指向调用对象。
9. static const可作为预编译，亦可作为静态常量，但都仅限常量。
10. static member function 用来表示唯一的，可共享的member，节约内存空间。类主题外进行定义时，无需加上static。
11. 前置自增：type& operator++（）
    后置自增：type operator++（int）
12. 不可重载的运算符：.，.*，：：，？：
13. 嵌套类型：用typedef在类内部进行类型别名的声明，类外部就可以隐藏原始类，改用别名类。
14. 友元函数可以访问类的private成员，要在类内部声明，friend class：：function（）。
     友元类声明：friend class class-name。
15. 仿函数就是类中重载了（）运算符，需要定义对象，使用对象名（）来调用。编译器进行转化：obj.operator（）。
16. 重载iostream，即《和》。不能放到类内部，而是将类作为ostream& operator《（ostream，const class& hs）来传递，并且返回ostream，方便级联。
17. pod 是没有使用面向对象思想设计的类和结构体，支持静态初始化，支持c语言的内存布局。
18. final修饰类，代表类禁止继承；修饰函数，代表此虚函数禁止重载。
19. const是运行时常量，constexpr是编译时常量
20. static member function 无法被声明为virtual。
21. 纯虚函数没有函数定义，是不完整的，所以不能定义对象。
22. 基类中有virtual函数，析构函数应该定义成虚函数。
23. 子类的构造函数不仅为子类的data提供初始化操作，还需要为父类的data提供初始化操作。
23. 默认拷贝构造是类中的data被逐一复制初始化，包括父类的data。
24. copy assignment operator 拷贝父类对象时需要调用父类的运算符。p160。
25. typeid（obj）查询多态的信息，返回type_info，是obj具体对象。
26. 父类指针或引用没有用new申请内存，直接指向父类地址，此时不可以直接调用子类函数，需要用dynamic_cast转化。