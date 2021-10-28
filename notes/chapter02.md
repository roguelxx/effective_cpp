## Item 6

不允许复制class的操作被编译（copy construct 、assignment），将class的copy ctor、assginment申明为private，同时不实现这些函数。STL中的iostream相关类就是这么做的。

![image-20211028154902787](D:\projects\CLionProjects\effective_cpp\notes\images\item6-conclusion.png)

## Item 7