# 特殊类设计

## 设计一个类，不能被拷贝

拷贝只会放生在两个场景中：拷贝构造函数以及赋值运算符重载，因此想要让一个类禁止拷贝，
只需让该类不能调用拷贝构造函数以及赋值运算符重载即可。

+ C++98

将拷贝构造函数与赋值运算符重载只声明不定义，并且将其访问权限设置为私有即可。

```cpp
class CopyBan
{
    // ...
    
private:
    CopyBan(const CopyBan&);
    CopyBan& operator=(const CopyBan&);
    //...
};
```

原因：

1. 设置成私有：如果只声明没有设置成private，用户自己如果在类外定义了，就可以不
能禁止拷贝了
2. 只声明不定义：不定义是因为该函数根本不会调用，定义了其实也没有什么意义，不写
反而还简单，而且如果定义了就不会防止成员函数内部拷贝了。

+ C++11

C++11扩展delete的用法，delete除了释放new申请的资源外，如果在默认成员函数后跟上
=delete，表示让编译器删除掉该默认成员函数。

```cpp
class CopyBan
{
    // ...
    CopyBan(const CopyBan&)=delete;
    CopyBan& operator=(const CopyBan&)=delete;
    //...
};
```

## 设计一个类，只能在堆上创建对象

实现方式：

1. 将类的构造函数私有，拷贝构造声明成私有。防止别人调用拷贝在栈上生成对象。
2. 提供一个静态的成员函数，在该静态成员函数中完成堆对象的创建

```cpp
// 1.
class HeapOnly {
public:
	template<class... Args>
	static HeapOnly* CreateObj(Args&&... args) {
		return new HeapOnly(args...);
	}

	HeapOnly(const HeapOnly&) = delete;

	HeapOnly& operator=(const HeapOnly&) = delete;
private:
	HeapOnly()
	{}

	HeapOnly(int x, int y)
		:_x(x)
		,_y(y)
	{}

	int _x;
	int _y;
};

// 2.
class HeapOnly {
public:
	HeapOnly()
	{}

	HeapOnly(int x, int y)
		:_x(x)
		, _y(y)
	{}

	void Destroy() {
		delete this;
	}
private:
	~HeapOnly()
	{
		;
	}

	int _x;
	int _y;
};

```

## 设计一个类，只能在栈上创建对象

```cpp
class StackOnly {
public:
	template<class... Args>
	static StackOnly CreateObj(Args&&... args) {
		return StackOnly(args...);
	}

	//StackOnly(const StackOnly&) = delete;

	void* operator new(size_t size) = delete;

	void operator delete(void*) = delete;

	StackOnly& operator=(const StackOnly&) = delete;
private:
	StackOnly()
	{}

	StackOnly(int x, int y)
		:_x(x)
		, _y(y)
	{}

	int _x;
	int _y;
};
```

## 设计一个类，不能被继承

+ C++98方式

```cpp
// C++98中构造函数私有化，派生类中调不到基类的构造函数。则无法继承
class NonInherit
{
public:
 static NonInherit GetInstance()
 {
 return NonInherit();
 }
private:
 NonInherit()
 {}
};
```

+ C++11方法
final关键字，final修饰类，表示该类不能被继承。

```cpp
class A  final
{
    // ....
};
```

## 设计一个类，只能创建一个对象(单例模式)

饿汉:一开始(main之前)就创建出对象
问题:
1. 如果单例对象数据较多,构造初始化成本较高,那么会影响程序启动的速度,迟迟进不了main函数
2. 多个单例类有初始化启动依赖关系,饿汉无法控制。假设A和B两个单例，假设要求A先初始化，B再初始化，饿汉无法保证
```cpp
class Singleton
{
public:
	static Singleton* GetInstance()
	{
		return &_sint;
	}

	void Print() {
		cout << _x << " " << _y << " ";
		for (auto& e : _vstr)
			cout << e << " ";
	}

	void Addstr(string s) {
		_vstr.push_back(s);
	}
	Singleton(const Singleton&) = delete;
	Singleton& operator=(const Singleton&) = delete;
private:
	Singleton(int x = 0, int y = 0, vector<string> vstr = { "hello", "world" })
		:_x(x)
		, _y(y)
		, _vstr(vstr)
	{}
	int _x;
	int _y;
	vector<string> _vstr;
	static Singleton _sint;
};
Singleton Singleton::_sint(1, 1, { "四川","北京" });
```

懒汉
```cpp
// 1.
class Singleton
{
public:
	static Singleton* GetInstance()
	{
		if (_psint == nullptr)
			_psint = new Singleton;
		return _psint;
	}

	void Print() {
		cout << _x << " " << _y << " ";
		for (auto& e : _vstr)
			cout << e << " ";
	}

	static void DelSingleton() {
		if (_psint) {
			delete _psint;
			_psint = nullptr;
		}
	}

	void Addstr(string s) {
		_vstr.push_back(s);
	}
	Singleton(const Singleton&) = delete;
	Singleton& operator=(const Singleton&) = delete;

	
private:
	Singleton(int x = 0, int y = 0, vector<string> vstr = { "hello", "world" })
		:_x(x)
		, _y(y)
		, _vstr(vstr)
	{}

	~Singleton()
	{
		cout << "~Singleton()" << endl;
	}
	int _x;
	int _y;
	vector<string> _vstr;
	static Singleton* _psint;

	class GC {
	public:
		~GC() {
			Singleton::DelSingleton();
		}
	};
	static GC gc;
};
Singleton* Singleton::_psint = nullptr;
Singleton::GC Singleton::gc;


// 2.
class Singleton
{
public:
	static Singleton* GetInstance()
	{
		// 局部的静态对象，第一次调用函数时构造初始化
		// C++11及之后这样写才可以
		// C++11之前无法保证这里的构造初始化是线程安全
		static Singleton _sint;
		return &_sint;
	}

	void Print() {
		cout << _x << " " << _y << " ";
		for (auto& e : _vstr)
			cout << e << " ";
	}


	void Addstr(string s) {
		_vstr.push_back(s);
	}
	Singleton(const Singleton&) = delete;
	Singleton& operator=(const Singleton&) = delete;


private:
	Singleton(int x = 0, int y = 0, vector<string> vstr = { "hello", "world" })
		:_x(x)
		, _y(y)
		, _vstr(vstr)
	{}

	~Singleton()
	{
		cout << "~Singleton()" << endl;
	}
	int _x;
	int _y;
	vector<string> _vstr;
};
```