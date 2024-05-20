# 进程间通信

两个进程之间,可以进行"数据"的直接传递吗?

不能.

为什么?

+ 数据传输：一个进程需要将它的数据发送给另一个进程
+ 资源共享：多个进程之间共享同样的资源。
+ 通知事件：一个进程需要向另一个或一组进程发送消息，通知它（它们）发生了某种事件（如进程终止
时要通知父进程）。
+ 进程控制：有些进程希望完全控制另一个进程的执行（如Debug进程），此时控制进程希望能够拦截另
一个进程的所有陷入和异常，并能够及时知道它的状态改变。
+ 需要多个进程协同,共同完成一些事情

是什么?

一个进程把自己的数据,能够交给另一个进程

怎么办?

a. 一般规律

1. 交换数据的空间
2. 不能由通信双方任何一个实现
3. 进程间通信的本质, 先让不同的进程, 看到同一份资源(一般都是OS提供)

b.具体做法

OS提供的"空间"有不同的样式,就决定了有不同的通信方式

1. 管道(匿名和命名)
2. 共享内存
3. 消息队列
4. 信号量

## 管道

一个以w打开文件,一个以r打开,会有两个文件描述符对象，用同一块内存.(先把文件加载到内存，才能读写文件)

创建一个子进程时, 父进程的进程描述符表也**需要**创建.父进程打开的文件,子进程**不需要**创建.

基于文件的，让不同进程看到同一份资源的方式，叫做管道.(管道只能被设计成单向通信)

struct file是允许多个进程通过指针指向！

为什么父进程最开始，就要按照rw，打开同一个文件呢？

答:一个人读，一个人写，为了支持我们进行管道通信,系统调用pipe()

int pipe(int fildes[2]);

fildes:输出型参数，得到两个fd, r, w

不需要向磁盘中刷新&&磁盘中并不存在的文件

内存级的文件,匿名文件(管道)

匿名管道:如何做到让不同的进程看到同一份资源呢?

创建子进程,子进程会进程父进程的属性信息.

匿名管道:可以(只能)进行具有血缘关系的进程, 进行进程间通信.

代码:

1.验证代码

查看进程:

while :; do ps ajx | head -1 && ps ajx | grep testpipe | grep -v grep; sleep 1; done 

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/types.h>
#include <sys/wait.h>

void writer(int wfd) {
	const char *str = "hello father, i am child";
	char buffer[128];
	int cnt = 0;
	pid_t pid = getpid(); 
	while (1) {
		snprintf(buffer, sizeof(buffer), "message = %s, pid = %d, cnt = %d\n", str, pid, cnt);
		write(wfd, buffer, strlen(buffer));
		cnt++;
		sleep(1);
	}
}

void reader(int rfd) {
	char buffer[1024];
	while (1) {
		ssize_t n = read(rfd, buffer, sizeof(buffer) - 1);
		(void)n;
		printf("father get a message: %s", buffer);
	}
}

int main() {
	int pipefd[2];
	int n = pipe(pipefd);
	if (n < 0)
		return 1;
	printf("pipefd[0]: %d, pipefd[1]: %d\n", pipefd[0], pipefd[1]);
	pid_t id = fork();
	if (id == 0) {
		// child:w
		close(pipefd[0]);
		writer(pipefd[1]);
		exit(0);
	}
	// father
    
	close(pipefd[1]);
	reader(pipefd[0]);
	wait(NULL);
	return 0; 
}
```
a.4种情况

1. 管道内部没有数据 && 子进程不关闭自己的写端文件fd, 读端(父)就要阻塞等待,直到pipe有数据
2. 管道内部被写满 && 父进程(读端)不关闭自己的fd,写端(子)写满之后,就要阻塞等待
3. 对于写端而言: 不写了 && 关闭了pipe,读端会将pipe中的数据读完,最后就会读到返回值为0，表示读结束，类似读到文件的结尾
4. 读端不读 && 关闭,写端再写,OS会直接终止写入的进程(子进程), 通过信号 13) SIGPIPE 信号杀掉

如何验证退出信号是13?
```cpp
//wait(NULL);
int status = 0;
pid_t rid = waitpid(id, &status, 0);
if (rid == id)
	printf("exit code: %d, exit signal: %d\n", WEXITSTATUS(status), status&0x7F);
```


b.5种特性

1. 自带同步机制
2. 血缘关系进程进行通信,常见与父子
3. pipe是面向字节流的
4. 父子退出,管道自动释放,文件的生命周期是随进程的
5. 管道只能单向通信,半双工的一种特殊情况

PIPE_BUF(4096byte):

当要写入的数据量不大于PIPE_BUF时，linux将保证写入的原子性。
当要写入的数据量大于PIPE_BUF时，linux将不再保证写入的原子性。

2.功能代码





