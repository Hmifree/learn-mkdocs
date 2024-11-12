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

```cpp
//processpool.cc

#include <iostream>
#include <string>
#include <unistd.h>
#include <vector>
#include <sys/types.h>
#include <sys/wait.h>
#include "task.hpp"
using namespace std;

enum
{
    UsageError = 1,
    ArgError,
    PipeError
};

void Usage(const std::string &proc)
{
    cout << "Usage:" << proc << " subprocess-num" << endl;
}

class Channel
{
public:
    Channel(int wfd, pid_t sub_id, const std::string &name)
        : _wfd(wfd), _sub_process_id(sub_id), _name(name)
    {
    }

    void PrintDebug()
    {
        cout << "wfd: " << _wfd << " ";
        cout << "id:" << _sub_process_id << " ";
        cout << "name" << _name << endl;
    }

    string name()
    {
        return _name;
    }

    int wfd()
    {
        return _wfd;
    }

    pid_t pid()
    {
        return _sub_process_id;
    }

    void Close()
    {
        close(_wfd);
    }

    ~Channel()
    {
    }

private:
    int _wfd;
    pid_t _sub_process_id;
    string _name;
};

class ProcessPool
{
public:
    ProcessPool(int sub_process_num)
        : _sub_process_num(sub_process_num)
    {
    }

    int CreateProcess(work_t work)
    {
        vector<int> fds;
        for (int number = 0; number < _sub_process_num; ++number)
        {
            int pipefd[2]{0};
            int n = pipe(pipefd);
            if (n < 0)
                return PipeError;
            pid_t id = fork();
            if (id == 0)
            {
                if (!fds.empty()) {
                    cout << "close w fd:";
                    for (auto fd : fds) 
                    {
                        close(fd);
                        cout << fd << " ";
                    }
                    cout << endl;
                }
                // child -> r
                close(pipefd[1]);
                // TODO
                dup2(pipefd[0], 0);
                work(pipefd[0]);
                exit(0);
            }
            string cname = "Channel-" + to_string(number);
            // father
            close(pipefd[0]);
            chaneels.push_back(Channel(pipefd[1], id, cname));

            // 把父进程的wfd保存
            fds.push_back(pipefd[1]);
        }
        return 0;
    }

    int NextChannel()
    {
        static int next = 0;
        int c = next++;
        next %= chaneels.size();
        return c;
    }

    void SendTaskCode(int index, uint32_t code)
    {
        cout << "send code: " << code << " to " << chaneels[index].name() << " sub process id: " << chaneels[index].pid() << endl;
        write(chaneels[index].wfd(), &code, sizeof(code));
    }

    void KillAll()
    {
        for (auto &chaneel : chaneels)
        {
            chaneel.Close();
            pid_t pid = chaneel.pid();
            pid_t rid = waitpid(pid, nullptr, 0);
            if (rid == pid)
            {
                cout << "wait pid process: " << pid << " success..." << endl;
            }
            cout << chaneel.name() << " close done" << " sub process quit now " << chaneel.pid() << endl;
        }
    }

    void Wait()
    {
        // for (auto &chaneel : chaneels) {
        //     pid_t pid = chaneel.pid();
        //     pid_t rid = waitpid(pid, nullptr, 0);
        //     if (rid == pid) {
        //         cout << "wait pid process: " << pid << " success..." << endl;
        //     }
        // }
    }

    void Debug()
    {
        for (auto &chaneel : chaneels)
        {
            chaneel.PrintDebug();
        }
    }

    ~ProcessPool()
    {
    }

private:
    int _sub_process_num;
    vector<Channel> chaneels;
};

void CtrlProcessPool(ProcessPool *processpool_ptr, int cnt)
{
    while (cnt)
    {
        // a. 选择一个进程和通道
        int chaneel = processpool_ptr->NextChannel();
        // cout << chaneel.name() << endl;

        // b.你要选择一个任务
        uint32_t code = NextTask();

        // c. 发送任务
        processpool_ptr->SendTaskCode(chaneel, code);

        sleep(1);
        cnt--;
    }
}

int main(int argc, char *argv[])
{
    if (argc != 2)
    {
        Usage(argv[0]);
        return UsageError;
    }
    int sub_process_num = std::stoi(argv[1]);
    if (sub_process_num <= 0)
        return ArgError;

    srand((uint64_t)time(nullptr));
    vector<Channel> chaneels;

    // 1. 创建腾信信号和子进程
    ProcessPool *processpool_ptr = new ProcessPool(sub_process_num);
    processpool_ptr->CreateProcess(worker);
    // processpool_ptr->Debug();

    // 2. 控制子进程
    CtrlProcessPool(processpool_ptr, 10);
    cout << "task: run done" << endl;
    // sleep(100);

    // 3.回收子进程:a.你怎么让所有的子进程退出?
    processpool_ptr->KillAll();
    // b.你怎么让所有已经退出的子进程把他进行资源回收wait/waitpid
    processpool_ptr->Wait();
    delete processpool_ptr;
    // wait sub process
    return 0;
}

// task.hpp

#pragma once
#include <iostream>
#include <unistd.h>
using namespace std;

typedef void (*work_t)(int);
typedef void (*task_t)(int, pid_t);
void PrintLog(int fd, pid_t pid)
{
    cout << "sub_process: " << pid << ", fd : " << fd << ", task is : print log task\n"
         << endl;
}

void ReloadConf(int fd, pid_t pid)
{
    cout << "sub_process: " << pid << ", fd : " << fd << ", task is : reload conf task\n"
         << endl;
}

void ConnectMysql(int fd, pid_t pid)
{
    cout << "sub_process: " << pid << ", fd : " << fd << ", task is : connect mysql conf\n"
         << endl;
}

task_t tasks[3] = {PrintLog, ReloadConf, ConnectMysql};

uint32_t NextTask()
{
    return rand() % 3;
}

void worker(int fd)
{
    // sleep(100);
    while (true)
    {
        uint32_t command_code = 0;
        ssize_t n = read(0, &command_code, sizeof(command_code));
        if (n == sizeof(command_code))
        {
            if (command_code >= 3)
                continue;
            tasks[command_code](fd, getpid());
        }
        else if (n == 0)
        {
            cout << " sub process: " << getpid() << " quit now..." << endl;
            break;
        }
    }
}
```

