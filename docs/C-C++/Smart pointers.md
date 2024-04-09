# 智能指针

智能指针就是一个函数模版,防止内存泄漏
```cpp
template<class T>
class SmartPtr
{
public:
	// RAII
	SmartPtr(T* ptr)
		:_ptr(ptr)
	{}

	~SmartPtr()
	{
		cout << "delete:" << _ptr << endl;

		delete _ptr;
	}

	// 像指针一样
	T& operator*()
	{
		return *_ptr;
	}

	T* operator->()
	{
		return _ptr;
	}

private:
	T* _ptr;
};
```

下面再来说智能智能的拷贝问题

vector/list ...利用资源存储管理数据，资源都是自己的。拷贝时，每个对象各自一份资源，各管各的，所以深拷贝。

智能指针/迭代器... 本质资源不是自己的，代为持有，方便访问修改数据。他们拷贝的时候期望的指向同一个资源，所以浅拷贝。(实际上是赋值 + 置空)

auto_ptr管理权转移,被拷贝对象把资源管理权转移给拷贝对象,导致被拷贝对象悬空，注意拷贝过后不能访问被拷贝对象，否则就出现空指针了。很多公司禁止使用它，因为很坑。
```cpp
template<class T>
class auto_ptr
{
public:
	// RAII
	auto_ptr(T* ptr)
		:_ptr(ptr)
	{}

	// ap2(ap1)
	auto_ptr(auto_ptr<T>& ap)
	{
		_ptr = ap._ptr;
		ap._ptr = nullptr;
	}

	~auto_ptr()
	{
		cout << "delete:" << _ptr << endl;

		delete _ptr;
	}

	// 像指针一样
	T& operator*()
	{
		return *_ptr;
	}

	T* operator->()
	{
		return _ptr;
	}

private:
	T* _ptr;
};
```
unique_ptr禁止拷贝、禁止赋值
```cpp
template<class T>
class unique_ptr
{
public:
	// RAII
	unique_ptr(T* ptr)
		:_ptr(ptr)
	{}

	// ap2(ap1)
	unique_ptr(const unique_ptr<T>& ap) = delete;
	unique_ptr<T>& operator=(const unique_ptr<T>& ap) = delete;

	~unique_ptr()
	{
		cout << "delete:" << _ptr << endl;

		delete _ptr;
	}

	// 像指针一样
	T& operator*()
	{
		return *_ptr;
	}

	T* operator->()
	{
		return _ptr;
	}

private:
	T* _ptr;
};
```
shared_ptr,运用引用计数的方式来管理。那么如何引用计数呢？实际上是创建一个指针来计数。但是会出现循环引用的问题(解决办法是在链表里面用weak_ptr)。
```cpp
template<class T>
class shared_ptr
{
public:
	// RAII
	shared_ptr(T* ptr = nullptr)
		:_ptr(ptr)
		, _pcount(new int(1))
	{}

	// sp2(sp1)
	shared_ptr(const shared_ptr<T>& sp)
	{
		_ptr = sp._ptr;
		_pcount = sp._pcount;

		// 拷贝时++计数
		++(*_pcount);
	}

	// sp1 = sp4
	// sp4 = sp4;
	// sp1 = sp2;
	shared_ptr<T>& operator=(const shared_ptr<T>& sp)
	{
		//if (this != &sp)
		if (_ptr != sp._ptr)
		{
			release();

			_ptr = sp._ptr;
			_pcount = sp._pcount;

			// 拷贝时++计数
			++(*_pcount);
		}

		return *this;
	}

	void release()
	{
		// 说明最后一个管理对象析构了，可以释放资源了
		if (--(*_pcount) == 0)
		{
			cout << "delete:" << _ptr << endl;
			delete _ptr;
			delete _pcount;
		}
	}

	~shared_ptr()
	{
		// 析构时，--计数，计数减到0，
		release();
	}

	int use_count()
	{
		return *_pcount;
	}

	// 像指针一样
	T& operator*()
	{
		return *_ptr;
	}

	T* operator->()
	{
		return _ptr;
	}

	T* get() const
	{
		return _ptr;
	}
private:
	T* _ptr;
	int* _pcount;
};
```
weak_ptr
```cpp
// 不支持RAII，不参与资源管理
template<class T>
class weak_ptr
{
public:
	// RAII
	weak_ptr()
		:_ptr(nullptr)
	{}

	weak_ptr(const shared_ptr<T>& sp)
	{
		_ptr = sp.get();
	}

	weak_ptr<T>& operator=(const shared_ptr<T>& sp)
	{
		_ptr = sp.get();
		return *this;
	}

	// 像指针一样
	T& operator*()
	{
		return *_ptr;
	}

	T* operator->()
	{
		return _ptr;
	}

private:
	T* _ptr;
};
```