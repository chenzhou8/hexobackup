---
title: C++内存管理
date: 2018-11-14 11:02:09
toc: true
categories: C/C++
---

## 内存管理的形式

* 栈： 栈又叫堆栈，非静态局部变量/函数参数/返回值等等，栈是向下增长的,当方法和语句块一结束，空间马上释放
* 内存映射段：是高效的I/O映射方式，用于装载一个共享的动态内存库。用户可使用系统接口创建共享共享内存，做进程间通信
* 堆：堆用于程序运行时动态内存分配，堆是可以上增长的，存放的是成员变量，随着对象而产生，随对象销毁而销毁
* 数据段：存储全局数据和静态数据
* 代码段：可执行的代码/只读常量



## malloc/calloc/realloc函数

```cpp
#include <stdlib.h>

void *malloc(size_t size);
void free(void *ptr);
void *calloc(size_t nmemb, size_t size);
void *realloc(void *ptr, size_t size);
```
The  malloc()  function allocates size bytes and returns a pointer to the allocated memory.  The memory is not initialized.
If size is 0, then malloc() returns either NULL, or a unique pointer value that can later be successfully passed to free().

The free() function frees the memory space pointed to by ptr, which must have been returned by a previous call to malloc(),calloc()  or  realloc().   Otherwise, or if free(ptr) has already been called before, undefined behavior occurs.  If ptr is NULL, no operation is performed.

The calloc() function allocates memory for an array of nmemb elements of size bytes each and returns a pointer to the allocated  memory.   The  memory is set to zero.  If nmemb or size is 0, then calloc() returns either NULL, or a unique pointer value that can later be successfully passed to free().

The realloc() function changes the size of the memory block pointed to  by  ptr  to  size  bytes.   The  contents  will  be unchanged  in the range from the start of the region up to the minimum of the old and new sizes.  If the new size is larger than the old size, the added memory will not be initialized.  If ptr is NULL, then the call is equivalent to  malloc(size), for  all  values  of size; if size is equal to zero, and ptr is not NULL, then the call is equivalent to free(ptr).  Unless ptr is NULL, it must have been returned by an earlier call to malloc(), calloc() or realloc().  If the area pointed to  was moved, a free(ptr) is done.


malloc()函数分配指定大小字节并返回指向已分配内存的指针。内存未初始化。
如果size为0，则malloc()返回NULL或一个以后可以成功传递给free()的唯一指针值。

free()函数释放ptr指向的内存空间，该内存空间必须由之前调用malloc()，calloc()或realloc()返回。否则，或者如果之前已经调用了free(ptr)，则会发生未定义的行为。如果ptr为NULL，则不执行任何操作。

calloc()函数为每个大小为nmemb字节的元素数组分配内存，并返回指向该区域的指针，存储内容设置为零。如果nmemb或size为0，则calloc()返回NULL或一个以后可以成功传递给free()的唯一指针值。

realloc()函数将ptr指向的内存块的大小更改为size字节。内容将在从区域的开始到新旧尺寸的最小范围内保持不变。如果新大小大于旧的大小，则不会初始化添加的内存。如果ptr为NULL，则对于所有size值，调用等效于malloc(size)；如果size等于零，并且ptr不为NULL，则调用等效于free(ptr)。除非ptr为NULL，否则必须由之前调用malloc()，calloc()或realloc()返回。如果指向的区域被移动，则free(ptr)。

## new/delete关键字
C语言内存管理方式在C++中可以继续使用，但有些地方就无能为力而且使用起来比较麻烦，因此C++又提出了自己的内存管理方式：通过new和delete操作符进行动态内存管理

### new/delete操作基本数据类型

```cpp
#include<iostream>

using namespace std;

int main(){

	int *ptr = new int;
	int *ptr2 = new int(1);

	//cout << *ptr << endl;
	//cout << *ptr2 << endl;


	//int *arr = new int[3]; //OK 但是没有初始化
	//int *arr = new int[3]{}; //OK 初始化为全0
	//int *arr = new int[3]{1, 2, 3}; //OK 指定内容初始化
	//int *arr = new int[]{1, 2, 3}; //OK 指定内容初始化
	int *arr = new int[3]{1, 2}; //OK 制定部分内容初始化
	//int *arr = new int[3]{1, 2, 3, 4};//Error 指定个数与实际不符合

	for (int i = 0; i < 3;i++){
		cout << arr[i] << endl;
	}
	delete[] arr;
	return 0;
}
```

### new/delete操作类

