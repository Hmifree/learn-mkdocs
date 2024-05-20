# C++的类型转换

有一定关联的类型才可以互相转换C

整形之间 隐式类型转换

整形和浮点数 隐式类型转换

bool和整形 bool和指针 隐式类型转换

指针和整形 强制类型转换

不同类型的指针之间  强制类型转换

CPP

构造函数只支持

内置类型->自定义类型之间，本质借助构造 隐式类型转换  如：string和const char*

自定义类型之间->内置类型, 本质要重载一个operator类型 隐式类型转换 如：下面的A

```cpp
class A
{
public:
	operator int()
	{
		return _a1 + _a2;
	}
private:
	int _a1 = 1;
	int _a2 = 2;
};
```
自定义类型之间->自定义类型之间，本质借助构造，隐式类型转换 如：initializer_list和容器

## static_cast

static_cast用于非多态类型的转换（静态转换），类型编译器隐式执行的任何类型转换都可用
static_cast，但它不能用于两个不相关的类型进行转换，隐式类型转换

##  reinterpret_cast

reinterpret_cast操作符通常为操作数的位模式提供较低层次的重新解释，用于将一种类型转换
为另一种不同的类型,强制类型转换

##  const_cast

const_cast最常用的用途就是删除变量的const属性，方便赋值

强制类型转换,但是为什么要把去掉const属性单独拿出来?

就是专门提醒,去掉const属性是有一些内存可见优化的风险,要注意是否加了volatile

```cpp
int main() {
	volatile const int a1 = 2;
	int* p1 = const_cast<int*>(&a1);
	*p1 = 3;
	cout << a1 << endl;
	cout << *p1 << endl;
}
```

## dynamic_cast

向上转型：子类对象指针/引用->父类指针/引用(不需要转换，赋值兼容规则)
向下转型：父类对象指针/引用->子类指针/引用(用dynamic_cast转型是安全的)

注意：

1. dynamic_cast只能用于父类含有虚函数的类
2. dynamic_cast会先检查是否能转换成功，能成功则转换，不能则返回0

```cpp
class A
{
public:
	virtual void f() {}
	
	int _a = 0;
};

class B : public A
{
public:
	int _b = 1;
};

void fun(A* pa)
{
	// 向下转换：父->子
	// pa指向子类对象，转回子类，是安全的
	// pa指向父类对象，转回子类，是不安全的，存在越界的风险问题

	// 不安全
	//B* pb = (B*)pa;
	
	//  pa指向子类对象，转回子类，正常转换
	//  pa指向父类对象，转回子类，转换失败
	B* pb = dynamic_cast<B*>(pa);
	if (pb)
	{
		cout << pb << endl;
		cout << pb->_a << endl;
		cout << pb->_b << endl;
	}
	else
	{
		cout << "转换失败" << endl;
	}
}
```
