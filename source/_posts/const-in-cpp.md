---
title: const-in-cpp
date: 2012-09-01 12:43:51
tags: c
---
[原文地址](http://www.cnblogs.com/liangxiaxu/archive/2012/09/01/2666741.html)

## 用const 修饰函数的参数 
如果参数作输出用，不论它是什么数据类型，不论它采用“指针传递”还是“引用传递”，都不能加const 修饰，否则该参数将失去输出功能。 
const 只能修饰输入参数： 
(1) 如果输入参数采用“指针传递”，那么加const 修饰可以防止意外地改动该指针，起到保护作用。 
例如StringCopy 函数：void StringCopy(char *strDestination, const char *strSource); 
其中strSource 是输入参数，strDestination 是输出参数。给strSource 加上const 修饰后，如果函数体内的语句试图改动strSource 的内容，编译器将指出错误。 
(2) 如果输入参数采用“值传递”，由于函数将自动产生临时变量用于复制该参数，该输入参数本来就无需保护，所以不要加const 修饰。 
例如不要将函数void Func1(int x) 写成void Func1(const int x)。同理不要将函数void Func2(A a) 写成void Func2(const A a)。其中A 为用户自定义的数据类型。对于非内部数据类型的参数而言，象void Func(A a) 这样声明的函数注定效率比较底。因为函数体内将产生A 类型的临时对象用于复制参数a，而临时对象的构造、复制、析构过程都将消耗时间。 
为了提高效率，可以将函数声明改为void Func(A &a)，因为“引用传递”仅借用一下参数的别名而已，不需要产生临时对象。但是函数void Func(A &a) 存在一个缺点：“引用传递”有可能改变参数a，这是我们不期望的。解决这个问题很容易，加const 修饰即可，因此函数最终成为void Func(const A &a)。 
以此类推，是否应将void Func(int x) 改写为void Func(const int &x)，以便提高效率？完全没有必要，因为内部数据类型的参数不存在构造、析构的过程，而复制也非常快，“值传递”和“引用传递”的效率几乎相当。 
(3) 将“const & ”，修饰输入参数的用法总结一下。 
对于非内部数据类型的输入参数，应该将“值传递”的方式改为“const 引用传递”，目的是提高效率。例如将void Func(A a) 改为void Func(const A &a)。 
对于内部数据类型的输入参数，不要将“值传递”的方式改为“const 引用传递”。否则既达不到提高效率的目的，又降低了函数的可理解性。例如void Func(int x) 不应该改为void Func(const int &x)。 
## 用const 修饰函数的返回值 
如果给以“指针传递”方式的函数返回值加const 修饰，那么函数返回值（即指针）的内容不能被修改，该返回值只能被赋给加const 修饰的同类型指针。 
例如函数：const char * GetString(void); 
如下语句将出现编译错误： 
char *str = GetString(); 
正确的用法是const char *str = GetString(); 
如果函数返回值采用“值传递方式”，由于函数会把返回值复制到外部临时的存储单元中，加const 修饰没有任何价值。 
例如不要把函数int GetInt(void) 写成const int GetInt(void)。同理不要把函数A GetA(void) 写成const A GetA(void)，其中A 为用户自定义的数据类型。 
如果返回值不是内部数据类型，将函数A GetA(void) 改写为const A & GetA(void) 的确能提高效率。但此时千万千万要小心，一定要搞清楚函数究竟是想返回一个对象的“拷贝”还是仅返回“别名”就可以了，否则程序会出错。 
函数返回值采用“引用传递”的场合并不多，这种方式一般只出现在类的赋值函数中，目的是为了实现链式表达。例如：
```
class A
{
    A & operate = (const A & other); // 赋值函数
};
A a, b, c; // a, b, c 为A 的对象
a = b = c; // 正常的链式赋值
(a = b) = c; // 不正常的链式赋值，但合法
```
如果将赋值函数的返回值加const 修饰，那么该返回值的内容不允许被改动. 
上例中，语句 a = b = c 仍然正确，但是语句 (a = b) = c 则是非法的。 
## const 成员函数 
任何不会修改数据成员（即函数中的变量）的函数都应该声明为const 类型。如果在编写const 成员函数时，不慎修改了数据成员，或者调用了其它非const 成员函数，编译器将指出错误，这无疑会提高程序的健壮性。 
以下程序中，类stack 的成员函数GetCount 仅用于计数，从逻辑上讲GetCount 应当为const 函数。编译器将指出GetCount 函数中的错误。
```
class Stack
{
public:
    void Push(int elem);
    int Pop(void);
    int GetCount(void) const; // const 成员函数
private:
    int m_num;
    int m_data[100];
};
int Stack::GetCount(void) const
{
    ++ m_num; // 编译错误，企图修改数据成员m_num
    Pop(); // 编译错误，企图调用非const 函数
    return m_num;
}
```
const 成员函数的声明看起来怪怪的：const 关键字只能放在函数声明的尾部。 
关于const 函数的几点规则： 
a. const 对象只能访问const 成员函数，而非const 对象可以访问任意的成员函数，包括const 成员函数。 
b. const 对象的成员是不可修改的，然而const 对象通过指针维护的对象却是可以修改的。 
c. const 成员函数不可以修改对象的数据，不管对象是否具有const 性质。它在编译时，以是否修改成员数据为依据，进行检查。 
d. 加上mutable 修饰符的数据成员，对于任何情况下通过任何手段都可修改，自然此时的const 成员函数是可以修改它的。