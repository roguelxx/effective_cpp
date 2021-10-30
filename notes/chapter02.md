## Item 5

在没有显示申明ctor、dtor、copy ctor、copy assignment时，如果代码中用到了这些函数，编译器会自动帮我们生成这些函数的默认版本。

如果class中有引用成员变量，那么编译器不会为我们生成copy assignment，我们需要手动写赋值函数。

## Item 6

不允许复制class的操作被编译（copy construct 、assignment），将class的copy ctor、assginment申明为private，同时不实现这些函数。STL中的iostream相关类就是这么做的。

![image-20211028154902787](images\item6-conclusion.png)

## Item 7

c++中删除父类指针指向的子类对象时，如果父类的析构函数不是虚函数（non-virtual），那么子类的析构函数就不会被调用，造成内存泄漏：

```c++
Base *pson = son; // class Base is father, class Son is derived class
delete pson;
```

将父类的析构函数标识为虚函数，可以避免这个问题。

如果class中不存在虚函数，一般而言不会将其析构函数申明为虚函数，也不会将这个class作为基类（base class）。因为如果class中存在虚函数，compiler会被这个class生成虚函数表（vtbl），以及指向虚函数表的虚指针（vptr），存储vtbl和vptr都需要占用额外的内存空间。所以当且仅当class中有虚函数时，才考虑是否将class的析构函数标识为virtual。

抽象类（abstract class）的析构函数如果是纯虚函数（`virtual ~dtor() = 0`），那么需要在类外定义（define）析构函数；不然继承类无法通过编译器的链接阶段。

当需要通过父类提供的接口来控制子类时（多态），需要将析构函数申明为虚函数，防止内存泄漏。

![image-20211030110110396](images/item7-conclusion.png)