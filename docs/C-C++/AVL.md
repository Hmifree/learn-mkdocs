# AVL树
## 一、AVL树

### （一）概念

二叉搜索树虽可以缩短查找的效率，但如果数据有序或接近有序二叉搜索树将退化为单支树，查

找元素相当于在顺序表中搜索元素，效率低下。因此，两位俄罗斯的数学家G.M.Adelson-Velskii
和E.M.Landis在1962年发明了一种解决上述问题的方法：当向二叉搜索树中插入新结点后，如果能保证每个结点的左右
子树高度之差的绝对值不超过1(需要对树中的结点进行调整)，即可降低树的高度，从而减少平均搜索长度。

一棵AVL树或者是空树，或者是具有以下性质的二叉搜索树：

1.它的左右子树都是AVL树

2.左右子树高度之差(简称平衡因子, 假定右减左)的绝对值不超过1(-1/0/1)

问: 为什么左右子树的高度差不能只设置成0?

答: 在只有两个节点的情况下，无法使得两棵树的子树平衡因子都为0.

补: 树的平衡因子不是必须要的，但是在这里我们用平衡因子来控制树.

### （二）定义

如下定义一个AVL树节点的定义，这里我们是用的一个三叉树。
```cpp
template<class K, class V>
struct AVLTreeNode {
	AVLTreeNode<K, V>* _left;
	AVLTreeNode<K, V>* _right;
	AVLTreeNode<K, V>* _parent;
	int bf;
	pair<K, V> _kv;
	AVLTreeNode(const pair<K, V> kv)
		:_left(nullptr)
		, _right(nullptr)
		, _parent(nullptr)
		, _bf(0)
		, _kv(kv)
	{}
};
```

## 二、AVL树的插入

AVL树就是在二叉搜索树的基础上引入了平衡因子，因此AVL树也可以看成是二叉搜索树。那么
AVL树的插入过程可以分为两步：
1. 按照二叉搜索树的方式插入新节点
2. 调整节点的平衡因子
```cpp
template<class K, class V> 
struct AVLTree {
	typedef AVLTreeNode<K, V> Node;
public:
	bool Insert(const pair<K, V>& kv) {
		//1. 按照二叉搜索树的方式插入新节点
		//2. 调整节点的平衡因子
		
	}
private:
	Node* _root = nullptr;
};
```

### （一）先按照二叉搜索树的规则将节点插入到AVL树中

```cpp
bool Insert(const pair<K, V>& kv) {
	//1. 按照二叉搜索树的方式插入新节点
	//2. 调整节点的平衡因子
	if (_root == nullptr) {
		_root = new Node(kv);
		return true;
	}
	Node* parent = nullptr;
	Node* cur = _root;
	while (cur) {
		if (cur->_kv.first > kv.first) {
			parent = cur;
			cur = cur->_left;
		}
		else if (cur->_kv.first < kv.first) {
			parent = cur;
			cur = cur->_right;
		}
		else {
			return false;
		}
	}
	cur = new Node(kv);
	if (parent->_kv.first > kv.first)
		parent->_left = cur;
	else
		parent->_right = cur;
	cur->_parent = parent;
}
```

### （二）调节平衡因子

问: 插入节点回影响那些平衡因子呢？

答: 新增节点的部分祖先

更新原则: cur更新到root位置, 结束。

cur是p的左边,p->_bf--

cur是p的右边,p->_bf++

1.更新后,p->_bf == 0,p所在的子树高度不变,不会影响爷爷，说明更新前,p的_bf是1或者-1

p在矮的节点那边插入了节点,左右均衡了,p的高度不变,不会影响爷爷.

2.更新后,p->_bf == 1 / -1,p所在的子树的高度变了,会影响爷爷,说明更新前,p的_bf是0

p的有一边插入,p变的不均衡,但是不违反规则,p的高度变了,会影响爷爷.

3.更新后p->_bf == 2 / -2,说明p所在的子树违反了平衡规则,需要进行处理->旋转

(让p所在子树高度回到插入之前,不会对上层的bf有影响)

```cpp
while (parent) {
	if (cur == parent->_left)
		parent->_bf--;
	else
		parent->_bf++;
	if (parent->_bf == 0)
		break;
	else if (parent->_bf == 1 || parent->_bf == -1) {
		cur = cur->_parent;
		parent = parent->_parent;
	}
	else if (parent->_bf == 2 || parent->_bf == -2) {
		// 旋转

	}
	else
		assert(false);
}
```

### （三）旋转

1.左旋转（当右边一条直线且bf == 2）
```cpp
void RotateL(Node* parent) {
	Node* subR = parent->_right; // parent的右节点
	Node* subRL = subR->_left;   // parent的右节点的左节点
	parent->_right = subRL;      
	if (subRL) // 判断subRL不为空
		subRL->_parent = parent;
	subR->_left = parent;
	Node ppnode = parent->_parent; // parent的父亲节点来判断subR的父亲节点
	parent->_parent = subR;
	if (parent == _root) {
		_root = subR;
		subR->_parent = nullptr;
	}
	else {
		if (parent == ppnode->_left)
			ppnode->_left = subR;
		else
			ppnode->_right = subR;
		subR->_parent = ppnode;
	}
	parent->_bf = 0;
	subR->_bf = 0;
}
```

