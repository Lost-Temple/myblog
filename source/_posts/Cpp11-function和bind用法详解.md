---
title: Cpp11-function和bind用法详解
tags:
  - function
  - bind
categories:
  - C++
  - 基础
cover: 'https://s2.loli.net/2022/12/08/VAJF2c3elutXjDM.png'
abbrlink: 32955
date: 2023-03-27 17:59:42
---

在设计回调函数的时候，无可避免地会接触到可回调对象。在C++11中，提供了std::function和std::bind两个方法来对可回调对象进行统一和封装。

# 可调用对象
C++ 中有如下几种可调用对象：函数、函数指针、lambda表达式、bind对象、函数对象。其中，lambda表达式和bind对象是C++11标准中提出的（bind机制并不是新标准中首次提出，而是对旧版本中bind1st和bind2nd的合并）。个人认为五种可调用对象中，函数和函数指针本质相同，而lambda表达式、bind对象及函数对象则异曲同工。

# 函数
这里的函数指的是普通函数，没什么可拓展的。

# 函数指针
函数指针和函数类型的区别：
- 函数指针指向的是函数而非对象。和其它指针类型一样，函数指针指向某种特定类型；
- 函数类型由它的返回值和参数类型决定，与函数名无关

例：
```cpp
bool fun(int a, int b)
```
上述函数的函数类型是：`bool(int, int)`

上述函数的函数指针`pf`是：`bool (*pf)(int, int)`

一般对于函数来说，函数名即为函数指针：
```cpp
#include <iostream>

int fun(int x, int y) {
    std::cout << x + y << std::endl;
    return x + y;
}

int fun1(int (*fp)(int, int), int x, int y) {
    return fp(x, y);
}

typedef int (*Ftype)(int, int);
int fun2(Ftype fp, int x, int y) {
    return fp(x, y);
}

//int main() {
//    fun1(fun, 100, 100);
//    fun2(fun, 200, 200);
//}

```
上述例子就是将函数指针作为参数，再使用函数指针对函数进行调用。传参（函数指针）时，只需要把函数名称传入即可

# lambda表达式
lambda表达式就是一段可调用的代码。主要适合于只用到一两次的简短代码段。由于lambda是匿名的，所以保证了其不会被不安全地访问：
```cpp
#include <iostream>

int fun3(int x, int y) {
    auto f = [](int x, int y) { return x + y; };
    std::cout << f(x, y) << std::endl;
}

int main() {
   fun3(300, 300);
}
```

# bind对象
std::bind可以用来生产一个可调用对象，来适应原对象的参数列表。

# 函数对象
重载了函数调用运算符`()`的类的对象，即为函数对象。

# std::function
由上文可以看出：由于可调用对象的定义方式比较多，但是函数的调用方式较为类似，因此需要使用一个统一的方式保存可调用对象或者传递可调用对象。于是，`std::function`就诞生了。

`std::function`是一个可调用对象包装器，是一个类模板，可以容纳除了类成员函数指针之外的所有可调用对象，它可以用统一的方式处理函数、函数对象、函数指针，并允许保存和延迟它们的执行。

定义function的一般形式：
```cpp
#include <functional>

std::function<函数类型>
```

例如：
```cpp
#include <functional>
#include <iostream>

typedef std::function<int(int, int)> comfun;

// 普通函数
int add(int a, int b) { return a + b; }

// lambda表达式
auto mod = [](int a, int b) {
    return a % b;
};

// 函数对象
struct divide {
    int operator()(int denominator, int divisor) {
        return denominator / divisor;
    }
};

int main() {
   comfun a = add;
   comfun b = mod;
   comfun c = divide();
   std::cout << a(5, 3) << std::endl;
   std::cout << b(5, 3) << std::endl;
   std::cout << c(5, 3) << std::endl;
}

```
std::function可以取代函数指针的作用，因为它可以延迟函数的执行，特别适合作为`回调函数`使用。它比普通函数指针更加的灵活和便利。

故而，std::function的作用可以归纳为：
1. std::function对C++中各种可调用实体（普通函数、Lambda表达式、函数指针、以及其它函数对象等）的封装，形成一个新的可调用的std::function对象，简化调用；
2. std::function对象是对C++中现有的可调用实体的一种类型安全的包裹（如：函数指针这类可调用实体，是类型不安全的）。

