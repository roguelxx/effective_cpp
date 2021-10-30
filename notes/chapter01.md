- A declaration tells compilers about the name and type of something, but it omits certain details
- A definition provides compilers with the details a declaration omits. For an object, the definition is where compilers set aside memory for the object. For a function or a function template, the definition provides the code body. For a class or a class template, the definition lists the members of the class or template
- Initialization is the process of giving an object its first value. For objects of user-defined types, initialization is performed by constructors. A default constructor is one that can be called without any arguments. Such a constructor either has no parameters or has a default value for every parameter

- ```c++
  const std::vector<int>::iterator iter = // iter acts like a T* const
  	vec.begin(); // OK, changes what iter points to
  *iter = 10;
  ++iter; // error! iter is const
  
  std::vector<int>::const_iterator cIter = //cIter acts like a const T*
  	vec.begin(); // error! *cIter is const
  *cIter = 10;
  ++cIter; // fine, changes cIter
  ```

- 

## Item 3

当一个函数的`const`版和`non-const`版做的事情完全相同时，可以通过cast，在`non-const`版内调用`const`版本。`non-const`版内部需要进行两次cast：

- 将`*this`通过`static_cast`转化为`const`，从而可以调用`const`版函数
- 将`const`版函数的返回值通过`const_cast`转化为`non-const`

## Item 4

对于内置类型，尽可能的手动赋予初始值；对于其他的类型，初始化的任务由ctor完成，要保证ctor内初始化了这个对象的所有成员。

对于对象的ctor，c++规定“对象成员的初始化发生在ctor函数体执行**之前**”，所以，下面这种初始化方法并不是最好的：

```c++
class ABEntry {
 // ABEntry = "Address Book Entry"
public:
	ABEntry(const std::string& name, const std::string& address,
	const std::list<PhoneNumber>& phones);
private:
	std::string theName;
	std::string theAddress;
	std::list<PhoneNumber> thePhones;
	int num TimesConsulted;
};

ABEntry::ABEntry(const std::string& name, const std::string& address,
	const std::list<PhoneNumber>& phones)
{
	theName = name; 		 // these are all assignments,
	theAddress = address;    // not initializations
	thePhones = phones
	numTimesConsulted = 0;
}
```

`ABEntry` ctor内部进行的时赋值（assignment）操作，在函数体执行前，非内置类型（`string`，`list`）调用对应的默认构造器初始化，而对于内置类型（`int`）就无法保证它的初始化。所以，正确的写法应该是使用初始化列表（initialization list）,通过赋值构造（copy-construct）初始化。

```c++
ABEntry::ABEntry(const std::string& name, const std::string& address,
	const std::list<PhoneNumber>& phones) 
: theName(name),theAddress(address), // these are now all initializations, using copy-construct
  thePhones(phones),
  numTimesConsulted(0)
{} // the ctor body is now empty
```

静态对象可以分为两类：定义在函数内部的叫local static对象，除此之外的叫non-local static对象（全局对象，class内部的static成员，文件内的static变量等）。对于在不同编译单元（源文件）中的non-local静态对象，它们的初始化顺序是不确定的：

```c++
class FileSystem { // from your library
public:
	...
	std::size_t numDisks() const; // one of many member functions
	...
};

extern FileSystem tfs; // global object for clients to use;
					   // "tfs" = "the file system"
/*----------------------------------------------------------------*/
class Directory { // created by library client
public:
	Directory( params );
	...
};
Directory::Directory( params )
{
	...
	std::size_t disks = tfs.numDisks(); // use the tfs object
	...
}
Directory tempDir( params ); // directory for temporary files
```

显然，我们在使用`tempDir`时，必须保证`tfs`已经被初始化了，但是non-local静态对象的初始化顺序是不确定的。所以，我们必须改变代码写法：

```c++
FileSystem& tfs()				// this replaces the tfs object; it could be
{								// static in the FileSystem class
	static FileSystem fs;	    // define and initialize a local static object
	return fs;					// return a reference to it
}

Directory& tempDir()			// this replaces the tempDir object; it
{								// could be static in the Directory class
	static Directory td;		// define/initialize local static object
	return td;					// return reference to it
}
```

我们通过一个返回对象引用的函数，将non-local静态对象转化为了local静态对象，使用者通过调用该函数获取对象。因为c++保证：调用对象成员函数时，对象一定已经被初始化了，即我们拿到的一定是已初始化的对象。这种写法也叫做单例模式（singleton）。单例模式还有一个好处，如果我们没调用函数（`tfs()`/`tempDir()`），那么对应的对象就不会被初始化，就不会有构造对象、析构对象的开销。

要注意的是，`non-const`静态对象不是线程安全的，上述写法仅适用于单线程环境。

![image-20211028142911794](images\item4-conclusion.png)