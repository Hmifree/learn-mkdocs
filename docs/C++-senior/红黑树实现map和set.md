# 红黑树实现map和set

为了运用红黑树的一套工具，所以我们把红黑树设置为3个模版

第一个表示key，第二个表示key/key_value，第三个表示仿函数来解决key直接的比较

同时添加了一个迭代器iterator来作为迭代器

BRTree.h:
```cpp
#pragma once
#include <iostream>
#include <vector>
#include <assert.h>
using namespace std;
enum color {
	RED,
	BLACK
};

template<class T>
struct RBTreeNode {
	RBTreeNode<T>* _left;
	RBTreeNode<T>* _right;
	RBTreeNode<T>* _parent;
	T _data;
	color _col;
	RBTreeNode(const T& data)
		:_left(nullptr)
		, _right(nullptr)
		, _parent(nullptr)
		, _data(data)
		, _col(RED)
	{}
};

template<class T, class Ptr, class Ref> 
struct RBTreeIterator {
	typedef RBTreeNode<T> Node;
	typedef RBTreeIterator<T, Ptr, Ref> Self;
	Node* _node;
	RBTreeIterator(Node* node)
		:_node(node)
	{}
	
	Ptr operator->() {
		return &_node->_data;
	}

	Ref operator*() {
		return _node->_data;
	}

	Self& operator++() {
		if (_node->_right) {
			Node* SubLeft = _node->_right;
			while (SubLeft->_left)
				SubLeft = SubLeft->_left;
			_node = SubLeft;
		}
		else {
			Node* cur = _node;
			Node* parent = cur->_parent;
			while (parent && cur == parent->_right) {
				cur = parent;
				parent = cur->_parent;
			}
			_node = parent;
		}
		return *this;
	}

	Self& operator--() {
		if (_node->_left) {
			Node* SubRight = _node->_left;
			while (SubRight->_right)
				SubRight = SubRight->_right;
			_node = SubRight;
		}
		else {
			Node* cur = _node;
			Node* parent = _node->_parent;
			while (parent && cur == parent->_left) {
				cur = parent;
				parent = parent->_parent;
			}
			_node = parent;
		}
		return *this;
	}

	bool operator!=(const Self& s) {
		return _node != s._node;
	}

	bool operator==(const Self& s) {
		return _node == s._node;
	}
};

template<class K, class T, class KeyOfT>
struct RBTree {
public:
	typedef RBTreeNode<T> Node;
	typedef RBTreeIterator<T, T*, T&> iterator;
	typedef RBTreeIterator<T, const T*, const T&> const_iterator;
	const_iterator begin() const {
		Node* SubLeft = _root;
		while (SubLeft && SubLeft->_left)
			SubLeft = SubLeft->_left;
		return const_iterator(SubLeft);
	}

	const_iterator end() const {
		const_iterator(nullptr);
	}

	iterator begin() {
		Node* SubLeft = _root;
		while (SubLeft && SubLeft->_left)
			SubLeft = SubLeft->_left;
		return iterator(SubLeft);
	}

	iterator end() {
		return iterator(nullptr);
	}

	iterator Find(const K& key) {
		KeyOfT kot;
		Node* cur = _root;
		while (cur) {
			if (kot(cur->_data) < key)
				cur = cur->_right;
			else if (kot(cur->_data) > key)
				cur = cur->_left;
			else
				return iterator(cur);
		}
		return end();
	}

	pair<iterator, bool> Insert(const T& data) {
		if (_root == nullptr) {
			_root = new Node(data);
			_root->_col = BLACK;
			return make_pair(iterator(_root), true);
		}
		KeyOfT kot;
		Node* cur = _root;
		Node* parent = nullptr;
		while (cur) {
			if (kot(cur->_data) > kot(data)) {
				parent = cur;
				cur = cur->_left;
			}
			else if (kot(cur->_data) < kot(data)) {
				parent = cur;
				cur = cur->_right;
			}
			else
				return make_pair(iterator(cur), false);
		}
		cur = new Node(data);
		Node* newnode = cur;
		if (kot(parent->_data) < kot(data))
			parent->_right = cur;
		else
			parent->_left = cur;
		cur->_parent = parent;
		while (parent && parent->_col == RED) {
			Node* grandfather = parent->_parent;
			if (parent == grandfather->_left) {
				Node* uncle = grandfather->_right;
				if (uncle && uncle->_col == RED) {
					// 情况一：uncle存在且为红
					parent->_col = uncle->_col = BLACK;
					grandfather->_col = RED;
					cur = grandfather;
					parent = cur->_parent;
				}
				else {
					//情况二：u不存在或者u为黑
					if (parent->_left == cur) {
						//     g
						//   p   u
						// c
						RotateR(grandfather);
						parent->_col = BLACK;
						grandfather->_col = RED;
					}
					else {
						//   g
						// p   u
						//   c
						RotateL(parent);
						RotateR(grandfather);
						cur->_col = BLACK;
						grandfather->_col = RED;
					}
					break;
				}
			}
			else {
				Node* uncle = grandfather->_left;
				if (uncle && uncle->_col == RED) {
					// 情况一：uncle存在且为红
					parent->_col = uncle->_col = BLACK;
					grandfather->_col = RED;
					cur = grandfather;
					parent = cur->_parent;
				}
				else {
					//情况二：u不存在或者u为黑
					if (parent->_right == cur) {
						//   g
						// u   p
						//       c
						RotateL(grandfather);
						parent->_col = BLACK;
						grandfather->_col = RED;
					}
					else {
						//   g
						// u   p
						//   c
						RotateR(parent);
						RotateL(grandfather);
						cur->_col = BLACK;
						grandfather->_col = RED;
					}
					break;
				}
			}
		}
		_root->_col = BLACK;
		return make_pair(iterator(newnode), false);
	}

	void RotateL(Node* parent)
	{

		Node* subR = parent->_right;
		Node* subRL = subR->_left;

		parent->_right = subRL;
		if (subRL)
			subRL->_parent = parent;

		subR->_left = parent;
		Node* ppnode = parent->_parent;
		parent->_parent = subR;

		if (parent == _root)
		{
			_root = subR;
			subR->_parent = nullptr;
		}
		else
		{
			if (ppnode->_left == parent)
			{
				ppnode->_left = subR;
			}
			else
			{
				ppnode->_right = subR;
			}
			subR->_parent = ppnode;
		}
	}

	void RotateR(Node* parent)
	{
		Node* subL = parent->_left;
		Node* subLR = subL->_right;

		parent->_left = subLR;
		if (subLR)
			subLR->_parent = parent;

		subL->_right = parent;

		Node* ppnode = parent->_parent;
		parent->_parent = subL;

		if (parent == _root)
		{
			_root = subL;
			subL->_parent = nullptr;
		}
		else
		{
			if (ppnode->_left == parent)
			{
				ppnode->_left = subL;
			}
			else
			{
				ppnode->_right = subL;
			}
			subL->_parent = ppnode;
		}
	}

	bool Check(Node* root, int BlackNum, int RefBlackNum) {
		if (root == nullptr) {
			if (BlackNum != RefBlackNum) {
				cout << "黑色个数不相等" << endl;
				return false;
			}
			return true;
		}

		if (root->_col == BLACK)
			BlackNum++;

		if (root->_col == RED && root->_parent->_col == RED) {
			cout << "出现连续红色节点" << endl;
			return false;
		}

		return Check(root->_left, BlackNum, RefBlackNum)
			&& Check(root->_right, BlackNum, RefBlackNum);
	}

	bool IsBalance() {
		if (_root && _root->_col == RED)
			return false;
		int RefBlackNum = 0;
		Node* cur = _root;
		while (cur) {
			if (cur->_col == BLACK)
				RefBlackNum++;
			cur = cur->_left;
		}
		return Check(_root, 0, RefBlackNum);
	}

private:
	Node* _root = nullptr;
};

```
MyMap.h:
```cpp
#pragma once
#include"RBTree.h"
namespace lkt {
	template<class K, class V>
	class map {
		struct MapKeyOfT {
			const K& operator()(const pair<K, V>& kv) {
				return kv.first;
			}
		};
	public:
		typedef typename RBTree<K, pair<const K, V>, MapKeyOfT>::iterator iterator;
		typedef typename RBTree<K, pair<const K, V>, MapKeyOfT>::const_iterator const_iterator;
		iterator begin() {
			return _t.begin();
		}

		iterator end() {
			return _t.end();
		}

		pair<iterator, bool> insert(const pair<K, V>& kv) {
			return _t.Insert(kv);
		}

		iterator find(const K& key) {
			return _t.Find(key);
		}

		V& operator[] (const K& key) {
			pair<iterator, bool> ret = insert(make_pair(key, V()));
			return ret.first->second;
		}
	private:
		RBTree<K, pair<const K, V>, MapKeyOfT> _t;
	};

	void TestMap() {
		map<int, int> m;
		int a[] = { 4, 2, 6, 1, 3, 5, 15, 7, 16, 14 };
		for (auto e : a)
		{
			m.insert(make_pair(e, e));
		}

		map<int, int>::iterator it = m.begin();
		while (it != m.end())
		{
			//it->first += 100;
			it->second += 100;

			cout << it->first << ":" << it->second << endl;
			++it;
		}
		cout << endl;

	}
}

```

