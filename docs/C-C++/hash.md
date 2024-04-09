# 哈希
## 概念

存储的值和存储位置的映射的关联关系

## 常见的哈希函数：

1.直接定址法--(常用)

取关键字的某个线性函数为散列地址：Hash（Key）= A*Key + B

优点：简单、均匀

缺点：需要事先知道关键字的分布情况

使用场景：适合查找比较小且连续的情况

2.除留余数法--(常用)

设散列表中允许的地址数为m，取一个不大于m，但最接近或者等于m的质数p作为除数，

按照哈希函数：Hash(key) = key% p(p<=m),将关键码转换成哈希地址

3.创建一个哈希表
```cpp
enum State {
	EMPTY,
    EXIST,
	DELETE
};

template<class K, class V>
struct HashData {
	pair<K, V> _kv;
	State _state = EMPTY;
};

template<class K, class V>
struct HashTable {
public:
private:
	vector<HashData<K, V>> _tables;
	size_t n = 0;
};
```

## 哈希冲突解决
a. 闭散列开放定址法(本质是当前位置冲突了,后面找一个合适的位置继续存储(x: 线性探测 y: 二次探测))

b. 开散列拉链法/哈希桶

### 线性探测解决哈希冲突
插入：

i = key % 表的大小

如果i位置已经有值了,就往后走找空位置,放进去

查找:

i = key % 表的大小

如果i位不是要查找的key就往后查找,直到找到或者遇到空

如果找到表结尾位置,要往头回绕

哈希冲突越多,效率就越低

负载因子/载荷因子 = 实际存进去的数据个数/表的大小

闭散列（开放定址法）:一般会控制在0.7左右

#### 线性探测的插入Insert
先看Find里面有没有Key,有的话就结束插入。

但是在映射key的哈希值时，如果是int那就好办,如果是string类型怎么办呐？

所以这里我们要写写一个哈希仿函数, 因为string比较常用，我们把string特化一下

```cpp
template <class K> 
struct HashFunc {
	size_t operator()(const K& key) {
		return (size_t)key;
	}
};

template<>
struct HashFunc <string> {
	size_t operator ()(const string& s) {
		size_t hash = 0;
		for (auto e : s) {
			hash += e;
			hash *= 131;
		}
		return hash;
	}
};
```

现在我们就可以写Find函数了
```cpp
HashData<K, V>* Find(const K& key) {
    Hash hs;
    size_t hashi = hs(key) % _tables.size();
    while (_tables[hashi]._state != EMPTY) {
        if (_tables[hashi]._kv.first == key
            && _tables[hashi]._state == EXIST) {
            return &_tables[hashi];
        }
        hashi++;
        hashi %= _tables.size();
    }
    return nullptr;
}
```

再来写Insert插入,这里负载因子大于0.7就去扩容,保证不满,且不会死递归

```cpp
bool Insert(const pair<K, V> kv) {
	if (Find(kv.first))
		return false;
	if (_n * 10 / _tables.size() >= 7) { // 注意这里是size,而不是capacity
		HashTable<K, V, Hash> NewHT(_tables.size() * 2);
		for (auto& e : _tables)
			if (e._state == EXIST) // 存在才加入
				NewHT.Insert(e._kv); // 插入e._kv
		_tables.swap(NewHT._tables); // 交换tables即可
	}
	Hash hs;
	size_t hashi = hs(kv.first) % _tables.size();
	while (_tables[hashi]._state == EXIST) {
		hashi++;
		hashi %= _tables.size();
	}
	_tables[hashi]._kv = kv;
	_tables[hashi]._state = EXIST;
	++_n; // 最后记得++_n个数
	return true; // 最后记得返回true
}
```

#### 线性探测的删除Erase

Erase只需把存在的状态设置成DELETE
```cpp
bool Erase(const K& key) {
	HashData<K, V>* ret = Find(key);
	if (ret) {
		--_n;
		ret->_state = DELETE;
		return true;
	}
	else
		return false;
}
```
### 哈希桶解决哈希冲突

首先创造一个节点,为链表
```cpp
struct HashNode {
	HashNode<K, V>* _next;
	pair<K, V> _kv;
	HashNode(const pair<K, V> kv) //记得列表初始化
		:_next(nullptr)
		,_kv(kv)
	{}
};
```

创建一个Hash表，加上默认构造函数
```cpp
template<class K, class V>
struct HashTable {
	typedef HashNode Node;
	HashTable() {
		_tables.resize(10, nullptr);
	}

private:
	vector<Node*> _tables;
	size_t n = 0;
};
```

