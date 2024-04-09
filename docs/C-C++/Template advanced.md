# 模版进阶

## 非类型模板参数

> 模板参数分类类型形参与非类型形参。
> 类型形参即：出现在模板参数列表中，跟在class或者typename之类的参数类型名称。
> 非类型形参，就是用一个常量作为类(函数)模板的一个参数，在类(函数)模板中可将该参数当成常量来使用。

C++里面的array就是一个典型的非类型模版参数
```cpp
#include <iostream>
#include <array>
using namespace std;

int main() {
    array<int, 10> a1;
    cout << sizeof(a1) << endl; // 40
    return 0;
}
```
array唯一的优势就是可以越界检查，但是vector也具有这个的功能，而且还能初始化。

## 函数模版的特化

概念

通常情况下，使用模板可以实现一些与类型无关的代码，但对于一些特殊类型的可能会得到一些错误的结
果，需要特殊处理，比如：实现了一个专门用来进行小于比较的函数模板
```cpp
// 函数模板 -- 参数匹配
template<class T>
bool Less(T left, T right)
{
 return left < right;
}
int main()
{
 cout << Less(1, 2) << endl; // 可以比较，结果正确
 Date d1(2022, 7, 7);
 Date d2(2022, 7, 8);
 cout << Less(d1, d2) << endl; // 可以比较，结果正确
 Date* p1 = &d1;
 Date* p2 = &d2;
 cout << Less(p1, p2) << endl; // 可以比较，结果错误
 return 0;
}
```
可以看到，Less绝对多数情况下都可以正常比较，但是在特殊场景下就得到错误的结果。上述示例中，p1指
向的d1显然小于p2指向的d2对象，但是Less内部并没有比较p1和p2指向的对象内容，而比较的是p1和p2指
针的地址，这就无法达到预期而错误。
此时，就需要对模板进行特化。即：在原模板类的基础上，针对特殊类型所进行特殊化的实现方式。模板特
化中分为函数模板特化与类模板特化。

函数模板特化

函数模板的特化步骤：

1. 必须要先有一个基础的函数模板

2. 关键字template后面接一对空的尖括号<>

3. 函数名后跟一对尖括号，尖括号中指定需要特化的类型

4. 函数形参表: 必须要和模板函数的基础参数类型完全相同，如果不同编译器可能会报一些奇怪的错误。

```cpp
// 函数模板 -- 参数匹配
template<class T>
bool Less(T left, T right)
{
 return left < right;
}
// 对Less函数模板进行特化
template<>
bool Less<Date*>(Date* left, Date* right)
{
     return *left < *right;
}
```
注意：一般情况下如果函数模板遇到不能处理或者处理有误的类型，为了实现简单通常都是将该函数直接给
出。

该种实现简单明了，代码的可读性高，容易书写，因为对于一些参数类型复杂的函数模板，特化时特别给
出，因此函数模板不建议特化。

## 类模版特化

1.全特化

全特化即是将模板参数列表中所有的参数都确定化。

```cpp
template<class T1, class T2>
class Data
{
public:
 Data() {cout<<"Data<T1, T2>" <<endl;}
private:
 T1 _d1;
 T2 _d2;
};
template<>
class Data<int, char>
{
public:
 Data() {cout<<"Data<int, char>" <<endl;}
private:
 int _d1;
 char _d2;
};
```

2.偏特化
偏特化有以下两种表现方式：
+ 部分特化
```cpp
// 将第二个参数特化为int
template <class T1>
class Data<T1, int>
{
public:
 Data() {cout<<"Data<T1, int>" <<endl;}
private:
 T1 _d1;
 int _d2;
}; 
```

参数更进一步的限制

偏特化并不仅仅是指特化部分参数，而是针对模板参数更进一步的条件限制所设计出来的一个特化版本。可以管理指针
```cpp
//两个参数偏特化为指针类型
template <class T1, class T2>
class Data <T1*, T2*>
{ 
public:
 Data() {cout<<"Data<T1*, T2*>" <<endl;}
 
private:
 T1 _d1;
 T2 _d2;
};

//两个参数偏特化为引用类型
template <typename T1, typename T2>
class Data <T1&, T2&>
{
public:
 Data(const T1& d1, const T2& d2)
 : _d1(d1)
 , _d2(d2)
 {
 cout<<"Data<T1&, T2&>" <<endl;
 }
 
private:
 const T1 & _d1;
 const T2 & _d2; 
 };
```

匹配顺序: 全特化 > 偏特化 > 原模版, 特化是建立在原模版之上,否则特化无效

坑：那么为什么不能写成这样
```cpp
template <class T1, class T2>
class Data <T1*, T2*>
{
public:
 Data(const T1*& d1, const T2*& d2)
 : _d1(d1)
 , _d2(d2)
 {
    cout<<"Data<T1&, T2&>" <<endl;
 } 
private:
 const T1 & _d1;
 const T2 & _d2; 
};
```

因为Data*转换成const Data *权限缩小, 会产生临时变量, 临时变量具有常数项所以不能被引用。
写成Data(const T1* const& d1, const T2* const& d2)才行,但是最好不要写这样

## 模板分离编译

**什么是分离编译**

一个程序（项目）由若干个源文件共同实现，而每个源文件单独编译生成目标文件，最后将所有目标文件链
接起来形成单一的可执行文件的过程称为分离编译模式。

模板的分离编译

假如有以下场景，模板的声明与定义分离开，在头文件中进行声明，源文件中完成定义：

```cpp
// a.h
template<class T>
T Add(const T& left, const T& right);
// a.cpp
template<class T>
T Add(const T& left, const T& right)
{
 return left + right;
}
```
在调用Add时编译器在链接时才会找其地址,但是这两个函数没有实例化没有生成具体代码,因此链接时报错。

那么为什么不在编译时去搜索文件需要什么类型呢？因为这样会大大降低模版的效率,如果文件内容过大，会耗费许多时间.

**解决办法**
+ 1.直接定义在.h -> 调用的地方,直接就有定义,直接实例化,不需要链接时再去找
+ 2.模版类型显示实例化
```cpp
// a.h
template
int Add<int>(const int&, const int&);
```
那么这样模版就没有意义了,不如直接自己定义,很挫.

## 模版总结

模版的本质是，本来应该由你写的多份代码。现在不需要你重复写了，你提供一个模版，编译器根据你的实例化，帮你去写出来。

**优点**

1.模板复用了代码，节省资源，更快的迭代开发，C++的标准模板库(STL)因此而产生

2.增强了代码的灵活性

**缺陷**

1.模板会导致代码膨胀问题，也会导致编译时间变长

2.出现模板编译错误时，错误信息非常凌乱，不易定位错误

编译错误不好定位发现,最佳实践方案:排除法,一段一段注释.日常,建议写一部分,编译一部分.

