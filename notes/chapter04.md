## Item 19

函数参数和函数返回值默认以值的形式传递（copy-by-value），这些行为由类型的copy-ctor控制。通过显式的将参数和返回值申明为类型的引用，可以避免ctor和dtor的开销；同时，为了让函数调用者（function caller）不用担心函数内部会改变实参，将形参声明为const。

另一个问题是，如果形参是base class，实参是derived class，通过value传递的话，只有父类的ctor会被调用，传进去的obj只带有父类的信息。将形参声明为`const &`也可以避免这个问题。

但是对于内置类型（e.g. int）、迭代器、函数，这些类型被设计成可以被高效copy，所以可以copy by value。

## Item 20

一个function有两个创建新对象的方法：在stack上或在heap上。在stack上创建的对象也被称为local object，当函数运行完成时就被销毁了。

所以返回指向局部对象（在stack上创建）的指针（引用）的函数都是坏东西，因为caller得到的是一个dangle pointer或undefined reference。

如果在函数内new一个新对象（在heap上创建）然后返回这个对象的引用或指针，则可能会造成内存泄漏问题。

所以最好的做法就是返回新对象的copy。

![image-20211113201800894](/home/lxx/Documents/books/cpp/effective_cpp/notes/images/item20.png)