2.右旋转（类似左旋转,左边一条直线且bf=-2）
```cpp
void RotateR(Node* parent) {
	Node* subL = parent->_left;//parent的左节点
	Node* subLR = subL->_right;//parent的左节点的右节点
	parent->_left = subLR;
	if (subLR)
		subLR->_parent = parent;
	subL->_right = parent;
	Node* ppnode = parent->_parent; //parent的父亲节点来判断subL的父亲节点 
	parent->_parent = subL;
	if (parent == _root) { // 判断subL是否更新为根节点
		_root = subL;
		subL->_parent = nullptr;
	}
	else {
		if (ppnode->_left == parent)
			ppnode->_left = subL;
		else
			ppnode->_right = subL;
		subL->_parent = ppnode;
	}
	parent->_bf = 0;
	subL->_bf = 0;
}
```

3.左右旋转（先往左走，再往右走。一个bf为-2，一个bf为1。先左旋，再右旋，难点在于调节平衡因子（关键在于画图））

```cpp
void RotateLR(Node* parent) {
	Node* subL = parent->_left;
	Node* subLR = subL->_right;
	int bf = subLR->_bf;
	RotateL(parent->_left);
	RotateR(parent);
	if (bf == -1) {
		subLR->_bf = 0;
		subL->_bf = 0;
		parent->_bf = 1;
	}
	else if (bf == 1) {
		subLR->_bf = 0;
		subL->_bf = -1;
		parent->_bf = 0;
	}
	else if (bf == 0) {
		subLR->_bf = 0;
		subL->_bf = 0;
		parent->_bf = 0;
	}
	else
		assert(false);
}
```

4.右左旋转（先往右走，再往左走。一个bf为2，一个bf为-1。先右旋，再左旋，难点在于调节平衡因子（关键在于画图））

```cpp
void RotateRL(Node* parent) {
	Node* subR = parent->_right;
	Node* subRL = subR->_left;
	int bf = subRL->_bf;
	RotateR(parent->_right);
	RotateL(parent);
	if (bf == -1) {
		subRL->_bf = 0;
		subR->_bf = 1;
		parent->_bf = 0;
	}
	else if (bf == 1) {
		subRL->_bf = 0;
		subR->_bf = 0;
		parent->_bf = -1;
	}
	else if (bf == 0) { // 只有3个节点的情况
		subRL->_bf = 0;
		subR->_bf = 0;
		parent->_bf = 0;
	}
	else
		assert(false);
}
```

旋转代码：

```cpp
if (parent->_bf == 2 && cur->_bf == 1)
	RotateL(parent);
else if (parent->_bf == -2 && cur->_bf == -1)
	RotateR(parent);
else if (parent->_bf == -2 && cur->_bf == 1)
	RotateLR(parent);
else
	RotateRL(parent);
break;
```
5.验证是否为AVL树:

```cpp
bool _IsBalance(Node* root, int& height) {
	if (root == nullptr) {
		height = 0;
		return true;
	}
	int LeftHeight = 0, RightHeight = 0;
	if (!_IsBalance(root->_left, LeftHeight) ||
		!_IsBalance(root->_right, RightHeight)) {
		return false;
	}
	if (abs(LeftHeight - RightHeight) >= 2) {
		cout << root->_kv.first << "不平衡" << endl;
		return false;
	}
	if (RightHeight - LeftHeight != root->_bf) {
		cout << root->_kv.first << "平衡英子异常" << endl;
		return false;
	}
	height = max(LeftHeight, RightHeight) + 1;
	return true;
}

bool IsBalance() {
	int height = 0;
	return _IsBalance(_root, height);
}
```
代码汇总：
```cpp
#pragma once
#include <iostream>
#include <vector>
#include <assert.h>
using namespace std;
template<class K, class V>
struct AVLTreeNode {
	AVLTreeNode<K, V>* _left;
	AVLTreeNode<K, V>* _right;
	AVLTreeNode<K, V>* _parent;
	int _bf;
	pair<K, V> _kv;
	AVLTreeNode(const pair<K, V> kv)
		:_left(nullptr)
		, _right(nullptr)
		, _parent(nullptr)
		, _bf(0)
		, _kv(kv)
	{}
};

template<class K, class V> 
struct AVLTree {
	typedef AVLTreeNode<K, V> Node;
public:
	bool Insert(const pair<K, V>& kv) {
		//1. 按照二叉搜索树的方式插入新节点
		//2. 调整节点的平衡因子
		if (_root == nullptr) {
			_root = new Node(kv);
			return true;
		}
		Node* parent = nullptr;
		Node* cur = _root;
		while (cur) {
			if (cur->_kv.first > kv.first) {
				parent = cur;
				cur = cur->_left;
			}
			else if (cur->_kv.first < kv.first) {
				parent = cur;
				cur = cur->_right;
			}
			else {
				return false;
			}
		}
		cur = new Node(kv);
		if (parent->_kv.first > kv.first)
			parent->_left = cur;
		else
			parent->_right = cur;
		cur->_parent = parent;
		while (parent) {
			if (cur == parent->_left)
				parent->_bf--;
			else
				parent->_bf++;
			if (parent->_bf == 0)
				break;
			else if (parent->_bf == 1 || parent->_bf == -1) {
				cur = cur->_parent;
				parent = parent->_parent;
			}
			else if (parent->_bf == 2 || parent->_bf == -2) {
				// 旋转
				if (parent->_bf == 2 && cur->_bf == 1)
					RotateL(parent);
				else if (parent->_bf == -2 && cur->_bf == -1)
					RotateR(parent);
				else if (parent->_bf == -2 && cur->_bf == 1)
					RotateLR(parent);
				else
					RotateRL(parent);
				break;
			}
			else
				assert(false);
		}
	}
	void RotateL(Node* parent) {
		Node* subR = parent->_right; // parent的右节点
		Node* subRL = subR->_left;   // parent的右节点的左节点
		parent->_right = subRL;      
		if (subRL) // 判断subRL不为空
			subRL->_parent = parent;
		subR->_left = parent;
		Node* ppnode = parent->_parent; // parent的父亲节点来判断subR的父亲节点
		parent->_parent = subR;
		if (parent == _root) {
			_root = subR;
			subR->_parent = nullptr;
		}
		else {
			if (parent == ppnode->_left)
				ppnode->_left = subR;
			else
				ppnode->_right = subR;
			subR->_parent = ppnode;
		}
		parent->_bf = 0;
		subR->_bf = 0;
	}

	void RotateR(Node* parent) {
		Node* subL = parent->_left;//parent的左节点
		Node* subLR = subL->_right;//parent的左节点的右节点
		parent->_left = subLR;
		if (subLR)
			subLR->_parent = parent;
		subL->_right = parent;
		Node* ppnode = parent->_parent; //parent的父亲节点来判断subL的父亲节点 
		parent->_parent = subL;
		if (parent == _root) { // 判断subL是否更新为根节点
			_root = subL;
			subL->_parent = nullptr;
		}
		else {
			if (ppnode->_left == parent)
				ppnode->_left = subL;
			else
				ppnode->_right = subL;
			subL->_parent = ppnode;
		}
		parent->_bf = 0;
		subL->_bf = 0;
	}

	void RotateLR(Node* parent) {
		Node* subL = parent->_left;
		Node* subLR = subL->_right;
		int bf = subLR->_bf;
		RotateL(parent->_left);
		RotateR(parent);
		if (bf == -1) {
			subLR->_bf = 0;
			subL->_bf = 0;
			parent->_bf = 1;
		}
		else if (bf == 1) {
			subLR->_bf = 0;
			subL->_bf = -1;
			parent->_bf = 0;
		}
		else if (bf == 0) {
			subLR->_bf = 0;
			subL->_bf = 0;
			parent->_bf = 0;
		}
		else
			assert(false);
	}

	void RotateRL(Node* parent) {
		Node* subR = parent->_right;
		Node* subRL = subR->_left;
		int bf = subRL->_bf;
		RotateR(parent->_right);
		RotateL(parent);
		if (bf == -1) {
			subRL->_bf = 0;
			subR->_bf = 1;
			parent->_bf = 0;
		}
		else if (bf == 1) {
			subRL->_bf = 0;
			subR->_bf = 0;
			parent->_bf = -1;
		}
		else if (bf == 0) { // 只有3个节点的情况
			subRL->_bf = 0;
			subR->_bf = 0;
			parent->_bf = 0;
		}
		else
			assert(false);
	}

	bool _IsBalance(Node* root, int& height) {
		if (root == nullptr) {
			height = 0;
			return true;
		}
		int LeftHeight = 0, RightHeight = 0;
		if (!_IsBalance(root->_left, LeftHeight) ||
			!_IsBalance(root->_right, RightHeight)) {
			return false;
		}
		if (abs(LeftHeight - RightHeight) >= 2) {
			cout << root->_kv.first << "不平衡" << endl;
			return false;
		}
		if (RightHeight - LeftHeight != root->_bf) {
			cout << root->_kv.first << "平衡英子异常" << endl;
			return false;
		}
		height = max(LeftHeight, RightHeight) + 1;
		return true;
	}

	bool IsBalance() {
		int height = 0;
		return _IsBalance(_root, height);
	}
private:
	Node* _root = nullptr;
};

void TestAVLTree()
{
	const int N = 1000000;
	vector<int> v;
	v.reserve(N);
	srand(time(0));

	for (size_t i = 0; i < N; i++)
	{
		v.push_back(rand() + i);
		//cout << v.back() << endl;
	}

	AVLTree<int, int> t;
	for (auto e : v)
	{
		t.Insert(make_pair(e, e));
		//cout << "Insert:" << e << "->" << t.IsBalance() << endl;
	}
	cout << t.IsBalance() << endl;
}
```