# std::bind
std::bind可以看作一个通用的函数适配器，**它接受一个可调用对象，生成一个新的可调用对象来适应原对象的参数列表**。

std::bind将可调用对象与其参数一起进行绑定，绑定后的结果可以使用std::function保存。std::bind主要有以下两个作用：
- 将可调用对象和其参数绑定成一个仿函数；
- 只绑定部分参数，减少可调用对象传入的参数；

调用bind的一般形式：
```cpp
auto newCallable = bind(callable, arg_list);
```
该形式表达的意思是：当调用`newCallable`时，会调用`callable`，并传给它`arg_list`中的参数。

需要注意的是：`arg_list`中的参数可能包含形如`_n`的名字。其中`n`是一个整数，这些参数是占位符，表示`newCallable`的参数，它们占据了传递给`newCallable`的参数的位置。数值n表示生成的可调用对象中参数的位置：`_1`为`newCallable`的第一个参数，`_2`为第`2`个参数，以此类推。

看下面代码：
```cpp

#include <iostream>

class A {
public:
    void fun_3(int k, int m) {
        std::cout << "print: k = " << k << ", m = " << m << std::endl;
    }
};

void fun_1(int x, int y, int z) {
    std::cout << "print: x = " << x << ", y = " << y << ", z = " << z << std::endl;
}

void fun_2(int &a, int &b) {
    ++a;
    ++b;
    std::cout << "print: a = " << a << ", b = " << b << std::endl;
}

int main(int argc, char *argv[]) {
    //f1的类型为function<void(int, int, int)>
    auto f1 = std::bind(fun_1, 1, 2, 3); // 表示绑定函数fun_1 的第一，二，三个参数值为： 1, 2, 3
    f1();

    // 表示绑定函数 fun_1的第三个参数为3，而fun_1函数第一，二个参数分别由调用f3的时指定，注意，这里传参的顺序是可以根据placeholders::_n 指定的
    auto f2 = std::bind(fun_1, std::placeholders::_1, std::placeholders::_2, 3);
    f2(1, 2);

    // 注意下面的std::placeholders::_n 的顺序
    auto f3 = std::bind(fun_1, std::placeholders::_2, std::placeholders::_1, 3);
    f3(1, 2);

    int m = 2;
    int n = 3;
    auto f4 = std::bind(fun_2, std::placeholders::_1, n); // 表示绑定了fun_2的第二个参数为n，第二个参数由f4调用时的第一个参数指定
    f4(m);
    std::cout << "m = " << m << std::endl; // m=3 说明bind对于不预先绑定的参数，是引用传递的
    std::cout << "n = " << n << std::endl; // n=3 说明bind对于预先绑定的参数，会产生一个值拷贝，而不是使用的n的引用

    A a;
    //f5的类型为function<void(int, int)>
    auto f5 = std::bind(&A::fun_3, &a, std::placeholders::_1, std::placeholders::_2);
    f5(10, 20); // 调用a.fun_3(10, 20)

    auto f6 = std::bind(&A::fun_3, a, std::placeholders::_1, std::placeholders::_2);
    f6(10, 20); // 调用a.fun_3(10, 20)

    std::function<void(int, int)> fc = std::bind(&A::fun_3, a, std::placeholders::_1, std::placeholders::_2);
    fc(10, 20);

    return 0;
}
```
由此例子可以看出：
- 预绑定的参数是以值传递的形式，不预绑定的参数要用std::placeholders(占位符)的形式占位，从_1开始，依次递增，是以引用传递的形式；（这里的值传递或引用传递是针对bind生成对象来说的，并不是指可调用对象的调用）
- std::placeholders表示新的可调用对象的参数的占位，其中的_n与原函数的该占位符所在位置进行匹配；也就是说bind产生的新的可调用对象的参数，可以和原函数的参数顺序不一致。
- std::bind绑定类成员函数时，第一个参数表示对象的成员函数的指针，第二个参数表示对象的地址（这里好像可以直接传对象的变量进去，应该是会做隐式的转换，即取地址，具体没研究过），这是因为对象的成员函数需要有this指针。并且编译器不会将对象的成员函数指针，需要通过显式& 取地址进行转换。
- std::bind的返回值是可调用实体，可以直接赋给std::function。


