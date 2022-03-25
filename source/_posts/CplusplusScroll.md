---
title: C++ 11 & C++14  
date: 2021-03-10 10:54:50
tags: C++
---



## 语言可用性强化  

### nullptr 与 constexpr  

### 类型推导  

在传统 C 和 C++中，参数的类型都必须明确定义，尤其是当我们面对一大堆复杂的模板类型时，必须明确的指出变量的类型才能进行后续的编码。

C++ 11 引入了 `auto` 和 `decltype` 这两个关键字实现了类型推导，让编译器来操心变量的类型。这使得 C++ 也具有了和其他现代编程语言一样，某种意义上提供了无需操心变量类型的使用习惯。

#### auto  

`auto` 在很早以前就已经进入了 C++，但是他始终作为一个存储类型的指示符存在，与 `register` 并存。

不使用auto书写迭代器:

```c++
for(vector<int>::const_iterator itr = vec.cbegin(); itr != vec.cend(); ++itr)
```

使用auto:

```c++  
for(auto itr = vec.cbegin(); itr != vec.cend(); ++itr);
```

其它用法：

```c++ 
auto i = 5;						//i被推导为 int
auto arr = new auto(10);		//arr被推导为 int *
```

notice: `auto` 不可用于函数传参和推导数组类型，如以下错误：

``` c++ 
int add(auto x, auto y);

{
    auto i = 5;
    int arr[10] = {0};
    auto auto_arr = arr;
    auto auto_arr2[10] = arr;
}
```

#### decltype  

`decltype` 关键字是为了解决 auto 关键字只能对变量进行类型推导的缺陷而出现的，其用法和`sizeof` 相似：

```c++ 
decltype(表达式);		//编译器分析表达式并得到其类型，却得不到其值

//计算某表达式的值
auto x = 1;
auto y = 2;
decltype(x + y) z;	  //z为int型
```

#### 尾返回类型、auto与decltype配合 

以下写法不能通过编译：

```c++ 
decltype(x + y) add(T x, U y);
```

C++11引入尾返回类型（trailing return type),利用`auto` 将返回类型后置:

```c++  
template<typename T, typename U>
auto add(T x, U y) -> decltype(x + y) {
    return x + y;
}
```

自C++14始，可直接让普通函数具备返回值推导，以下合法：

```c++  
template<template T, typename U>
auto add(T x, U y) {
    return x + y;
}
```

### 区间迭代  

区间迭代是指基于范围的 for 循环。

```c++
int array[] = {1, 2, 3, 4, 5};
for(auto &x : array) {
    std::cout << x << std::endl;
}

//std::vector的遍历
//不使用区间迭代
std::vector<int> arr(5, 100);
for(std::vector<int>::iterator i = arr.begin(); i != arr.end(); ++i) {
    std::cout<< *i << std::endl;
}
//使用区间迭代
//&启用引用，若无则元素可读不可修改
for(auto &i : arr) {
    std::cout<< i << std::endl;
}
```

### 初始化列表  

在传统 C++ 中，不同的对象有着不同的初始化方法，例如普通数组、POD （plain old data，没有构造、析构和虚函数的类或结构体）类型都可以使用 `{}` 进行初始化，也就是我们所说的初始化列表。而对于类对象的初始化，要么需要通过拷贝构造、要么就需要使用 `()` 进行。这些不同方法都针对各自对象，不能通用。

```c++  
int arr[3] = {1,2,3};   // 列表初始化

class Foo {
    private:
        int value;
    public:
        Foo(int) {}
};

Foo foo(1);             // 普通构造初始化
```

在C++11中使用`std::initializer_list` 类型允许构造函数或其它函数像参数一样使用初始化列表，这就为类对象的初始化与普通数组和 POD 的初始化方法提供了统一的桥梁，如：

```c++ 
#include <initializer_list>
class Happiness {
    public:
    	//初始化列表构造函数
    	Happiness(std::initializer_list<int> list) {}
};

Happiness happiness = {1, 2, 3, 4, 5};

std::vector<int> v = {1, 2, 3, 4};

//初始化列表作为普通函数的形参
void func(std::initializer_list<int> list)
{
    return;
}
func({1, 2, 3});

//C++11提供统一语法以初始化任意对象 
struct A {
    int a;
    float b;
};

struct B {
    B(int _a, float _b): a(_a), b(_b) {}
    private:
    	int a;
    	float b;
};

A a {1, 1.1};
B b {2, 2.2};
```

### 模板增强  

#### 外部模板  