## 命名管道

1.理论

让不同的进程之间可以通信,让不同的进程看到同一份资源.

2. 代码

```cpp
// PipeServer.cc
#include "Comm.hpp"
int main()
{
    Fifo fifo(Path);

    int rfd = open(Path, O_RDONLY);
    if (rfd < 0)
    {
        cerr << "open failed, errno: " << errno << ", errstring" << strerror(errno) << endl;
        return 1;
    }
    // 如果我们的写端没打开,读先打开,open的时候就会阻塞,直到把写端打开,读open才会返回
    cout << "open success" << endl;
    char buffer[1024];
    while (true)
    {
        ssize_t n = read(rfd, buffer, sizeof(buffer) - 1);
        if (n > 0)
        {
            buffer[n] = 0;
            cout << "client say" << buffer << endl;
        }
        else if (n == 0)
        {
            cout << "client quit, me too" << endl;
            break;
        }
        else
        {
            cerr << "read failed, errno: " << errno << ", errstring" << strerror(errno) << endl;
            break;
        }
    }
    close(rfd);
    return 0;
}

// PipeClient.cc
#include "Comm.hpp"

int main() {
    int wfd = open(Path, O_WRONLY);
    if (wfd < 0)
    {
        cerr << "open failed, errno: " << errno << ", errstring" << strerror(errno) << endl;
        return 1;
    }
    string inbuffer;
    while (true)
    {
        cout << "Please Enter Your Message";
        getline(cin, inbuffer);
        if (inbuffer == "quit")
            break;
        ssize_t n = write(wfd, inbuffer.c_str(), inbuffer.size());
        if (n < 0)
        {
            cerr << "write failed, errno: " << errno << ", errstring" << strerror(errno) << endl;
            break;
        }
    }
    close(wfd);
    return 0;
}

//Comm.hpp
#ifndef __COMM_HPP__
#define __COMM_HPP__

#include <iostream>
#include <string>
#include <cerrno>
#include <cstring>
#include <sys/types.h>
#include <sys/stat.h>
#include <unistd.h>
#include <fcntl.h>
using namespace std;

#define Mode 0666
#define Path "fifo"
class Fifo {
    public:
        Fifo(const string &path)
            :_path(path)
        {
            umask(0);
            int n = mkfifo(_path.c_str(), Mode);
            if (n == 0) {
                cout << "mkfifo success" << endl;
            }
            else {
                cerr << "mkfifo failed, errno: " << errno << ", errstring" << strerror(errno) << endl;
            }
        }

        ~Fifo()
        {
            int n = unlink(_path.c_str());
            if (n == 0) {
                cout << "remove fifo file:" << _path << " success" << endl;
            }
            else {
                cerr << "remove failed, errno: " << errno << ", errstring: " << strerror(errno) << endl;
            }
        }
    private:
        string _path; 
};

#endif
```

## system V共享内存

