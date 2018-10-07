---
layout: post
category: C++
title: 继承中的两个小陷阱
tagline: 正确理解 virtual 函数
tags: [C++]
---
继承中有两个容易出错的地方：**虚函数**和**析构函数**

<!--more-->
先看代码示例：

~~~cpp
#include <iostream>
#include <string>
using namespace std;

class Base
{
public:
    Base() {
        std::cout << "Base::Base()" << std::endl;
    }
    virtual ~Base() {
        std::cout << "Base::~Base()" << std::endl;
    }

    virtual void func(string param = "Base param") {
        std::cout << "In Base::func(" << param << ")" << std::endl;
    }
};

class Derived : public Base
{
public:
    Derived() {
        std::cout << "Derived::Derived()" << std::endl;
    }
    ~Derived() {
        std::cout << "Derived::~Derived()" << std::endl;
    }

    void func(string param = "Derived param") {
        std::cout << "In Derived::func(" << param << ")" << std::endl;
    }
};

int main(int argc, char *argv[])
{
    Base *pbase = new Derived;
    pbase->func();
    delete pbase;
    return 0;
}
~~~

## 虚函数与其默认参数

使用基类指针指向子类对象，可以实现多态的功能。例如定义一个“动物”类的基类数组，然后将各动物实例放入其中，然后用遍历的方式去执行一个基类函数时，各子类对此函数均有自己的实现，即实现多态。

基类指针指向子类对象时，基类的指针是如何调用子类函数的？
对于类的普通函数来说，函数只与类有关，也就是实例本身不会保存函数的运行地址。
但是对于虚函数（加了virtual关键字的函数），C++ 在基类中保存了一个指针来指向其实现函数，例如执行`Base *pbase = new Derived`时，Base 中的 func 指针便指向了 Derived::func 函数。即便该实例是通过Base指针访问的，基类依然可以找到 Derived::func 函数。

但是虚函数的默认参数就没有记录在实例中了，这样就会导致以上面的代码中调用`pbase->func();`时输出的结果是

```
In Derived::func(Base param)
```

由这个结果可以看出来，虚函数虽然会运行正确的函数，但是默认参数却只能从当前指针对应的类中去取，此处使用的 Base 对应的指针来运行 func 函数，所以取出的默认参数是 Base::func() 中指定的默认参数。

## 你的子类实例正确析构了吗？

看到过有的代码在写基类的析构函数时没有在前面加`virtual`，经过上面的分析，如果基类析构前面没有写`virtual`，那么在实现多态的时候根据基类的指针只能取到基类Base的析构函数。会导致 Derived::~Derived 执行不到。

文章开始的例子完整输出结果为

~~~
Base::Base()
Derived::Derived()
In Derived::func(Base param)
Derived::~Derived()
Base::~Base()
~~~

如果漏写了~Base前的`virtual`，那么执行结果如下：

~~~
Base::Base()
Derived::Derived()
In Derived::func(Base param)
Base::~Base()
~~~
对比之后，可以明显看出如果漏写基类析构函数前的`virtual`，则子类实例的内存释放会不完全。如果子类中有内存申请操作，准备在析构时释放内存，那么就会有内存泄漏的隐患。