添加仿函数
```cpp
template <class K>
struct HashFunc {
	size_t operator()(const K& key) {
		return (size_t)key;
	}
};

template<>
struct HashFunc <string> {
	size_t operator ()(const string& s) {
		size_t hash = 0;
		for (auto e : s) {
			hash += e;
			hash *= 131;
		}
		return hash;
	}
};
template<class K, class V, class Hash = HashFunc<K>>
```

写一个Find函数，看tables里面有没有重复，这样可以防止后面再去查找是否含有这个元素

```cpp
Node* Find(const K& key) {
	Hash hs;
	size_t hashi = hs(key) % _tables.size();
	Node* cur = _tables[hashi];
	while (cur) {
		if (cur->_kv.first == key)
			return cur;
		cur = cur->_next;
	}
	return nullptr;
}
```

再写一个析构函数,保证可析构

```cpp
~HashTable() {
	for (int i = 0; i < _tables.size(); ++i) {
		Node* cur = _tables[i];
		while (cur) {
			Node* next = cur->_next;
			delete cur;
			cur = next;
		}
		_tables[i] = nullptr;
	}
}
```

#### 哈希桶的Insert插入函数
```cpp
bool Insert(const pair<K, V> kv) {
	if (Find(kv.first))
		return false;
	Hash hs;
	if (_n == _tables.size()) {
		vector<Node*> NewHT(_tables.size() * 2, nullptr);
		for (int i = 0; i < _tables.size(); ++i) {
			Node* cur = _tables[i];
			while (cur) {
				Node* next = cur->_next;
				size_t hashi = hs(cur->_kv.first) % NewHT.size(); // 需要hs一下first
				cur->_next = NewHT[hashi];//直接用cur来链接，不需要自己newnode
				NewHT[hashi] = cur;
				cur = next;
			}
			_tables[i] = nullptr;
		}
		_tables.swap(NewHT);
	}
	size_t hashi = hs(kv.first) % _tables.size();
	Node* newnode = new Node(kv);
	newnode->_next = _tables[hashi];
	_tables[hashi] = newnode;
	++_n; //不要忘记++_n
	return true;
}
```
#### 哈希桶的Erase删除函数
```cpp
bool Erase(const K& key) {
	Hash hs;
	size_t hashi = hs(key) % _tables.size();
	Node* prev = nullptr;
	Node* cur = _tables[hashi];
	while (cur) {
		if (cur->_kv.first == key) {
			if (prev)
				prev->_next = cur->_next;
			else
				_tables[hashi] = cur->_next;
			delete cur; // 记得delete cur和--_n
			--_n;
			return true;
		}
		prev = cur;
		cur = cur->_next;
	}
	return false;
}
```

## 哈希封装umap、set

根据红黑树封装map和set的时候，需要用一个红黑树来封装map和set

所以这里我们依然用一个哈希表来封装unordered_map、unordered_set

### 创建一个HashNode

```cpp
template<class T>
struct HashNode {
	HashNode<T>* _next;
	T _data;
	HashNode(const T data)
		:_next(nullptr)
		, _data(data)
	{}
};
```

### 创建一个__HTIterator

```cpp
template<class K, class T, class KeyOfT, class Hash>
struct __HTIterator {
	typedef HashNode<T> Node;
	typedef HashTable<K, T, KeyOfT, Hash> HT;
	typedef __HTIterator<K, T, KeyOfT, Hash> Self;

	Node* _node;
	HT* _ht;//HT*

	__HTIterator(Node* node, HT* ht) 
		:_node(node)
		,_ht(ht)
	{}
	T& operator*() {//T&
		return _node->_data;
	}
	Self& operator++() {
		if (_node->_next) {
			_node =  _node->_next;
		}
		else {
			Hash hs;
			KeyOfT kot;
			size_t hashi = hs(kot(cur->_data)) % _ht->_tables.size();//->
			hashi++;
			while (hashi < _ht->_tables.size()) {
				if (_ht->_tables[hashi]) {
					_node = _ht->_tables[hashi];
					break;
				}
				hashi++;
			}

			if (hashi == _ht->_tables.size())
				_node = nullptr;
		}
		return *this;
	}

	bool operator!=(const Self& s) {
		return _node != s._node;
	}
};
```
### 把HashTable用KeyOfT改一下

