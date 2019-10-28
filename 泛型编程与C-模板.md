---
title: 泛型编程与C++模板
date: 2019-01-12 22:30:47
toc: true
categories: C/C++
---

## 模板初阶
模板就是让编译器去推到类型，从而使我们的代码更加简洁，复用性更好！

泛型编程：其实在高级语言中大多数都是支持泛型编程的，所谓泛型编程就是编写与类型无关的代码，是一种代码的复用，对于C++来说，模板就是实现泛型编程的基础，没有模板就没有STL，对于Java来说就没有集合框架，由此可见泛型编程的重要性！

### 函数模板
#### 函数模板格式	
注意：typename 是定义模板的关键字，<font color='red'>也可以用class代替，不可以用struct代替</font>
```cpp
template<typename t1,typename t2,...typename t3>
返回值类型 函数名(参数列表){ }
```
案例：（交换两个数字的值）

模板是一个蓝图，它本身并不是函数，是编译器产生特定具体类型函数的模具。所以其实模板就是将本来应该我们做的重复的事情交给了编译器。在编译阶段，对于模板函数的使用，编译器需要根据传入的实参类型来推演生成对应类型的函数以供调用。
例如：当用double类型使用函数模板时，编译器通过对实参类型的推演，将T确定为double类型，然后产生一份专门处理double类型的代码，所以模板只是把程序员应该写的代码交给了编译器去做，并没有减轻计算机的工作！

