---
title: Cpp11-lambda表达式
tags:
  - c++
  - cpp
  - lambda
categories:
  - C++
  - 基础
cover: 'https://s2.loli.net/2022/12/08/VAJF2c3elutXjDM.png'
abbrlink: 47235
date: 2023-03-24 14:56:36
---

# 引言
lambda表达式是C++11最重要也是最常用的特性之一。lambda来源于函数式编程的概念，也是现代编程语言的一个特点。lambda表达式有如下优点：
- 声明式编程风格：就地匿名定义目标函数或函数对象，不需要额外写一个命名函数或者函数对象。以更直接的方式去写程序，好的可读性和可维护性。
- 简洁：不需要额外再写一个函数或者函数对象，避免了代码膨胀和功能分散，让开发者更加集中精力在手边的问题，同时也获取了更高的生产率。
- 在需要的时间和地点实现功能闭包，使程序更灵活。

# lambda表达式的概念和基本用法
lambda表达式定义了一个匿名函数，并且可以捕获一定范围内的变量。lambda表达式的语法形式可简单归纳如下：
```
[capture](params)opt->ret{body;};
```
> 其中capture是捕获列表，params是参数列表，opt是函数选项，ret是返回值类型，body是函数体。

因此，一个完整的lambda表达式看起来像这样：
```cpp
...
auto f = [](int a) -> int { return a + 1; };

std::cout << f(1) << std::endl; // 输出： 2
...
```

可以看到，上面通过一行代码定义了一个小小的功能闭包，用来将输入加1并返回。

在很多时候，lambda表达式的返回值是非常明显的，比如下面这个例子。因此，C++11中允许省略lambda表达式的返回值定义：
```cpp
...
auto f = [](int a){ return a + 1; };
...
```
这样编译器就会根据return语句自动推导出返回值类型。

需要注意的是，初始化列表不能用于返回值的自动推导：
```cpp
...
auto x1 = [](int i) { return i; }; //OK: return type is int
auto x2 = []() { return {1, 2}; }; // error:无法推导出返回值类型
...
```
这时我们需要显式给出具体的返回值类型。

另外，lambda表达式在没有参数列表时，参数列表是可以省略的。因此像下面的写法都是正确的：
```cpp
...
auto f1 = []() { return 1; };
auto f2 = [] { return 1; }; // 省略空参数列表
...
```

# 使用lambda表达式捕获列表
lambda表达式还可以通过捕获列表捕获一定范围的变量：
- []不捕获任何变量
- [&]捕获外部作用域中所有变量，并作为引用在函数体中使用（按引用捕获）。
- [=]捕获外部作用域中所有变量，并作为副本在函数体中使用（按值捕获）。
- [=, &foo]按值捕获外部作用域中所有变量，并按引用捕获foo变量。
- [bar]按值捕获bar变量，同时不捕获其它变量。
- [this]按值捕获当前类中的this指针，让lambda表达式拥有和当前类成员函数同样的访问权限。如果已经使用了&或者=，就默认添加此选项。捕获this的目的是可以在lambda中使用当前的成员函数和成员变量。

下面看一下它的具体用法：
```cpp
class A
{
    public:
    int i_ = 0;
    void func(int x, int y)
    {
        auto x1 = []{ return i_; };                    // error，没有捕获外部变量
        auto x2 = [=]{ return i_ + x + y; };           // OK，捕获所有外部变量
        auto x3 = [&]{ return i_ + x + y; };           // OK，捕获所有外部变量
        auto x4 = [this]{ return i_; };                // OK，捕获this指针
        auto x5 = [this]{ return i_ + x + y; };        // error，没有捕获x、y
        auto x6 = [this, x, y]{ return i_ + x + y; };  // OK，捕获this指针、x、y
        auto x7 = [this]{ return i_++; };              // OK，捕获this指针，并修改成员的值
    }
};
```

从上例中可以看到，lambda 表达式的捕获列表精细地控制了 lambda 表达式能够访问的外部变量，以及如何访问这些变量。

需要注意的是，默认状态下 lambda 表达式无法修改通过复制方式捕获的外部变量。如果希望修改这些变量的话，我们需要使用引用方式进行捕获。

一个容易出错的细节是关于 lambda 表达式的延迟调用的：
```cpp
...
int a = 0;
auto f = [=]{ return a; };      // 按值捕获外部变量
a += 1;                         // a被修改了
std::cout << f() << std::endl;  // 输出？
...
```
在这个例子中，lambda 表达式按值捕获了所有外部变量。在捕获的一瞬间，a 的值就已经被复制到f中了。之后 a 被修改，但此时 f 中存储的 a 仍然还是捕获时的值，因此，最终输出结果是 0。

