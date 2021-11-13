需要编程人员自己管理的除了从堆上申请到的动态内存，还有文件描述符（fd），锁（mutex）等。我们通常会设计一个资源管理类来控制资源的生命周期，一种设计原则叫做RAII（Resource Acquisition Is Initialization），意思是：当资源被获取的时候，就初始化资源管理类来控制资源，当资源管理类被销毁时，其控制的资源也响应的被释放。

## Item 14

简单的用来控制锁的获取和释放的类：

```c++
class Lock {
public:
	explicit Lock(Mutex *pm) : mutexPtr(pm) { lock(mutexPtr); } // acquire resource
	~Lock() { unlock(mutexPtr); } // release resource
private:
	Mutex *mutexPtr;
};
```

但是资源管理类的copy行为需要如何实现呢，一般有以下几个方面可以考虑：

- 禁止拷贝：mutex
- 对资源计数（reference count）：允许资源被多个object共享
- 拷贝资源：允许存在资源，同时保证每个资源都被释放
- 转移资源的使用权

## Item 17

编译器对于函数参数的初始化可能不是按顺序执行的，所以当参数初始化中涉及动态内存空间的分配时，可能会造成内存泄漏。正确的做法是将这种类型的初始化分到独立的语句中，再将分配好空间的变量传入函数。