传统 C++ 中，模板只有在使用时才会被编译器实例化。换句话说，只要在每个编译单元（文件）中编译的代码中遇到了被完整定义的模板，都会实例化。这就产生了重复实例化而导致的编译时间的增加。并且，我们没有办法通知编译器不要触发模板实例化。

C++11 引入了外部模板，扩充了原来的强制编译器在特定位置实例化模板的语法，使得能够显式的告诉编译器何时进行模板的实例化：

```c++ 
template class std::vector<bool>;			//强制实例化
extern template class std::vector<double>   //不再该编译文件中实例化模板
```

#### 尖括号">"

在传统 C++ 的编译器中，`>>`一律被当做右移运算符来进行处理，而自C++ 11 始，连续的右尖括号将变得合法，并且能够顺利通过编译:

```c++ 
//嵌套模板
std::vector<std::vector<int>> wow;
```

#### 类型别名模板  

模板是用来产生类型的，模板不是类型。在传统 C++中，`typedef` 可以为类型定义一个新的名称，但是却没有办法为模板定义一个新的名称。

```c++
template< typename T, typename U, int value>
class SuckType {
public:
    T a;
    U b;
    SuckType():a(value),b(value){}
};
template< typename U>
typedef SuckType<std::vector<int>, U, 1> NewType; // 不合法
```



C++ 11 使用 `using` 引入了下面这种形式的写法，并且同时支持对传统 `typedef` 相同的功效：

``` c++ 
//通常我们使用 typedef 定义别名的语法是：typedef 原名称 新名称;
//但是对函数指针等别名的定义语法却不相同，这通常给直接阅读造成了一定程度的困难。
typedef int (*process)(void *);  // 定义了一个返回类型为 int，参数为 void* 的函数指针类型，名字叫做 process
using process = int(*)(void *); // 同上, 更加直观

template <typename T>
using NewType = SuckType<int, T, 1>;    // 合法
```

#### 默认模板参数  

定义一个加法函数：

```c++ 
template<typename T, typename U>
auto add(T x, U y) -> decltype(x+y) {
    return x+y
}
```

要使用 add，就必须每次都指定其模板参数的类型。在 C++11 中提供了一种便利，可以指定模板的默认参数：

```c++
template<typename T = int, typename U = int>
auto add(T x, U y) -> decltype(x+y) {
    return x+y;
}
```

#### 变长参数模板  

模板一直是 C++ 所独有的黑魔法（**Dark Magic**）之一。在 C++11 之前，无论是类模板还是函数模板，都只能按其指定的样子，接受一组固定数量的模板参数；而 C++11 加入了新的表示方法，允许任意个数、任意类别的模板参数，同时也不需要在定义时将参数的个数固定。

```c++
//任意个数、任意类别的模板参数
template<typename... Ts> class Magic;
```

模板类 Magic 的对象，能够接受不受限制个数的 typename 作为模板的形式参数:

```c++
class Magic<int,
            std::vector<int>,
            std::map<std::string,
                     std::vector<int>>> darkMagic;
```

既然是任意形式，所以个数为 0 的模板参数也是可以的：`class Magic<> nothing;`，如果不希望产生的模板参数个数为 0，可以手动的定义至少一个模板参数：

```c++
//手动定义至少一个模板参数
template<typename Require, typename... Args> class Magic;
```

变长参数模板也能被直接调整到到模板函数上：

``` c++  
//传统 C 中的 printf 函数，虽然也能达成不定个数的形参的调用，但其并非类别安全
//C++11 除了能定义类别安全的变长参数函数外，还可以使类似 printf 的函数能自然地处理非自带类别的对象
//函数参数也使用 ... 代表不定长参数
template<typename... Args> void printf(const std::string &str, Args... args);
```

如何对变长的模板参数进行解包：

``` c++
template<typename... Args>
void magic(Args... args) {
    std::cout << sizeof...(args) << std::endl;
}

//我们可以传递任意个参数给 magic 函数：
magic();        // 输出0
magic(1);       // 输出1
magic(1, "");   // 输出2
```

对参数进行解包，到目前为止还没有一种简单的方法能够处理参数包，但有两种经典的处理手法：

##### 1. 递归模板函数  

递归是非常容易想到的一种手段，也是最经典的处理方法。这种方法不断递归的向函数传递模板参数，进而达到递归遍历所有模板参数的目的：

```c++
#include <iostream>
template<typename T>
void printf(T value) {
    std::cout << value << std::endl;
}
template<typename T, typename... Args>
void printf(T value, Args... args) {
    std::cout << value << std::endl;
    printf(args...);
}
int main() {
    printf(1, 2, "123", 1.1);
    return 0;
}
```