**如果希望 lambda 表达式在调用时能够即时访问外部变量，我们应当使用引用方式捕获。**

从上面的例子中我们知道，按值捕获得到的外部变量值是在 lambda 表达式定义时的值。此时所有外部变量均被复制了一份存储在 lambda 表达式变量中。此时虽然修改 lambda 表达式中的这些外部变量并不会真正影响到外部，我们却仍然无法修改它们。

那么如果希望去**修改按值捕获的外部变量**应当怎么办呢？这时，需要显式指明 lambda 表达式为 mutable：

```cpp
...
int a = 0;
auto f1 = [=]{ return a++; };               // error, 修改按值捕获的外部变量
auto f2 = [=]() mutable { return a++; };    // OK, mutable
...
```
需要注意的一点是，被mutable修饰的lambda表达式就算没有参数也要写明参数列表

# lambda表达式的类型
最后，介绍一下lambda表达式的类型。
lambda表达式的类型在C++11中被称为“闭包类型（Closure Type）“。因此，我们可以认为它是一个带有operator()的类——对括号运算符进行了重载的类，即仿函数。因此，我们可以使用std::function和std::bind来存储和操作lambda表达式：
```cpp
...
std::function<int(int)> f1 = [](int a) { return a; };
std::function<int(void)> f2 = std::bind([](int a){ return a; }, 123);
f1(555); // 555
f2(); // 123
...
```
另外，**对于没有捕获任何变量的lambda表达式**，还可以被转换成一个普通的函数指针：
```cpp
...
using funct_t = int(*)(int);
func_t f = [](int a) { return a; };
f(123);
...
```
lambda表达式可以说是就地定义仿函数闭包的”语法糖“。它的捕获列表捕获住的任何外部变量，最终均会变为**闭包类型的成员变量**。而一个使用了成员变量的类的operator(),如果能直接被转换为普通的函数指针，**那么lambda表达式本身的this指针就丢失掉了**。而没有捕获任何外部变量的lambda表达式则不存在这个问题。

这里也可以很自然地解释为何按值捕获无法修改捕获的外部变量（这里指的是原始外部变里的值的副本，在闭包内部的一个成员变量）。因为按C++标准，lambda表达式的operator()默认是const的。一个const成员函数是无法修改成员变量的值的。而mutable的作用，就在于取消operator()的const。

需要注意的是，没有捕获变量的lambda表达式可以直接转换为函数指针，而捕获变量的lambda表达式则不能转换为函数指针。看看下面的代码：
```cpp
...

typedef void(*Ptr)(int*);

Ptr p = [](int* p) { delete p; }; //正确，无状态的lambda（没有捕获）表达式可以直接转换为函数指针
Ptr p1 = [&](int* p) { delete p; }; // 错误，有状态的lambda不能直接转换为函数指针
...
```
上面第二行代码能编译通过，而第三行代码不能编译通过，因为第三行代码捕获了变量，不能直接转为函数指针。

# 声明式的编程风格，简洁的代码
就地定义匿名函数，不再需要定义函数对象，大大简化了标准库算法的调用。比如，在C++11之前，我们要调用for_each函数将vecotr中的偶数打印出来，如下所示。

【实例】lambda表达式代替函数对像的示例。
```cpp
class CountEven
{
    int& count_;
public:
    CountEven(int& count) : count_(count) {}
    void operator()(int val)
    {
        if (!(val & 1))       // val % 2 == 0
        {
            ++ count_;
        }
    }
};
```
```cpp
...
std::vector<int> v = { 1, 2, 3, 4, 5, 6 };
int even_count = 0;
for_each(v.begin(), v.end(), CountEven(even_count));
std::cout << "The number of even is " << even_count << std::endl;
...
```

这样写既烦琐又容易出错。有了lambda表达式以后，我们可以使用真正的闭包概念来替换掉这里的仿函数，代码如下：
```cpp
std::vector<int> v = { 1, 2, 3, 4, 5, 6 };
int even_count = 0;
for_each( v.begin(), v.end(), [&even_count](int val)
        {
            if (!(val & 1))  // val % 2 == 0
            {
                ++ even_count;
            }
        });
std::cout << "The number of even is " << even_count << std::endl;
```
lambda表达式的价值在于，就地封装短小的功能闭包，可以极其方便地表达出我们希望执行的具体操作，并让上下文结合得更加紧密。


