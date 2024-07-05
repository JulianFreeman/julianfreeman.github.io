---
title: C++ 语言杂记
date: 2023-08-19 18:42:00 -0400
categories: [编程笔记, C++]
tags: [programming, cpp]
description: 本文记录了一些自己对 C++ 语言的理解
---

## 指针和引用

### 左值和右值

简单地说，

**左值表达式** 可以计算为 **变量** 或者 **其他可标识对象**，并且这些变量或对象可以 **在表达式结束之后存在**。

**右值表达式** 可以计算为 **字面量** 或者 **函数返回值** 或者 **运算结果**，并且这些结果会 **在表达式结束之后被丢弃**。

### 左值引用

左值引用变量只能引用 **可修改的左值**。

左值引用常量可以引用 **可修改与不可修改的左值**，甚至 **右值**。

所以左值引用常量更灵活一些，默认可以使用常量，除非一定要通过这个引用修改其值，再去掉常量限制。

```cpp
#include <iostream>

int main()
{
    int a{ 10 };
    int& ref{ a };

    const int& b{ a };
    const int& c{ ref };
    const int& cre{ 10 };

    return 0;
}
```

### 左值引用传递

对于拷贝速度很快的基本类型，默认用值传递，对于拷贝速度慢的复合类型，比如类等，默认用 **左值引用常量** 传递，也就是 `const Type&`。如果有其他需要，再更改。

函数传参其实就是用从函数外传过来的变量初始化函数参数，所以如果引用传递没有加常量限制的话，是可以在函数内通过这个引用修改函数外的实际的值的。跟指针一样。

## 纠结初始化