MySet.h:
```cpp
#pragma once
#include "RBTree.h"
namespace lkt {
	template<class K>
	class set {
		struct SetKeyOfT {
			const K& operator() (const K& key) {
				return key;
			}
		};
	public:

		typedef typename RBTree<K, const K, SetKeyOfT>::iterator iterator;
		typedef typename RBTree<K, const K, SetKeyOfT>::const_iterator const_iterator;
	
		iterator begin() {
			return _t.begin();
		}

		iterator end() {
			return _t.end();
		}

		pair<iterator, bool> insert(const K& key) {
			return _t.Insert(key);
		}

		iterator find(const K& key) {
			return _t.Find(key);
		}

	private:
		RBTree<K, const K, SetKeyOfT> _t;
	};

	void TestSet() {
		set<int> s;
		int a[] = { 4, 2, 6, 1, 3, 5, 15, 7, 16, 14 };
		for (auto e : a)
		{
			s.insert(e);
		}
		set<int>::iterator it = s.begin();
		while (it != s.end()) {
			cout << *it << " ";
			++it;
		}
		cout << endl;
	}
}

```

test.cpp:

```cpp
#define _CRT_SECURE_NO_WARNINGS 1
#include"MySet.h"
#include "MyMap.h"
int main() {
	lkt::TestSet();
	lkt::TestMap();
	return 0;
}
```