##### 2. 初始化列表展开  

递归模板函数是一种标准的做法，但缺点显而易见的在于必须定义一个终止递归的函数。以下记录了使用初始化列表展开的方法（**黑魔法**）：

```c++
// 编译这个代码需要开启 -std=c++14
//通过初始化列表，(lambda 表达式, value)... 将会被展开。
//由于逗号表达式的出现，首先会执行前面的 lambda 表达式，完成参数的输出。
//唯一不美观的地方在于如果不使用 return 编译器会给出未使用的变量作为警告。
template<typename T, typename... Args>
auto print(T value, Args... args) {
    std::cout << value << std::endl;
    return std::initializer_list<T>{([&] {
        std::cout << args << std::endl;
    }(), value)...};
}
int main() {
    print(1, 2.1, "123");
    return 0;
}
```

### 面向对象增强  

#### 委托构造  

C++ 11 引入了委托构造的概念，这使得在同一个类中一个构造函数调用另一个构造函数，从而达到**简化代码**的目的：

```c++
class Base {
public:
    int value1;
    int value2;
    Base() {
        value1 = 1;
    }
    Base(int value) : Base() {  // 委托 Base() 构造函数
        value2 = 2;
    }
};

int main() {
    Base b(2);
    std::cout << b.value1 << std::endl;
    std::cout << b.value2 << std::endl;
}
```

#### 继承构造  

在传统 C++ 中，构造函数如果需要继承是需要将参数一一传递的，这将导致效率低下。C++ 11 利用关键字 using 引入了继承构造函数的概念：

```c++
class Base {
public:
    int value1;
    int value2;
    Base() {
        value1 = 1;
    }
    Base(int value) : Base() {           // 委托 Base() 构造函数
        value2 = 2;
    }
};
class Subclass : public Base {
public:
    using Base::Base;                   // 继承构造
};
int main() {
    Subclass s(3);
    std::cout << s.value1 << std::endl;
    std::cout << s.value2 << std::endl;
}
```

#### 显式虚函数重载  

在传统 C++ 中，经常容易发生意外重载虚函数的事情，如：

```c++ 
struct Base {
    virtual void foo();
};
struct SubClass: Base {
    //可能是恰好加入的与基类中同名的函数或重载虚函数
    void foo();
};
```

C++ 11 引入了 `override` 和 `final` 这两个关键字来防止上述情形的发生：

##### override  

当重载虚函数时，引入 `override` 关键字将显式的告知编译器进行重载，编译器将检查基函数是否存在这样的虚函数，否则将无法通过编译：

``` c++ 
struct Base {
    virtual void foo(int);
};
struct SubClass: Base {
    virtual void foo(int) override; // 合法
    virtual void foo(float) override; // 非法, 父类没有此虚函数
};
```

##### final   

`final` 则是为了防止类被继续继承以及终止虚函数继续重载引入的。

```c++  
struct Base {
        virtual void foo() final;
};
struct SubClass1 final: Base {
};                  // 合法

struct SubClass2 : SubClass1 {
};                  // 非法, SubClass 已 final

struct SubClass3: Base {
        void foo(); // 非法, foo 已 final
};
```

##### 显式禁用默认函数  

在传统 C++ 中，如果程序员没有提供，编译器会默认为对象生成默认构造函数、复制构造、赋值算符以及析构函数。另外，C++ 也为所有类定义了诸如 `new` `delete` 这样的运算符。当程序员有需要时，可以重载这部分函数。

这就引发了一些需求：无法精确控制默认函数的生成行为。例如禁止类的拷贝时，必须将赋值构造函数与赋值算符声明为 `private`。尝试使用这些未定义的函数将导致编译或链接错误，则是一种非常不优雅的方式。

并且，编译器产生的默认构造函数与用户定义的构造函数无法同时存在。若用户定义了任何构造函数，编译器将不再生成默认构造函数，但有时候我们却希望同时拥有这两种构造函数，这就造成了尴尬。

C++11 提供了上述需求的解决方案，允许显式的声明采用或拒绝编译器自带的函数，如：

```c++  
class Magic {
public:
    Magic() = default;  // 显式声明使用编译器生成的构造
    Magic& operator=(const Magic&) = delete; // 显式声明拒绝编译器生成构造
    Magic(int magic_number);
}
```

### 强类型枚举   

