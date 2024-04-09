# 初始C++
 
## 命名空间

在 C/C++ 中，变量、函数和后面要学到的类都是大量存在的，这些变量、函数和类的名称将都存
在于全局作用域中，可能会导致很多冲突。使用命名空间的目的是 对标识符的名称进行本地化 ，
以 避免命名冲突或名字污染 ， namespace 关键字的出现就是针对这种问题的。
```cpp
#include <stdio.h>
#include <stdlib.h>
int rand = 10;
// C语言没办法解决类似这样的命名冲突问题，所以C++提出了namespace来解决
int main()
{
 printf("%d\n", rand);
return 0;
}
// 编译后后报错：error C2365: “rand”: 重定义；以前的定义是“函数”
```

**命名空间的定义**
定义命名空间，需要使用到namespace关键字，后面跟命名空间的名字，然后后面跟一对{}即可,{}中即为命名空间的成员。
```cpp
// 正常的命名空间
namespace lkt
{
 // 命名空间中可以定义变量/函数/类型
 int rand = 10;
 int Add(int left, int right)
 {
 return left + right;
 }
 struct Node
 {
     struct Node* next;
     int val;
 };
}
 
//2. 命名空间可以嵌套
// test.cpp
namespace N1
{
int a;
int b;
int Add(int left, int right)
 {
     return left + right;
 }
namespace N2
 {
     int c;
     int d;
     int Sub(int left, int right)
     {
         return left - right;
     }
 }
}
//3. 同一个工程中允许存在多个相同名称的命名空间,编译器最后会合成同一个命名空间中。
// ps：一个工程中的test.h和上面test.cpp中两个N1会被合并成一个
// test.h
namespace N1
{
int Mul(int left, int right)
 {
     return left * right;
 }
}
```

**命名空间的使用**
```cpp
namespace lkt
{
 // 命名空间中可以定义变量/函数/类型
 int a = 0;
 int b = 1;
 int Add(int left, int right)
 {
 return left + right;
 }
 struct Node
 {
 struct Node* next;
 int val;
 };
}
int main()
{
 // 编译报错：error C2065: “a”: 未声明的标识符
 printf("%d\n", a);
return 0;
}
```

**加命名空间作用域限定符**
```cpp
int main()
{
    printf("%d\n", lkt::a);
    return 0;    
}
```

**使用using将命名空间某个成员加入**
```cpp
using lkt::b;
int main()
{
    printf("%d\n", lkt::a);
    printf("%d\n", b);
    return 0;    
}
```

**使用using namespace命名空间导入**
```cpp
using namespce lkt;
int main()
{
    printf("%d\n", lkt::a);
    printf("%d\n", b);
    Add(10, 20);
    return 0;    
}
```

## C++的输入和输出
```cpp
#include<iostream>
// std是C++标准库的命名空间名，C++将标准库的定义实现都放到这个命名空间中
using namespace std;
int main()
{
    cout<<"Hello world!!!"<<endl;
    return 0;
}
```

**说明**
1. 使用cout标准输出对象(控制台)和cin标准输入对象(键盘)时，必须包含< iostream >头文件
以及按命名空间使用方法使用std。
2. cout和cin是全局的流对象，endl是特殊的C++符号，表示换行输出，他们都包含在包含< 
iostream >头文件中。
3. <<是流插入运算符，>>是流提取运算符。
4. 使用C++输入输出更方便，不需要像printf/scanf输入输出时那样，需要手动控制格式。
C++的输入输出可以自动识别变量类型

```cpp
#include <iostream>
using namespace std;
int main()
{
   int a;
   double b;
   char c; 
   // 可以自动识别变量的类型
   cin>>a;
   cin>>b>>c;
     
   cout<<a<<endl;
   cout<<b<<" "<<c<<endl;
   return 0;
}
```

**std命名空间使用惯例**
std是C++标准库的命名空间，如何展开std使用更合理呢？
1. 在日常练习中，建议直接using namespace std即可，这样就很方便。
2. using namespace std展开，标准库就全部暴露出来了，如果我们定义跟库重名的类型/对
象/函数，就存在冲突问题。该问题在日常练习中很少出现，但是项目开发中代码较多、规模
大，就很容易出现。所以建议在项目开发中使用，像std::cout这样使用时指定命名空间 + 
using std::cout展开常用的库对象/类型等方式

## 缺省参数

**缺省参数的概念**
```cpp
void Func(int a = 0)
{
 cout<<a<<endl;
}
int main()
{
 Func();     // 没有传参时，使用参数的默认值
 Func(10);   // 传参时，使用指定的实参
return 0;
}
```

**缺省参数的分类**