```cpp
template<class K, class T, class KeyOfT, class Hash>
struct HashTable {
	typedef HashNode<T> Node;
	public:
	typedef __HTIterator<K, T, KeyOfT, Hash> iterator;

	iterator begin() {
		for (int i = 0; i < _tables.size(); ++i) {
			if (_tables[i]) {
				return iterator(_tables[i], this);
			}
		}
		return end();
	}

	iterator end() {
		return iterator(nullptr, this);
	}

	HashTable() {
		_tables.resize(10, nullptr);
		_n = 0; // _n = 0 不要忘记
	}

	~HashTable() {
		for (int i = 0; i < _tables.size(); ++i) {
			Node* cur = _tables[i];
			while (cur) {
				Node* next = cur->_next;
				delete cur;
				cur = next;
			}
			_tables[i] = nullptr;
		}
	}

	bool Insert(const T& data) { //T&
		KeyOfT kot;
		if (Find(kot(data)))
			return false;
		Hash hs;
		if (_n == _tables.size()) {
			vector<Node*> NewHT(_tables.size() * 2, nullptr);
			for (int i = 0; i < _tables.size(); ++i) {
				Node* cur = _tables[i];
				while (cur) {
					Node* next = cur->_next;
					size_t hashi = hs(kot(cur->_data)) % NewHT.size(); // 需要hs一下first
					cur->_next = NewHT[hashi];//直接用cur来链接，不需要自己newnode
					NewHT[hashi] = cur;
					cur = next;
				}
				_tables[i] = nullptr;
			}
			_tables.swap(NewHT);
		}
		size_t hashi = hs(kot(data)) % _tables.size();
		Node* newnode = new Node(data);
		newnode->_next = _tables[hashi];
		_tables[hashi] = newnode;
		++_n;
		return true;
	}

	Node* Find(const K& key) {
		KeyOfT kot;
		Hash hs;
		size_t hashi = hs(key) % _tables.size();
		Node* cur = _tables[hashi];
		while (cur) {
			if (kot(cur->_data) == key)
				return cur;
			cur = cur->_next;
		}
		return nullptr;
	}

	bool Erase(const K& key) {
		KeyOfT kot;
		Hash hs;
		size_t hashi = hs(key) % _tables.size();
		Node* prev = nullptr;
		Node* cur = _tables[hashi];
		while (cur) {
			if (kot(cur->_data) == key) {
				if (prev)
					prev->_next = cur->_next;
				else
					_tables[hashi] = cur->_next;
				delete cur; // 记得delete cur和--_n
				--_n;
				return true;
			}
			prev = cur;
			cur = cur->_next;
		}
		return false;
	}
private:
	vector<Node*> _tables;
	size_t _n;
};
```

### 添加一个MyOrdered_set.h

```cpp
#include "HashTable.h"

namespace lkt {
	template <class K, class Hash = HashFunc<K>> 
	class unordered_set {
		struct SetKeyOfT
		{
			const K& operator()(const K& key) {
				return key;
			}
		};
		typedef typename lkt2::HashTable<K, const K, SetKeyOfT, Hash>::iterator iterator;
		iterator begin() {
			return	_ht.begin();
		}

		iterator end() {
			return _ht.end();
		}

		bool Insert(const K& key) {
			return _ht.Insert(key);
		}
	private:
		lkt2::HashTable<K, V, SetKeyOfT, Hash> _ht;
	};
}

```

在__HTIterator前添加，前置声明
```cpp
template<class K, class T, class KeyOfT, class Hash>
struct HashTable;
```

在HashTab里面友元一个
```cpp
template<class K, class T, class KeyOfT, class Hash>
friend struct __HTIterator;
```
### 添加一个MyOrdered_map.h
```cpp
#include "HashTable.h"
namespace lkt {
	template <class K, class V, class Hash = HashFunc<K>>
	class unordered_map {
		struct MapKeyOfT {
			const K& operator()(const pair<K, V>& kv) {
				return kv.first;
			}
		};
	public:
		typedef typename lkt2::HashTable<K, pair<const K, V>, MapKeyOfT, Hash>::iterator iterator;
		iterator begin() {
			return _ht.begin();
		}

		iterator end() {
			return _ht.end();
		}

		bool Insert(const pair<K, V> kv) {
			return _ht.Insert(kv);
		}

	private:
		lkt2::HashTable<K, pair<const K, V>, MapKeyOfT, Hash> _ht;
	};
}
```