#### 显/隐式实例化
如果类型不匹配，编译器会尝试进行隐式类型转换，如果无法转换成功编译器将会报错
```cpp
template<class T>
T Func(const T& x1, const T& x2)
{
	return x1 - x2;
}
int main()
{
	Func(10, 20);
	Func(10.0, 20.0);
	//Func(a1, d1);Error，两个类型编译器不知道要用那个类型生成新的代码
	
	//解决方法:1.将d1强制类型转换为int或者把a1强制类型转换为double
	Func(10, (int)20.0);
	
	//解决方法:2.采用显式实例化
	Func<int>(10, 20.0);
	return 0;
}
```
#### 模板参数的匹配原则
一、一个非模板函数可以和一个同名的函数模板同时存在，而且该函数模板还可以被实例化为这个非函数模板函数
```cpp
#include <iostream>

using namespace std;

//专门处理int的函数
int Func(const int& x1, const int& x2)
{
	cout << "Sub(int,int)" << endl;
	return x1 - x2;
}
//调用的函数模板
template<class T>
T Func(const T& x1, const T& x2)
{
	cout << "Sub(T,T)" << endl;
	return x1 - x2;
}
int main()
{
	int a1 = 2, a2 = 3;
	Func(a1, a2);//与非模板函数匹配，编译器不需要特化 
	Func<int>(a1, a2);//调用编译器特化的函数
	return 0;
}
```
![](https://s2.ax1x.com/2019/05/08/E6W4zD.png)



二、对于非模板函数和同名函数模板，如果其他条件都相同，在调动时会优先调用非模板函数而不会从该模板产生出一个实例。如果模板可以产生一个具有更好匹配的函数， 那么将选择模板

```cpp
#include <iostream>

// 专门处理int的函数
int Func(int left, int right)
{
	std::cout << "Func(int,int)" << std::endl;
	return left + right;
}

// 通用模板函数
template<class T1, class T2>
T1 Func(T1 left, T2 right)
{
	std::cout << "Func(T1,T2)" << std::endl;
	return left + right;
}

void main()
{
	Func(1, 2); // 与非函数模板类型完全匹配，不需要函数模板实例化
	Func(1, 2.0); // 模板函数可以生成更加匹配的版本，编译器根据实参生成更加匹配的Add函数
}
```
![](https://s2.ax1x.com/2019/05/08/E6WosH.png)
三、显式指定一个空的模板实参列表，该语法告诉编译器只有模板才能来匹配这个调用， 而且所有的模板参数都应该根据实参推演出来

```cpp
#include <iostream>

//专门处理int的函数
int Func(const int& x1, const int& x2)
{
	std::cout << "Func(int, int)" << std::endl;
	return x1 - x2;
}
//调用的函数模板
template<class T>
T Func(const T& x1, const T& x2)
{
	std::cout << "Func(T, T)" << std::endl;
	return x1 - x2;
}
int main()
{
	Func(1, 2);//与非函数模板类型完全匹配，不需要函数模板实例化
	Func<>(1, 2);//调用模板生成的Add函数
	return 0;
}
```
![](https://s2.ax1x.com/2019/05/08/E6WTLd.png)
四、模板函数不允许自动类型转换，但普通函数可以进行自动类型转换

### 类模板
```cpp
template<class T1, class T2, ..., class Tn>
class 类模板名{ };
```
类模板实例化与函数模板实例化不同，类模板实例化需要在类模板名字后跟<>，然后将实例化的类型放在<> 中即可，类模板名字不是真正的类，而实例化的结果才是真正的类。
## 模板进阶
### 非类型模板参数
模板参数分类类型形参与非类型形参：
* 类型形参：出现在模板参数列表中，跟在class或者typename之类的参数类型名称
* 非类型形参：就是用一个常量作为类(函数)模板的一个参数，在类(函数)模板中可将该参数当成常量来使用

```cpp
#include <iostream>

using namespace std;

// 定义一个模板类型的静态数组
template<class T, size_t N = 10>
class Array {

private:
	T _arr[N];
	size_t _size;
};
```
浮点数、类对象以及字符串是不允许作为非类型模板参数的！
非类型的模板参数必须在编译期就能确认结果，比如10+20、rand()...都是可以的!
### 模板的特化
通常情况下，使用模板可以实现一些与类型无关的代码，但对于一些特殊类型的可能会得到一些错误的结果，如下所示：
```cpp
template<class T>
bool IsEqual(T& x1, T& x2)
{
	return x1 == x2;
}
void Test()
{
	char* s1 = "hello";
	char* s2 = "world";

	if (IsEqual(s1, s2))
		cout << "Equal" << endl;
	else
		cout << "No Equal" << endl;
}
```
此时，就需要对模板进行特化。即：在原模板类的基础上，针对特殊类型所进行特殊化的实现方式。模板特化中分为函数模板特化与类模板特化！
#### 函数模板特化
函数模板的特化步骤：
1. 必须要先有一个基础的函数模板
2. 关键字template后面接一对空的尖括号<>
3. 函数名后跟一对尖括号，尖括号中指定需要特化的类型
4. 函数形参表: 必须要和模板函数的基础参数类型完全相同，如果不同编译器可能会报一些奇怪的错误

```cpp
template<>
bool IsEqual<char*>(char*& left, char*& right)
{
	if (strcmp(left, right) == 0)
		return true;
	return false;
}
```
一般情况下如果函数模板遇到不能处理或者处理有误的类型，为了实现简单通常都是将该函数直接给出而不是进行特化，模板匹配时自动会匹配类型严格的函数，匹配的规则在上面已经说到！
#### 类模板特化
类模板特化又分为全特化与偏特化：
##### 全特化
全特化指的是在类模板的基础上，再重新定义一个类，该类与类模板的内容完全一致，唯一的区别是指定了类模板的所有类型
```cpp
#include <iostream>

template<class T1, class T2>
class Data
{
public:
	Data() { std::cout << "Data<T1, T2>" << std::endl; }
private:
	T1 _d1;
	T2 _d2;
};

template<>
class Data<int, char>
{
public:
	Data() { std::cout << "Data<int, char>" << std::endl; }
private:
	int _d1;
	char _d2;
};

int main()
{
	Data<int, int> d1;
	Data<int, char> d2;
}
```
![](https://s2.ax1x.com/2019/05/08/E6WbdI.png)
##### 偏特化
任何针对模版参数进一步进行条件限制设计的特化版本。比如对于以下模板类：
```cpp
template<class T1, class T2>
class Data
{
public:
	Data() {std::cout << "Data<T1, T2>" << std::endl;}
private:
	T1 _d1;
	T2 _d2;
};
```
偏特化有以下两种表现形式：
① 部分特化：将参数模板类表中的一部分参数特化
```cpp
template<class T1>
class Data<T1, int>
{
public:
	Data() { std::cout << "Data<T1, T2>" << std::endl; }
private:
	T1 _d1;
	T2 _d2;
};
```
② 对参数更进一步的限制：偏特化并不仅仅是指特化部分参数，而是针对模板参数更进一步的条件限制所设计出来的一个特化版本
```cpp
//两个参数偏特化为指针类型
template<class T1, class T2>
class Data<T1*, T2*>
{
public:
	Data() { std::cout << "Data<T1*, T2*>" << std::endl; }
private:
	T1* _d1;
	T2* _d2;
};

//两个参数偏特化为引用类型
template<class T1, class T2>
class Data<T1&, T2&>
{
public:
	Data(const T1& d1, const T2& d2)
	:_d1(d1),
	_d2(d2) { std::cout << "Data<T1&, T2&>" << std::endl; }
private:
	T1& _d1;
	T2& _d2;
};

Data<int , double> d2; // 调用基础的模板
Data<double , int> d1; // 调用特化的int版本
Data<int *, int*> d3; // 调用特化的指针版本
Data<int&, int&> d4(1, 2); // 调用特化的指针版本
```
### 类模板特化应用之类型萃取
现在假设我们要实现一个通用的拷贝函数：
#### 使用memcpy\循环复制
```cpp
template<class T>
void Copy(T* dst, const T* src, size_t size)
{
	memcpy(dst, src, sizeof(T)*size);
}

template<class T>
void Copy(T* dst, const T* src, size_t size)
{
	for (size_t i = 0; i < size; ++i)
	{
		dst[i] = src[i];
	}
}
```
拷贝自定义类型对象就可能会出错，因为自定义类型对象有可能会涉及到深拷贝(比如string)，而memcpy属于浅拷贝。如果对象中涉及到资源管理，就只能用赋值，用循环赋值的方式虽然可以，但是代码的效率比较低，而C/C++程序最大的优势就是效率高。那能否将另种方式的优势结合起来呢？遇到内置类型就用memcpy来拷贝，遇到自定义类型就用循环赋值方式来做呢？

答案是肯定的，但是由用户来判断是自定义类型还是内置类型有时难免传参会出错，所以优先使用函数自动推导来帮助我们完成这个问题：
```cpp
#include <string>
bool IsPODType(const char* strType)
{
	const char* arrType[] = { "char", "short", "int", "long", "long long", "float", "double", "long double" };
	for (size_t i = 0; i < sizeof(arrType) / sizeof(arrType[0]); ++i)
	{
		if (0 == strcmp(strType, arrType[i]))
			return true;
	}
	return false;
}

template<class T>
void Copy(T* dst, const T* src, size_t size)
{
	if (IsPODType(typeid(T).name()))
		memcpy(dst, src, sizeof(T)*size);
	else
	{
		for (size_t i = 0; i < size; ++i)
			dst[i] = src[i];
	}
}
```
运行时类型识别[（Run-Time Type Identification）RTTI](http://www.cppblog.com/smagle/archive/2010/05/14/115286.aspx)，RTTI允许应用程序在执行期间标识一个对象的类型，在非多态语言(如C语言)中找不到这个概念的。非多态语言不需要运行时的类型信息，因为每个对象的类型在编译时就已经确定了。但是在支持多态的语言中(如C++)，可能存在这种情况：在编译时你并不知道某个对象的类型信息，而只有在程序运行时才能获得对象的准确信息。C++是通过类的层次结构、虚函数以及基类指针来实现多态的。基类指针可以用来指向基类的对象或者其派生类的对象，也就是说，我们并不总是能够在任何时刻都预先知道基类指针所指向对象的实际类型。因此，必须在程序中使用"运行时类型识别"来识别对象的实际类型。typeid返回指针或引用所指对象的实际类型!
#### 类型萃取
为了将内置类型与自定义类型区分开，给出以下两个类分别代表内置类型与自定义类型。
```cpp
// 代表内置类型
struct TrueType{
	static bool Get(){
		return true;
	}
};
// 代表自定义类型
struct FalseType{
	static bool Get(){
		return false;
	}
};

template<class T>
struct TypeTraits
{
	typedef FalseType IsPODType;
};

template<>
struct TypeTraits<char>
{
	typedef TrueType IsPODType;
};
template<>
struct TypeTraits<short>
{
	typedef TrueType IsPODType;
};
template<>
struct TypeTraits<int>
{
	typedef TrueType IsPODType;
};
template<>
struct TypeTraits<long>
{
	typedef TrueType IsPODType;
};

// ... 所有内置类型都特化一下
template<class T>
void Copy(T* dst, const T* src, size_t size)
{
	if (TypeTraits<T>::IsPODType::Get())
		memcpy(dst, src, sizeof(T)*size);
	else
	{
		for (size_t i = 0; i < size; ++i)
		dst[i] = src[i];
	}
}
```
T为int的时候：`TypeTraits<int>`已经特化过，程序运行时就会使用已经特化过的`TypeTraits<int>`, 该类中的IsPODType刚好为类TrueType，而TrueType中Get函数返回true，内置类型使用memcpy方式拷贝，T为string：`TypeTraits<string>`没有特化过，程序运行时使用TypeTraits类模板, 该类模板中的IsPODType刚好为类FalseType，而FalseType中Get函数返回true，自定义类型使用赋值方式拷贝

在STL中也使用了类型萃取，可以参考[【STL】类型萃取（TypeTraits）](https://blog.csdn.net/bit_clearoff/article/details/53728516)

### 模板分离编译
一个程序（项目）由若干个源文件共同实现，而每个源文件单独编译生成目标文件，最后将所有目标文件链接起来形成单一的可执行文件的过程称为分离编译模式

C/C++程序要运行，一般经历以下步骤：
预处理--->编译--->汇编--->链接

预处理：头文件展开、宏替换、条件编译、去注释、
编译：语法检查、生成汇编代码
汇编：生成机器码，生成目标文件
链接：把目标文件组合起来，生成可执行程序或者动（静）态库

现假设有a.hpp、a.cpp、main.cpp等文件
![](https://s2.ax1x.com/2019/05/08/E6WjW8.png)
模板不支持分离编译的解决方案：
1.将声明和定义放到一个文件 "xxx.hpp" 里面或者xxx.h其实也是可以的。推荐使用这种
2.模板定义的位置显式实例化。这种方法不实用，不推荐使用

### 模板总结
优点：模板复用了代码，节省资源，更快的迭代开发，C++的标准模板库(STL)因此而产生，同时模板也增强了代码的灵活性

缺点：模板会导致代码膨胀问题，也会导致编译时间变长，出现模板编译错误时，错误信息非常凌乱，不易定位错误