共享内存在内核中同时可以存在很多个-OS必须要管理所有的共享内存->如何管理呢?先描述,在组织->系统会存在很多共享内存哦->你怎么保证
两个或者多个不同的进程看到的是同一个共享内存呢?->要给共享内存提供一个唯一性的标识！

IPC_CREAT and IPC_EXCL

IPC_CREAT:如果共享内存不存在,就创建。如果共享内存已经存在,直接获取它.

IPC_EXCL:不能单独使用,没意义

IPC_CREAT|IPC_EXCL:如果共享内存不存在,就创建。如果共享内存已经存在,出错返回.

如果创建成功的,则一定是一个新的共享内存.

int shmget(key_t key, size_t size, int shmflg);

1.这个key怎么形成?意义是什么?

key_t ftok(const char *pathname, int proj_id);

int shmget(key_t key, size_t size, int shmflg);

是多少,不重要,只要能够标识唯一性即可.

2.为什么要让用户传入呢?

共享内存,如果结束进程,我们没有主动释放它,则一直存在,共享内存的生命周期,随内核--除非重启系统

文件操作,一个进程打开一个文件,进程退出的时候,这个被打开的文件就会被系统释放掉--文件的生命周期随进程

**缺点**:默认情况,shm读取方,根本就没管写入方。共享内存不提供进程间协同的任何机制。

**优点**:共享内存是所有进程间通信最快的!  

## 信号量原理

1.对于共享资源进行保护,是多个执行流场景下,一个比较常见重要的话题

2.互斥:在访问一部分共享资源的时候,任何时刻只有我一个人访问。同步:访问资源在安全的前提下,具有一定是顺序性

3.被保护起来任何时刻只允许一个执行访问的公共资源 --- 临界资源

4.能访问临界资源的代码,我们叫做临界区 -- 非临界区 --所谓的保护公共资源-临界志愿 -> 保护公共资源的本质:是程序员保护临界区

5.原子性:操作对象的时候,只有两种状态,要么是还没开始,要么已经结束.


## 信号量理论

信号量(信号灯) -- 看电影 -- 资源 -- 本质: 对资源的预定机制 -- 资源不一定被我持有,才是我的,只要我预定了,在未来的某个时间,就是我的.

我有多少资源,最多我卖多少票 --- 每一份资源不会被并发访问

只有一个座位,就只有一张票 --- 电影院资源是被整体使用的

只要买票的过程是合理的->电影院是公共资源,座位->切分电影院,让电影院里面有无数个小资源

信号量:本质是一个计数器,描述临界资源的计数器


进程:申请信号量[申请成功才能继续往下走] --- 买票 --- 预定 --- 信号量申请成功,就一定有你的资源

释放信号量 --- "看完电影" --- 别人可以继续申请了

多进程场景,int能不能实现信号量的效果?

不能.

1.无法在进程间共享 -- 让不同的进程先看到相同一份资源 -- 计数器资源!!

2.count++,count--不是原子的

申请sem和释放sem来保护临界资源,大家都要遵守的规则

所有的进程，访问临界资源，都必须先申请信号量---所有的进程都得先看到同一个信号量--信号量本身就是共享资源！！！---信号量的申请(--)和释放

如果信号量就是1呢?

不就是互斥吗!!!二元信号量就是一把锁。

1.申请信号量资源

2.释放信号量资源

3.PV操作

一个信号量就是一个计数器

一次申请多个信号量 VS 信号量是几 完全不同的概念

sem_t sems[4]

## 信号量的接口

1.semget(信号量申请)
```cpp
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/sem.h>
int semget(key_t key, int nsems, int semflg);
// int:信号量集标识符
// nsems:你想创建几个信号量
```

2.semctl(信号量删除)
```cpp
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/sem.h>
int semctl(int semid, int semnum, int cmd, ...);
```

3.semop(PV操作)
```cpp
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/sem.h>
int semop(int semid, struct sembuf *sops, unsigned nsops);
// spos = -1, P操作
// spos = 1, V操作
```

共享内存 信号量 消息队列 --- 共性 --- OS特意设计的。System V进程间通信的. --- OS注定要对IPC资源 -> 先描述，再组织

内核中，所有的描述管理IPC资源的结构体，第一个成员都一样kern_ipc_perm(C++多态)

我们怎么知道它的类型是什么呢?? 
```cpp
#define IPC_TYPE_SHM 0x1
#define IPC_TYPE_MSG (0x1 << 1)
#define IPC_TYPE_SEM(0x1 << 2)
shmid_kernel* (kern_ip_perm* p) {
    if (o->mode & IPC_TYPE_SHM)
        return (shmid_kernel*)p;
    else
        return null;
}
```