在传统 C++ 中，枚举类型并非类型安全，枚举类型会被视作整数，则会让两种完全不同的枚举类型可以进行直接的比较（虽然编译器给出了检查，但并非所有），**甚至枚举类型的枚举值名字不能相同**，这不是我们希望看到的结果。

C++ 11 引入了枚举类（enumaration class），并使用 `enum class` 的语法进行声明：

```c++
enum class new_enum : unsigned int {
    value1,
    value2,
    value3 = 100,
    value4 = 100
};
```

这样定义的枚举实现了类型安全，首先他不能够被隐式的转换为整数，同时也不能够将其与整数数字进行比较，更不可能对不同的枚举类型的枚举值进行比较。但相同枚举值之间如果指定的值相同，那么可以进行比较：

```c++
if (new_enum::value3 == new_enum::value4) {
    // 会输出
    std::cout << "new_enum::value3 == new_enum::value4" << std::endl;
}
```

在这个语法中，枚举类型后面使用了冒号及类型关键字来指定枚举中枚举值的类型，这使得我们能够为枚举赋值（未指定时将默认使用 int）。

而我们希望获得枚举值的值时，将必须显式的进行类型转换，不过我们可以通过重载 `<<` 这个算符来进行输出：

```c++
#include <iostream>
template<typename T>
std::ostream& operator<<(typename std::enable_if<std::is_enum<T>::value, std::ostream>::type& stream, const T& e)
{
    return stream << static_cast<typename std::underlying_type<T>::type>(e);
}

std::cout << new_enum::value3 << std::endl
```

## 语言运行期强化  

### Lambda表达式  

Lambda 表达式是 C++ 11 中最重要的新特性之一，而 Lambda 表达式，实际上就是提供了一个类似匿名函数的特性，而匿名函数则是在需要一个函数，但是又不想费力去命名一个函数的情况下去使用的。这样的场景其实有很多很多，所以匿名函数几乎是现代编程语言的标配。

#### 基础知识 

Lambda 表达式的基本语法如下：

```c++
[捕获列表](参数列表) mutable(可选) 异常属性 -> 返回类型 {
    // 函数体
}
[ caputrue ] ( params ) opt -> ret { body; };
```

所谓捕获列表，其实可以理解为参数的一种类型，lambda 表达式内部函数体在默认情况下是不能够使用函数体外部的变量的，这时候捕获列表可以起到传递外部数据的作用。

根据传递的行为，捕获列表也分为以下几种：

#### 值捕获  

与参数传值类似，值捕获的前提是变量可以拷贝，不同之处则在于，被捕获的变量在 lambda 表达式被创建时拷贝，而非调用时才拷贝：

```c++  
void learn_lambda_func_1() {
    int value_1 = 1;
    auto copy_value_1 = [value_1] {
        return value_1;
    };
    value_1 = 100;
    auto stored_value_1 = copy_value_1();
    // 这时, stored_value_1 == 1, 而 value_1 == 100.
    // 因为 copy_value_1 在创建时就保存了一份 value_1 的拷贝
    cout << "value_1 = " << value_1 << endl;
    cout << "stored_value_1 = " << stored_value_1 << endl;
}
```

#### 引用捕获  

与引用传参类似，引用捕获保存的是引用，值会发生变化。

```c++
void learn_lambda_func_2() {
    int value_2 = 1;
    auto copy_value_2 = [&value_2] {
        return value_2;
    };
    value_2 = 100;
    auto stored_value_2 = copy_value_2();
    // 这时, stored_value_2 == 100, value_2 == 100.
    // 因为 copy_value_2 保存的是引用
    cout << "value_2 = " << value_2 << endl;
    cout << "stored_value_2 = " << stored_value_2 << endl;
}
```

#### 隐式捕获  

手动书写捕获列表有时候是非常复杂的，这种机械性的工作可以交给编译器来处理，这时候可以在捕获列表中写一个 `&` 或 `=` 向编译器声明采用引用捕获或者值捕获.

总结一下，捕获提供了 Lambda 表达式对外部值进行使用的功能，捕获列表的最常用的四种形式可以是：

- `[]` 空捕获列表
- `[name1, name2, ...]` 捕获一系列变量
- `[&]` 引用捕获, 让编译器自行推导捕获列表
- `[=]` 值捕获, 让编译器执行推导应用列表

#### 表达式捕获（C++14）   

值捕获、引用捕获都是已经在外层作用域声明的变量，因此这些捕获方式捕获的均为左值，而不能捕获右值。