```cpp
#include<iostream>
#include <stdlib.h>

using namespace std;

class Demo{
public:
	Demo(){
		cout << "构造函数" << endl;
	}

	~Demo(){
		cout << "析构函数" << endl;
	}
};
int main(){
	Demo *pd = (Demo*)malloc(sizeof(Demo));
	free(pd);
	pd = NULL;

	cout << endl;
	Demo *pd2 = new Demo();
	delete pd2;
	

	cout << endl;
	Demo *pd_arr = new Demo[10];
	delete[] pd_arr;
	return 0;
}
```
![](https://s2.ax1x.com/2019/05/07/EsGqy9.png)
注意：在申请自定义类型的空间时，new会调用构造函数，delete会调用析构函数，而malloc与free不会。

## malloc/free 与 new/delete区别
* **malloc/free和new/delete的共同点是：**
	* 都是从堆上申请空间，并且需要用户手动释放。

* **malloc/free和new/delete的不同点是：**
	* malloc和free是函数，new和delete是操作符
	* malloc申请的空间不能初始化，new可以初始化
	* malloc申请空间时，需要手动计算空间大小并传递，new只需在其后跟上空间的类型即可
	* malloc的返回值为void*, 在使用时必须强转，new不需要，因为new后跟的是空间的类型
	* malloc申请空间失败时，返回的是NULL，因此使用时必须判空，new不需要，但是new需要捕获异常
	* malloc/free只能申请内置类型的空间，不能申请自定义类型的空间，因为其不会调用构造与析构函数， 而new可以，new在申请空间后会调用构造函数完成对象的构造，delete在释放空间前会调用析构函数 完成空间中资源的清理
	* malloc申请的空间一定在堆上，new不一定，因为operator new函数可以重新实现 （new的空间可能在哪呢？）
	* new/delete比malloc和free的效率稍微低点，因为new/delete的底层封装了malloc/free




## operator new与operator delete函数
new和delete是用户进行动态内存申请和释放的操作符，operator new 和operator delete是系统提供的全局函数，new在底层调用operator new全局函数来申请空间，delete在底层通过operator delete全局函数来释放空间。


```cpp
/* 
operator new：该函数实际通过malloc来申请空间，当malloc申请空间成功时直接返回
申请空间失败，尝试执行空间不足应对措施，如果改应对措施用户设置了，则继续申请，否则抛异常。
*/
void *__CRTDECL operator new(size_t size) _THROW1(_STD bad_alloc)
{
	// try to allocate size bytes
	void *p;
	while ((p = malloc(size)) == 0)
	if (_callnewh(size) == 0)
	{
		// report no memory
		static const std::bad_alloc nomem;
		_RAISE(nomem);
	}
	return (p);
}
/*
operator delete: 该函数最终是通过free来释放空间的
*/
void operator delete(void *pUserData)
{
	_CrtMemBlockHeader * pHead;
	RTCCALLBACK(_RTC_Free_hook, (pUserData, 0));
	if (pUserData == NULL)
		return;
	_mlock(_HEAP_LOCK); /* block other threads */

	__TRY
		/* get a pointer to memory block header */
		pHead = pHdr(pUserData);
	/* verify block type */
	_ASSERTE(_BLOCK_TYPE_IS_VALID(pHead->nBlockUse));
	_free_dbg(pUserData, pHead->nBlockUse);
	__FINALLY
		_munlock(_HEAP_LOCK); /* release other threads */
	__END_TRY_FINALLY
		return;
}
```
通过上述两个全局函数的实现知道，operator new 实际也是通过malloc来申请空间，如果malloc申请空间成功就直接返回，否则执行用户提供的空间不足应对措施，如果用户提供该措施就继续申请，否则就抛异常。operator delete 最终是通过free来释放空间的。
注意：operator new和operator delete用户也可以自己实现，用户实现时即可实现成全局函数，也可实现成类的成员函数，但是一般情况下不需要实现，除非有特殊需求。

### operator new/operator delete与malloc/delete
new会调用构造方法，delete会调用析构函数
operator new 与 malloc 用法是一样的，不会调用构造函数
operator new 实际上就是malloc + 失败抛出异常
operator delete 与 free 用法是一样的，不会调用构造与析构函数
operator delete 实际上就是free + 失败抛出异常

## new和delete的实现原理

### 内置类型
如果申请的是内置类型的空间，new和malloc，delete和free基本类似，不同的地方是：new/delete申请和释放的是单个元素的空间，new[]和delete[]申请的是连续空间，而且new在申请空间失败时会抛异常，malloc会返回NULL。
### 自定义类型
**new的原理**
1. 调用`operator new`函数申请空间
2. 在申请的空间上执行构造函数，完成对象的构造

**delete的原理**

1. 在空间上执行析构函数，完成对象中资源的清理工作
2. 调用`operator delete`函数释放对象的空间

**new T[N]的原理**

1. 调用`operator new[]` 函数，在`operator new[]`中实际调用`operator new`函数完成N个对象空间的申请
2. 在申请的空间上执行N次构造函数

**delete[]的原理**

1. 在释放的对象空间上执行N次析构函数，完成N个对象中资源的清理
2. 调用`operator delete[]`释放空间，实际在`operator delete[]`中调用`operator delete`来释放空间

## 定位new表达式(placement-new)
定位new表达式是在已分配的原始内存空间中调用构造函数初始化一个对象。
使用格式：
```cpp
new (place_address) type
//或者
new (place_address) type(initializer-list)
```
place_address必须是一个指针，initializer-list是类型的初始化列表
使用场景：
定位new表达式在实际中一般是配合内存池使用。因为内存池分配出的内存没有初始化，所以如果是自定义类型的对象，需要使用new的定义表达式进行显示调构造函数进行初始化。

```cpp
#include <iostream>

class Demo{
public:
	Demo(){
		std::cout << "构造函数" << std::endl;
	}
};
int main(){
	//pt现在指向的只不过是与Test对象相同大小的一段空间，还不能算是一个对象，因为构造函数没有执行
	Demo *p = (Demo*)malloc(sizeof(Demo));
	
	new(p)Demo;// 注意：如果Demo类的构造函数有参数时，此处需要传参

	delete p;
	return 0;
}
```
## 如何设计只能在堆/栈上创建的类
### 只能在堆上创建的类
1. 将类的构造函数私有，拷贝构造声明成私有。防止别人调用拷贝在栈上生成对象。
2. 提供一个静态的成员函数，在该静态成员函数中完成堆对象的创建
```cpp
class HeapOnly{
public:
	static HeapOnly* CreatInstance(){
		  return new HeapOnly;
	}
private:
	HeapOnly(){}
	// 防拷贝
	HeapOnly(const HeapOnly&);
};
```
### 只能在栈上创建的类
只能在栈上创建对象，即不能在堆上创建，因此只要将new的功能屏蔽掉即可，即屏蔽掉operator new和定位new表达式，注意：屏蔽了operator new，实际也将定位new屏蔽掉
```cpp
class StackOnly{
public:
	StackOnly(){}
private:
	void* operator new(size_t size) = delete;
	void operator delete(void *p) = delete;
};
```

## 设计模式之单例模式
一个类只能创建一个对象，即单例模式，该模式可以保证系统中该类只有一个实例，并提供一个访问它的全局访问点，该实例被所有程序模块共享！

### 饿汉式单例模式
在程序初始化的时候就会创造出对象！

```cpp
class Singleton{
public:
	Singleton* GetInstance(){
		return &instance;
	}
private:
	Singleton(){}
	Singleton(const Singleton& other) = delete;
	Singleton& operator=(const Singleton& other) = delete;
	static Singleton instance;
};

Singleton Singleton::instance;// 在程序入口之前就完成单例对象的初始化
```
### 懒汉式单例模式
有时使用饿汉模式会导致程序启动时间变长，所以还有这种懒汉式单例模式来解决这个问题！

```cpp
#include <iostream>
#include <mutex>

class Singleton{
public:
	static Singleton* GetInstance(){
		// 注意这里一定要使用Double-Check的方式加锁，才能保证效率和线程安全
		if (m_pInstance == nullptr){
			m_mutex.lock();
			if (m_pInstance == nullptr){
				m_pInstance = new Singleton();
			}
			m_mutex.unlock();
		}
		return m_pInstance;
	}
	// 实现一个内嵌垃圾回收类
	class Recycle {
	public:
		~Recycle(){
			if (Singleton::m_pInstance)
				delete Singleton::m_pInstance;
		}
	};

	// 定义一个静态成员变量，程序结束时，系统会自动调用它的析构函数从而释放单例对象
	static Recycle recycle;
private:
	//构造函数私有
	Singleton(){}

	//防止拷贝
	Singleton(const Singleton& other) = delete;
	Singleton& operator=(const Singleton& other) = delete;

	static Singleton* m_pInstance; // 单例对象指针
	static std::mutex m_mutex;
};
Singleton* Singleton::m_pInstance = nullptr;
std::mutex Singleton::m_mutex;
Singleton::Recycle recycle;
```