直接看结论：[总结](#总结)

### 分析

其实基本变量的初始化也好理解，形如 `int a;` 的就是只声明但没有任何初始化，形如 `int a{};` 就是默认初始化，一般可能是 `0` 之类的，形如 `int a{10};` 就是有值的初始化。

```cpp
#include <iostream>

int main()
{
    int a;
    int b{};
    int c{10};

    std::cout << a << '\n'; //32656
    std::cout << b << '\n'; //0
    std::cout << c << '\n'; //10

    return 0;
}
```

上述代码中，变量 `a` 只开辟了内存空间，但是没有初始化，所以它输出的是这块空间中的垃圾值，每次运行都可能是不一样的输出。剩下两个就好理解了，开辟空间之后，都填入了值，或者是默认值 `0` 或者是指定的值。

但其实问题在于类变量怎么初始化的？

做个实验：

```cpp
#include <iostream>

class Person
{
public:
    Person()
    {
        std::cout << "init\n";
    }
};

int main()
{
    Person john; //init
    return 0;
}
```

这里仅仅是声明了变量 `john` 但并没有用花括号，却也触发了构造函数。但是如果我们把这个例子改一改：

```cpp
#include <iostream>

class Person
{
public:
    Person(int id)
    {
        std::cout << id << "init\n";
    }
};

int main()
{
    Person john; //compiler error
    return 0;
}
```

因为我们自己写了构造函数，程序就不会自动给我们加无参数的默认构造函数了，这时候声明 `john` 的时候也编译出错了，说明即便没有加花括号，类变量还是会尝试调用无参数的构造函数，这等同于 `Person john{};` 。也就是说，对于类变量来说，声明时加不加空的花括号，都会调用无参的构造函数。有参的构造就很简单了，就不说了。

如果类中有成员变量呢，他们什么时候初始化的？

再来个实验：

```cpp
#include <iostream>

class Person
{
public:
    Person()
    {
        std::cout << "a = " << a << '\n'; //a = 32539
        std::cout << "b = " << b << '\n'; //b = 0
        std::cout << "c = " << c << '\n'; //c = 10
    }

private:
    int a;
    int b{};
    int c{10};
};

int main()
{
    Person john;
    return 0;
}
```

其实可以看到，这跟声明普通变量是一样的，声明成员变量时可以选择初始化或者不初始化，如果不初始化，该内存空间内的就是垃圾值。

如果成员变量也是类变量呢？

```cpp
#include <iostream>

class ID
{
public:
    ID()
    {
        std::cout << "ID init\n";
    }
};

class Person
{
public:
    Person()
    {
        std::cout << "Person init\n";
    }

private:
    ID id;
};

int main()
{
    Person john;
//    ID init
//    Person init
    return 0;
}
```

其实可以看到，成员变量是在构造函数之前初始化的，这也正常，因为毕竟构造函数是有可能用到成员变量的。这里 `ID id;` 和 `ID id{};` 是一样的作用，都会调用 `ID` 的构造函数，因为 `id` 是一个类变量。

如果所有的一切都变成指针呢？

```cpp
#include <iostream>

int main()
{
    int n{10};
    int* a;
    int* b{};
    int* c{&n};
    std::cout << a << '\n';
    std::cout << b << '\n';
    std::cout << c << '\n';

    return 0;
}
```

对于基本变量来说，其实是一样的，只不过类型都变成了指针而已，没有太大区别。 `int* a;` 因为没有初始化，所以内存空间里的是垃圾值，但是这个值在输出时会被转成地址的样式输出， `int* b{};` 默认使用 `nullptr` 初始化，当然输出的话也只是 `0` 而已，其他的好理解。

注意，此时， `a` 是一个野指针， `b` 是一个空指针，都不能解引用的。

对于类变量的话，可能就稍微有点不同了。

```cpp
#include <iostream>

class Person
{
public:
    Person()
    {
        std::cout << "init\n";
    }
};

int main()
{
    Person* john;
    Person* karl{};
    std::cout << john << '\n';
    std::cout << karl << '\n';
    return 0;
}
```

此时声明的 `john` 和 `karl` 都没调用构造函数，因为他们只是一个 `Person*` 类型的指针而已，本质上还是个指针，所以目前 `john` 是一个野指针，没经过初始化，会输出垃圾值， `karl` 是一个空指针，会输出 `0` 。

要想开辟空间，当然就得 `new` 了。

```cpp
#include <iostream>

class Person
{
public:
    Person()
    {
        std::cout << "init\n";
    }
    Person(int id)
    {
        std::cout << id <<" init\n";
    }
};

int main()
{
    Person* john{ new Person };
    Person* karl{ new Person{} };
    Person* liz{ new Person{ 10 } };
    delete liz;
    delete karl;
    delete john;
    return 0;
}
```

`new` 的时候， `new Person` 和 `new Person{}` 的作用是一样的，都会调用无参构造函数。

当然对于类的成员变量是指针的情况，也就好理解了。

其实指针变量的 `new` 初始化就是一个普通变量的有值初始化，因为 `new` 返回一个开辟空间的地址，然后用这个地址初始化指针变量的内存空间（对于 64 位的机器，这个内存空间就是 8 个字节）。

而 `new` 后面的操作就是在堆上开辟空间创建一个普通的类变量，只不过这个类变量是匿名的，只有它的地址被返回了。所以对待这个匿名类变量的初始化操作就跟初始化一个普通的类变量一样了（有没有空的花括号都会调用无参构造函数）。

### 总结

对于任何变量，不管基本变量还是类变量，如果声明时没有进行有值初始化，那就全都带上花括号进行默认初始化。因为尽管对于类变量不加花括号也会调用无参构造，但是不是很直观，而且跟基本变量的行为不同，所以为了统一行为，都加上花括号简单明了。

同时对于类中的成员变量也是一样的，不管是基本变量还是类变量，没有进行有值初始化，就全都加上空的花括号以声明是默认初始化。但是我们得知道，类中的成员变量在该类进行构造之前就完成了初始化，也就是说如果成员变量是非指针的类变量，那在该类构造之前它们就已经调用了一遍自己的构造函数了。

所有的指针变量包含在上述基本变量中。
