# 单调栈单调队列

**单调栈**，就是满足栈中元素是单调递增或递减的线性数据结构。
利用栈先进后出的特点和其中元素的单调性可以解决许多问题

## 求下一个更大的数
给你 n 个整数 a1,a2,…,an，请问从每个数字往后看，第一个比它大的数字的下标是多少？如果后面没有比它大的数字，则输出 0。

方法一：从左向右,建立单调递减栈
```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
ll n, top, s[200001], a[200001], ans[200001];
int main() {
	scanf("%lld", &n);
	for (ll i = 1; i <= n; ++i)
		scanf("%lld", &a[i]);
	for (ll i = 1; i <= n; ++i) {
		while (top && a[i] > a[s[top]]) {
			ans[s[top]] = i;
			--top;
		}
		s[++top] = i;
	}
	for (ll i = 1; i <= top; ++i)
		ans[s[i]] = 0;
	for (ll i = 1; i <= n; ++i)
		printf("%lld ", ans[i]);
	printf("\n");
	return 0;
}
```

方法二：从右往左,建立单调递减栈
```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
ll n, top, s[200001], a[200001], ans[200001];
int main() {
	scanf("%lld", &n);
	for (ll i = 1; i <= n; ++i)
		scanf("%lld", &a[i]);
	for (ll i = n; i; --i) {
		while (top && a[i] >= a[s[top]])
			--top;
		if (top)
			ans[i] = s[top];
		else
			ans[i] = 0;
		s[++top] = i;
	} 
	for (ll i = 1; i <= n; ++i)
		printf("%lld ", ans[i]);
	printf("\n");
	return 0;
}
```

## 最大矩形面积

有一张 n列的网格图，每列有一些格子被小蜗从底向上涂了色。现在给你每一列被涂色的格子的高度 ai，请你求出被涂色的格子组成的最大矩形的面积。

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
ll n, top, l[200001], a[200001], r[200001], s[200001];
int main() {
	scanf("%lld", &n);
	for (ll i = 1; i <= n; ++i)
		scanf("%lld", &a[i]);
	for (ll i = 1; i <= n; ++i) {
		while (top && a[i] <= a[s[top]])
			--top;
		if (top)
			l[i] = s[top];
		else
			l[i] = 0;
		s[++top] = i;
	}
	top = 0;
	for (ll i = n; i; --i) {
		while (top && a[i] <= a[s[top]])
			--top;
		if (top)
			r[i] = s[top];
		else
			r[i] = n + 1;
		s[++top] = i;
	}
	ll ans = 0;
	for (ll i = 1; i <= n; ++i)
		ans = max(ans, (r[i] - l[i] - 1) * a[i]);
	printf("%lld\n", ans);
	return 0;
}
```

## 数对统计
给你 n个数字 a1,a2,…,an，这些数字各不相同。询问共有多少对数字 (i,j) (1≤i<j≤n)，满足 ai 到 aj 中没有数字比 ai 或 aj 大。 即对所有位置 k (i<k<j)，满足 ak<min(ai,aj)。
```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
ll n, top, a[200001], s[200001];
int main() {
	scanf("%lld", &n);
	for (ll i = 1; i <= n; ++i)
		scanf("%lld", &a[i]);
	ll ans = 0;
	for (ll i = 1; i <= n; ++i) {
		while (top && a[i] > a[s[top]])
			--top, ++ans;
		if (top)
			++ans;
		s[++top] = i;
	}
	printf("%lld\n", ans);
	return 0;
}
```

**单调队列**:就是满足队列中元素是单调递增或递减的线性数据结构

## 动态区间最大数

给你 n个数字 a1,a2,...,an，请从左到右输出每个长度为 m 的数列段内的最大数。这些数列段分别为 [1,m],[2,m+1],...,[n−m+1,n]，共 n−m+1 个。

解析:维护一个单调递减的队列即可

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
ll n, m, a[200001], q[200001], front = 1, rear = 0;
int main() {
	scanf("%lld%lld", &n, &m);
	for (ll i = 1; i <= n; ++i)
		scanf("%lld", &a[i]);
	for (ll i = 1; i <= n; ++i) {
		while (front <= rear && a[q[rear]] <= a[i])
			--rear;
		q[++rear] = i;
		if (q[front] < i - m + 1)
			++front;
		if (i >= m)
			printf("%lld ", a[q[front]]);
	}
	return 0;
}
```

##最大连续区间和

给你 n 个数字 a1,a2,...,an 和两个整数 l,r，你需要在 a1,a2,...,an 中选出一段连续的区间满足区间长度 ∈[l,r] 并且区间内的数字的和最大。请求出满足条件的最大连续区间和。

解析: i固定时,维护一个区间最大值,再利用前缀和求最大.

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
ll n, l, r, front = 1, rear = 0, a[200001], s[200001], q[200001];
int main() {
	scanf("%lld%lld%lld", &n, &l, &r);
	for (ll i = 1; i <= n; ++i)
		scanf("%lld", &a[i]);
	for (ll i = 1; i <= n; ++i)
		s[i] = s[i - 1] + a[i];
	ll x = l, ans = - 1 << 30;
	for (ll i = 1; i + l - 1 <= n; ++i) {
		while (x <= i + r - 1 && x <= n) {
			while (front <= rear && s[q[rear]] <= s[x]) 
				--rear;
			q[++rear] = x;
			++x;
		}
		if (q[front] < i + l - 1)
			++front;
		ans = max(ans, s[q[front]] - s[i - 1]);
	}
	printf("%lld\n", ans);
	return 0;
} 
```