## HashTable汇总代码
```cpp
#pragma once

#include <iostream>
using namespace std;
#include <string>
#include <vector>

template <class K>
struct HashFunc {
	size_t operator()(const K& key) {
		return (size_t)key;
	}
};

template<>
struct HashFunc <string> {
	size_t operator ()(const string& s) {
		size_t hash = 0;
		for (auto e : s) {
			hash += e;
			hash *= 131;
		}
		return hash;
	}
};

namespace lkt2 {

	template<class T>
	struct HashNode {
		HashNode<T>* _next;
		T _data;
		HashNode(const T data)
			:_next(nullptr)
			, _data(data)
		{}
	};

	template<class K, class T, class KeyOfT, class Hash>
	struct HashTable;



	template<class K, class T, class KeyOfT, class Hash>
	struct __HTIterator {
		typedef HashNode<T> Node;
		typedef HashTable<K, T, KeyOfT, Hash> HT;
		typedef __HTIterator<K, T, KeyOfT, Hash> Self;
		__HTIterator(Node* node, HT* ht) 
			:_node(node)
			,_ht(ht)
		{}
		T& operator*() {//T&
			return _node->_data;
		}
		Self& operator++() {
			if (_node->_next) {
				_node =  _node->_next;
			}
			else {
				Hash hs;
				KeyOfT kot;
				size_t hashi = hs(kot(_node->_data)) % _ht->_tables.size();//->
				hashi++;
				while (hashi < _ht->_tables.size()) {
					if (_ht->_tables[hashi]) {
						_node = _ht->_tables[hashi];
						break;
					}
					hashi++;
				}

				if (hashi == _ht->_tables.size())
					_node = nullptr;
			}
			return *this;
		}

		bool operator!=(const Self& s) {
			return _node != s._node;
		}
	private:
		Node* _node;
		HT* _ht;//HT*
	};

	template<class K, class T, class KeyOfT, class Hash>
	struct HashTable {
		template<class K, class T, class KeyOfT, class Hash>
		friend struct __HTIterator;
		typedef HashNode<T> Node;
	public:
		typedef __HTIterator<K, T, KeyOfT, Hash> iterator;

		iterator begin() {
			for (int i = 0; i < _tables.size(); ++i) {
				if (_tables[i]) {
					return iterator(_tables[i], this);
				}
			}
			return end();
		}

		iterator end() {
			return iterator(nullptr, this);
		}

		HashTable() {
			_tables.resize(10, nullptr);
			_n = 0; // _n = 0 不要忘记
		}

		~HashTable() {
			for (int i = 0; i < _tables.size(); ++i) {
				Node* cur = _tables[i];
				while (cur) {
					Node* next = cur->_next;
					delete cur;
					cur = next;
				}
				_tables[i] = nullptr;
			}
		}

		bool Insert(const T& data) { //T&
			KeyOfT kot;
			if (Find(kot(data)))
				return false;
			Hash hs;
			if (_n == _tables.size()) {
				vector<Node*> NewHT(_tables.size() * 2, nullptr);
				for (int i = 0; i < _tables.size(); ++i) {
					Node* cur = _tables[i];
					while (cur) {
						Node* next = cur->_next;
						size_t hashi = hs(kot(cur->_data)) % NewHT.size(); // 需要hs一下first
						cur->_next = NewHT[hashi];//直接用cur来链接，不需要自己newnode
						NewHT[hashi] = cur;
						cur = next;
					}
					_tables[i] = nullptr;
				}
				_tables.swap(NewHT);
			}
			size_t hashi = hs(kot(data)) % _tables.size();
			Node* newnode = new Node(data);
			newnode->_next = _tables[hashi];
			_tables[hashi] = newnode;
			++_n;
			return true;
		}

		Node* Find(const K& key) {
			KeyOfT kot;
			Hash hs;
			size_t hashi = hs(key) % _tables.size();
			Node* cur = _tables[hashi];
			while (cur) {
				if (kot(cur->_data) == key)
					return cur;
				cur = cur->_next;
			}
			return nullptr;
		}

		bool Erase(const K& key) {
			KeyOfT kot;
			Hash hs;
			size_t hashi = hs(key) % _tables.size();
			Node* prev = nullptr;
			Node* cur = _tables[hashi];
			while (cur) {
				if (kot(cur->_data) == key) {
					if (prev)
						prev->_next = cur->_next;
					else
						_tables[hashi] = cur->_next;
					delete cur; // 记得delete cur和--_n
					--_n;
					return true;
				}
				prev = cur;
				cur = cur->_next;
			}
			return false;
		}
	private:
		vector<Node*> _tables;
		size_t _n;
	};
}
```


