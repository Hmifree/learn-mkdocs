# C++的IO流

```cpp
int main() {
	string str;
	while (cin >> str) {
		cout << str << endl;
	}
	return 0;
}
```

问:上面cin的返回值是iostream,为什么ctrl+Z可以终止程序呢?

答:里面重载了operator bool(), operator bool调用时如果接收流失败，或者有结束标志，则返回false。