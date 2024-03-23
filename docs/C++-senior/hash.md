# 哈希
## 概念

存储的值和存储位置的映射的关联关系

## 常见的哈希函数：

1. 直接定址法--(常用)

取关键字的某个线性函数为散列地址：Hash（Key）= A*Key + B

优点：简单、均匀

缺点：需要事先知道关键字的分布情况

使用场景：适合查找比较小且连续的情况

2. 除留余数法--(常用)

设散列表中允许的地址数为m，取一个不大于m，但最接近或者等于m的质数p作为除数，

按照哈希函数：Hash(key) = key% p(p<=m),将关键码转换成哈希地址

3. 创建一个哈希表
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

再来写Insert插入

```cpp
bool Insert(const pair<K, V> kv) {
	if (Find(kv.first))
		return false;
	if (_n * 10 / _tables.capacity() >= 7) {
		HashTable<K, V, Hash> NewHT(_tables.size() * 2);
		for (auto& e : _tables)
			if (e._state == EXIST) // 存在才加入
				NewHT.Insert(e); 
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

#### Erase

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