C++14 允许捕获的成员用任意的表达式进行初始化，这就允许了右值的捕获，被声明的捕获变量类型会根据表达式进行判断，判断方式与使用 `auto` 本质上是相同的：

```c++  
#include <iostream>
#include <utility>
void learn_lambda_func_3(){
    auto important = std::make_unique<int>(1);
    auto add = [v1 = 1, v2 = std::move(important)](int x, int y) -> int {
        return x+y+v1+(*v2);
    };
    std::cout << "add(3, 4) = " << add(3, 4) << std::endl;
}
```

在上面的代码中，`important` 是一个独占指针，是不能够被捕获到的，这时候我们需要将其转移为右值，在表达式中初始化。

#### 泛型Lambda(C++ 14)  

我们知道 `auto` 关键字不能够用在参数表里，这是因为这样的写法会与模板的功能产生冲突。但是 Lambda 表达式并不是普通函数，所以 Lambda 表达式并不能够模板化。这就为我们造成了一定程度上的麻烦：参数表不能够泛化，必须明确参数表类型。

然而自 C++14 始，Lambda 函数的形式参数可以使用 `auto` 关键字来产生意义上的泛型：

```c++   
void learn_lambda_func_4(){
    auto generic = [](auto x, auto y) {
        return x+y;
    };

    std::cout << "generic(1,2) = " << generic(1, 2) << std::endl;
    std::cout << "generic(1.1,2.2) = " << generic(1.1, 2.2) << std::endl;
}
```

### 函数对象包装器   

#### std::function   

Lambda 表达式的本质是一个函数对象，当 Lambda 表达式的捕获列表为空时，Lambda 表达式还能够作为一个函数指针进行传递，例如：

```c++  
#include <iostream>

using foo = void(int);  // 定义函数指针
void functional(foo f) {
    f(1);
}

int main() {
    //Lambda表达式的捕获列表为空，可作为函数指针被传递
    auto f = [](int value) {
        std::cout << value << std::endl;
    };
    functional(f);  // 函数指针调用,f作为函数指针被传递
    f(1);           // lambda 表达式调用
    return 0;
}
```

上面的代码给出了两种不同的调用形式，一种是将 Lambda 作为函数指针传递进行调用，而另一种则是直接调用 Lambda 表达式，在 C++11 中，统一了这些概念，将能够被调用的对象的类型，统一称之为可调用类型。而这种类型，便是通过 `std::function` 引入的。

C++11 `std::function` 是一种通用、多态的**函数封装**，它的实例可以对任何可以调用的目标实体进行存储、复制和调用操作，它也是对 C++中现有的可调用实体的一种类型安全的包裹（相对来说，函数指针的调用不是类型安全的），换句话说，就是函数的容器。当我们有了函数的容器之后便能够更加方便的将函数、函数指针作为对象进行处理，如：

```c++  
#include <functional>
#include <iostream>

int foo(int para) {
    return para;
}

int main() {
    // std::function 包装了一个返回值为 int, 参数为 int 的函数
    std::function<int(int)> func = foo;

    int important = 10;
    std::function<int(int)> func2 = [&](int value) -> int {
        return 1+value+important;
    };
    std::cout << func(10) << std::endl;
    std::cout << func2(10) << std::endl;
}
```

#### std::bind/std::placeholder  

 `std::bind` 是用来绑定函数调用的参数的，它解决的需求是我们有时候可能并不一定能够一次性获得调用某个函数的全部参数，通过这个函数，我们可以将部分调用参数提前绑定到函数身上成为一个新的对象，然后在参数齐全后，完成调用，如：

```c++  
int foo(int a, int b, int c) {
    ;
}
int main() {
    // 将参数1,2绑定到函数 foo 上，但是使用 std::placeholders::_1 来对第一个参数进行占位
    auto bindFoo = std::bind(foo, std::placeholders::_1, 1,2);
    // 这时调用 bindFoo 时，只需要提供第一个参数即可
    bindFoo(1);
}
```

> **notice：**注意 `auto` 关键字的妙用。有时候我们可能不太熟悉一个函数的返回值类型，但是我们却可以通过 `auto` 的使用来规避这一问题的出现。

### 右值引用  

右值引用是 C++11 引入的与 Lambda 表达式齐名的重要特性之一。它的引入解决了 C++ 中大量的历史遗留问题，消除了诸如 `std::vector`、`std::string` 之类的额外开销，也才使得函数对象容器 `std::function` 成为了可能。



## 新增容器  

## 智能指针和引用计数  

## 正则表达式  

## 语言级线程支持  

## 其它  

## C++17  

