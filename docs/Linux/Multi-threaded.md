# Linux多线程

## 线程的概念

线程是进程内部的一个执行分支，线程是CPU调度的基本单位(?)

加载到内存中的内存，叫做进程。修正：进程 = 内核数据结构 + 进程代码和数据



## 线程的理解(Linux系统为例) --- 一般系统0

v1:

代码段(区)，我们的代码在进程中，全部都是串行调用的！

进程新建，成本较高(时间和空间)

地址空间和地址空间的虚拟地址，本质是一种"资源"

v2:

如果我们要设计线程,OS也要对线程进行管理！！！先描述，再组织

为什么这么设计Linux线程？

Linux的设计这认为，进程和线程都是执行流，具有极度的相似性，没必要单独设计数据结构和算法直接复用代码.使用进程来模拟线程！！

v3:

进程 vs 线程

以前的进程:一个内部只有一个线程的进程

现在的进程:一个内部至少有一个的进程

我们以前讲的进程，是今天讲的特殊情况

什么是进程？

进程的内核角度：承担分配系统资源的基本实体

不要站在调度的角度理解进程，而应该站在资源角度理解进程

v4:

关于调度问题

不用区分task_struct(进程???都是执行流)

线程<=执行流<=进程(轻量级进程！！！，Linux所有的调度执行流都叫做:轻量级进程)

OS理论的视角:进程内部只有一个执行流:进程。进程内部有多个执行流:线程

## 地址空间的第四谈(页表、虚拟地址和物理地址)

多个执行流是如何进行代码划分?如何理解?理论，实操

操作系统要不要管理内存呢？

内存是4KB为单位，4GB是1048576(1024*1024)个数据块。

对内存进行管理，就是对该数组的增删查改！！！

操作系统的内存管理的基本单位是4KB！！！

虚拟地址:前10位页目录，中间10位页表,0x1234+虚拟地址后12位对应的数据！！--页内偏移

给不同的线程分配不同的区域,本质就是让不同的线程,各自看到全部页表的子集！！

## 线程的控制 -- 重点在验证 --- 谈完 --- C++线程 vs 我们的线程
```cpp
int pthread_create(pthread_t *thread, const pthread_attr_t *attr, void *(*start_routine) (void *), void *arg);
// thread:线程ID
// attr:线程属性
// start_routine:函数指针
// arg:传递给线程函数的参数
```
```cpp
#include <iostream>
#include <pthread.h>
#include <unistd.h>
using namespace std;

void* newthreadrun(void* args) {
    while (true) {
        cout << "I am new thread" << endl;
        sleep(1);
    }
}

int main() {
    pthread_t tid;
    pthread_create(&tid, nullptr, newthreadrun, nullptr);
    while (true) {
        cout << "I am main thread" << endl;
        sleep(1);
    }
    return 0;
}
```

ps -aL | head -1 && ps -aL | grep test_thread

LWP:light weight process 轻量级进程

所以，操作系统在进行调度的时候，用哪一个ID来进行调度呢？

单进程，多进程？？---每一个进程内部只有一个执行流，LWP==PID

函数编译完成,是若干行代码(每一行代码都有地址---虚拟地址(逻辑地址))，函数名是该代码块的入口地址！

最后形成的是一个可执行程序 --- 所有的函数，都要按照地址空间统一编址

## 线程控制

1.铺垫

Linux里面有没有真线程呢？没有，Linux里只有轻量级进程。

用户知道"轻量级进程"这个概念吗？没有！！进程和线程。

Linux系统，不会有线程相关的系统调用，只有轻量级进程的系统调用。

用户和内核之间有一个pthread库---原生线程库，核心工作：将轻量级进程的系统调用进行封装，转成线程相关的接口语义提供给用户。(Linux系统必须自带)->用户级线程

编写多线程,都必须-lpthread



2.操作
```cpp
#include <iostream>
#include <pthread.h>
#include <unistd.h>
#include <string>
using namespace std;

// 同一个进程内的线程,大部分资源是共享的,地址空间是共享的！

string ToHex(pthread_t tid) {
    char id[64];
    snprintf(id, sizeof(id), "0x%lx", tid);
    return id;
}

// 线程退出
// 1.代码跑完,结果对
// 2.代码跑完,结果错
// 3.出异常了 --- 重点 --- 多线程中，任何一个线程出现异常(div 0, 野指针),都会导致整个进程退出! ---- 多线程代码往往健壮性不好

void *newthreadrun(void *args)
{
    string threadname = (char*)args;
    int cnt = 5;
    while (cnt)
    {
        cout << threadname << " is running" << cnt << ", pid: " << getpid() << " mythread id: " << ToHex(pthread_self())<< endl;
        sleep(1); 
        cnt--;
    }
    // 1. 线程函数结束
    // 2. 
    return (void*)123;
}
// 主线程退出 == 进程退出 == 所有线程都要退出
// 1. 往往我们需要main最后结束
// 2. 线程也要被"wait", 要不然会产生类似进程哪里内存泄漏的问题
int main()
{
    // 1. id
    pthread_t tid;
    pthread_create(&tid, nullptr, newthreadrun, (void*)"thread-1");
    // 在主线程中，你保证新线程已经启动
    sleep(2);
    pthread_cancel(tid);


    // 2.新和主两个线程,谁先运行呢?不确定,由调度器决定
    // int cnt = 10;
    // while (cnt)
    // {
    //     cout << "main thread is running" << cnt << ", pid: " << getpid() 
    //     << "new thread id: " << ToHex(tid) << " "
    //     << " main thread id: " << ToHex(pthread_self())<< endl;
    //     sleep(1);
    //     cnt--;
    // }
    void* ret = nullptr;
    // 不考虑线程异常情况！
    int n = pthread_join(tid, &ret); // 我们怎么没有像进程一样获取线程退出的退出信号呢?只有你手动写的退出码
    cout << "main thread quit, n = " << n << "main thread get a ret: " << (long long)ret << endl;
    return 0;
}
```

a.线程创建

```cpp
int pthread_create(pthread_t *thread, const pthread_attr_t *attr, void *(*start_routine) (void *), void *arg);
```

b.线程等待

```cpp
int pthread_join(pthread_t thread, void **retval);
```

c.线程终止

return 

pthread_exit

pthread_cancel

线程私有:

1.线程的硬件上下文(CPU寄存器的值)(调度)

2.线程的独立栈结构(常规运行)

线程共享:

1.代码和全局数据

2.进程文件描述符表

1.一个线程出问题,导致其他线程也出问题,导致整个进程退出 --- 线程安全问题

2.多线程中,公共函数如果被多个线程同时进入 --- 该函数被重入了。

```cpp
#include <iostream>
#include <pthread.h>
#include <unistd.h>
#include <string>
#include <vector>
using namespace std;

const int threadnum = 5;

class Task {
public:
    void SetTask(int x, int y) {
        _x = x;
        _y = y;
    }
    int Excute() {
        return _x + _y;
    }
private:
    int _x;
    int _y;
};

class ThreadData {
public:
    ThreadData(int x, int y, string threadname)
        :_threadname(threadname)
    {
        _t.SetTask(x, y);
    }
    string threadname() {
        return _threadname;
    }
    int run() {
        return _t.Excute();
    }
private:
    string _threadname;
    Task _t;
};

class Result {
public:
    void SetResult(int result, string threadname) {
        _result = result;
        _threadname = threadname;
    }
    void Print() {
        cout << _threadname << " : " << _result << endl; 
    }
private:
    int _result;
    string _threadname;
};

void *handlerTask(void *args) { 
    ThreadData* td = static_cast<ThreadData*>(args);
    string name = td->threadname();
    int result = td->run();
    Result* res = new Result();
    res->SetResult(result, name);
    //cout << name << " run result: " << result << endl;
    delete td; 
    sleep(2);
    return res;
    // const char* threadname = static_cast<char*>(args);
    // while(true) {
    //     sleep(1);
    //     cout << "I am " << threadname << endl;
    // }
    // delete[] threadname;
    // return nullptr;
}

// 1.多线程创建
// 2.线程传参和返回值,我们可以传递级别信息,也可以传递其他对象(包括你自己定义的！)
// 3.C++11也带了多线程,和上面有什么关系？？？
int main() {
    vector<pthread_t> threads;
    for (int i = 0; i < threadnum; ++i) {
        char* threadname = new char[64];
        snprintf(threadname, 64, "Thread-%d", i + 1);
        ThreadData *td = new ThreadData(10, 20, threadname);
        pthread_t tid;
        pthread_create(&tid, nullptr, handlerTask, td);
        threads.push_back(tid);
    }
    vector<Result*> result_set;
    void* ret = nullptr;
    for (auto& tid:threads) {
        pthread_join(tid, &ret);
        result_set.push_back((Result*)ret);
    }
    for (auto &res: result_set) {
        res->Print();
        delete res;
    }
    return 0;
}
```

C++ 11的多线程,是对原生线程的封装！！！--- 我们也要做一次

1.为什么要封装

语言的跨平台性->C++标准库

2.Windows呢？

不需要

3.其他语言呢？

Linux提供多线程的底层唯一方式！

d.线程分离

系统中没有线程,只有轻量级进程的概念

用户能不能通过接口，管理线程呢？比如创建，终止进程等。

线程库首先要映射到当前进程的地址空间中！

线程的管理工作,由库来进行管理！

```cpp

__thread int g_val = 100;

// 线程是可以分离的:默认线程是joinable的.
// 如果我们main thread不关心新线程的执行信息，我们可以将新线程设置为分离状态
// 你是如何理解线程分离呢?底层依旧数据进程！只是不需要等待了
// 一般都希望mainthread是最后一个退出的,无论是否是join，detach

void *threadrun1(void* args) {
    // pthread_detach(pthread_self());
    string name = static_cast<const char*>(args);
    // int cnt = 5;
    while (true) {
        sleep(1);
        cout << "I am a new thread..." << getpid() << ", g_val: " << g_val << "&g_val" << &g_val << endl;
    }
    // while (cnt) {
    //     // if (!(cnt--))
    //     //     break;
    //     cout << "I am a new thread..." << getpid() << ", cnt: " << cnt << "&cnt" << &cnt << endl;
    //     cnt--;
    //     sleep(1);
    // }
    return nullptr;
}

void *threadrun2(void* args) {
    // pthread_detach(pthread_self());
    string name = static_cast<const char*>(args);
    // int cnt = 5;
   while (true) {
        cout << "I am a new thread..." << getpid() << ", g_val: " << g_val << "&g_val" << &g_val << endl;
        g_val--;
        sleep(1);
   }
    // while (cnt) {
    //     // if (!(cnt--))
    //     //     break;
    //     cout << "I am a new thread..." << getpid() << ", cnt: " << cnt << "&cnt" << &cnt << endl;
    //     cnt--;
    //     sleep(1);
    // }
    return nullptr;
}

int main() {
    pthread_t tid1;
    pthread_t tid2;
    pthread_create(&tid1, nullptr, threadrun1, (void*)"thread 1");
    pthread_create(&tid2, nullptr, threadrun2, (void*)"thread 1");
    
    pthread_join(tid1, nullptr);
    pthread_join(tid2, nullptr);
    // pthread_detach(tid);

    // while (true) {
    //     cout << "I am main thread" << endl;
    //     sleep(1);
    // }
    // cout << "main thread wait block" << endl;
    // int n = pthread_join(tid, nullptr);
    // cout << "main thread wait return" << n << ": " << strerror(n) << endl;

    return 0;
}
```

线程封装:
```cpp

// .hpp:
#include <iostream>
#include <string>
#include <unistd.h>
#include <pthread.h>
#include <functional>
namespace ThreadModule
{
    template<typename T>
    using func_t = std::function<void(T&)>;
    
    template<typename T>
    class Thread
    {
    public:
        void Excute()
        {
            _func(_data);
        }
        Thread(func_t<T> func, const T& data, const std::string& name = "none-name")
            :_func(func),
            _data(data),
            _threadname(name),
            _stop(true)
        {}

        static void* threadroutine(void* args) // 类成员函数,形参是有this指针的！！
        {
            Thread<T>* self = static_cast<Thread<T>*>(args);
            self->Excute();
            return nullptr;
        }

        bool Start()
        {
            int n = pthread_create(&_tid, nullptr, threadroutine, this);
            if (n == 0) 
            {
                _stop = false;
                return true;
            }
            else 
            {
                return false;
            }
        }

        void Detach()
        {
            if (!_stop) 
            {
                pthread_detach(_tid);
            }
        }

        void Join()
        {
            if (!_stop)
            {
                pthread_join(_tid, nullptr);
            }
        }

        std::string name()
        {
            return _threadname;
        }

        void Stop()
        {
            _stop = true;
        }

        ~Thread(){}
    private:
        pthread_t _tid;
        std::string _threadname;
        T _data;
        func_t<T>  _func;
        bool _stop;
    };
}

// .cpp
#include <iostream>
#include <vector>
#include "Thread.hpp"
using namespace ThreadModule;

void print(int& cnt) 
{
    while (cnt) 
    {
        std::cout << "hello I am myself thread, cnt: " << cnt-- << std::endl;
        sleep(1);
    }
}
const int num = 10;
int main()
{
    std::vector<Thread<int>> threads;
    // 1. 创建一批线程
    for (int i = 0; i < num; ++i)
    {
        std::string name = "thread-" + std::to_string(i + 1);
        threads.emplace_back(print, 10, name);
    }
    // 2. 启动一批线程
    for (auto& thread : threads)
    {
        thread.Start();
    }
    // 3. 等待一批线程
    for (auto& thread : threads)
    {
        thread.Join();
        std::cout << "Wait thread done, thread is: " << thread.name() << std::endl;
    }







    Thread<int> t1(print, 10);
    t1.Start();
    
    std::cout << "name: " << t1.name() << std::endl;
  
    t1.Join();
    return 0;
}
```

抢票系统:
```cpp
//.hpp
#include <iostream>
#include <string>
#include <unistd.h>
#include <pthread.h>
#include <functional>
namespace ThreadModule
{
    template<typename T>
    using func_t = std::function<void(T)>;
    
    template<typename T>
    class Thread
    {
    public:
        void Excute()
        {
            _func(_data);
        }
        Thread(func_t<T> func, T data, const std::string& name = "none-name")
            :_func(func),
            _data(data),
            _threadname(name),
            _stop(true)
        {}

        static void* threadroutine(void* args) // 类成员函数,形参是有this指针的！！
        {
            Thread<T>* self = static_cast<Thread<T>*>(args);
            self->Excute();
            return nullptr;
        }

        bool Start()
        {
            int n = pthread_create(&_tid, nullptr, threadroutine, this);
            if (n == 0) 
            {
                _stop = false;
                return true;
            }
            else 
            {
                return false;
            }
        }

        void Detach()
        {
            if (!_stop) 
            {
                pthread_detach(_tid);
            }
        }

        void Join()
        {
            if (!_stop)
            {
                pthread_join(_tid, nullptr);
            }
        }

        std::string name()
        {
            return _threadname;
        }

        void Stop()
        {
            _stop = true;
        }

        ~Thread(){}
    private:
        pthread_t _tid;
        std::string _threadname;
        T _data; // 为了让所有的线程访问同一个全局变量
        func_t<T>  _func;
        bool _stop;
    };
}
//.cpp
int g_tickets = 10000; //共享资源,没有保护的

class ThreadData
{
public:
    ThreadData(int& tickets, const std::string& name)
        :_tickets(tickets),
        _name(name),
        _total(0)
    {}
    ~ThreadData()
    {}

public:
    int& _tickets; //所有的
    std::string _name;
    int _total;
};

void route(ThreadData* td)
{
    while (true) 
    {
        if (td->_tickets > 0) 
        {
            usleep(1000);
            printf("%s running, get tickets:%d\n", td->_name.c_str(), td->_tickets);
            td->_tickets--;
            td->_total++;
        }
        else
        {
            break;
        }
    }
}

const int num = 4;
int main()
{
    std::vector<Thread<ThreadData*>> threads;
    std::vector<ThreadData*> datas;
    // 1. 创建一批线程
    for (int i = 0; i < num; ++i)
    {
        std::string name = "thread-" + std::to_string(i + 1);
        ThreadData* td = new ThreadData(g_tickets, name);
        threads.emplace_back(route, td, name);
        datas.emplace_back(td);
    }
    // 2. 启动一批线程
    for (auto& thread : threads)
    {
        thread.Start();
    }
    // 3. 等待一批线程
    for (auto& thread : threads)
    {
        thread.Join();
        std::cout << "Wait thread done, thread is: " << thread.name() << std::endl;
    }

    // 4. 输出统计数据
    for (auto& data : datas)
    {
        std::cout << data->_name << " : " << data->_total << std::endl;
        delete data;
    }
    return 0;
}
```
1.先解释为什么抢到了负数

判断是逻辑运算,必须在CPU内部运行

tickets-- 等价与 tickets = tickets - 1

1.从内存读取到CPU

2.CPU内部进行--操作

3.写回内存

不是原子的

共享资源在被访问的时候，没有被保护

什么是原子的？

一条汇编

解决问题:

pthread_mutex_init、pthread_nutex_destroy

如果你定义的锁是静态的或者全局的,不需要init,destroy

pthread_mutex_lock(申请锁成功:函数就会返回,允许你继续向后运行.申请锁失败:函数就会阻塞,不允许你继续向后运行)

pthread_mutex_unlock:解锁

解决方案一:

出现的并发访问的问题，本质是因为多个执行流执行访问全局数据的代码道闸机的.

保护全局资源 本质是通过保护临界区完成的!

cpu寄存器硬件只有一套,但是CPU寄存器内部的数据,数据线程的硬件上下文。

数据在内存里,所有线程都能访问,属于共享的.但是如果转移到CPU内部寄存器中,就属于一个线程私有了.

交换的本质：不是拷贝到寄存器,而且所有的线程在争锁的时候,只有一个1.

一个线程正在访问临界区,没有释放之前,对于其他线程:1.锁被释放 2.曾经我没有申请到锁

## 条件变量

1.理解条件变量是什么? 例子

条件变量 = 铃铛 + 队列

pthread_cond_t

2.快速认识接口

int pthread_cond_init(pthread_cond_t *restrict cond, const pthread_condattr_t *restrict attr);

int pthread_cond_destroy(pthread_cond_t *cond);

int pthread_cond_wait(pthread_cond_t *restrict cond, pthread_mutex_t *restrict mutex);

int pthread_cond_broadcast(pthread_cond_t *cond);(唤醒所有在等待的线程)

int pthread_cond_signal(pthread_cond_t *cond);(唤醒一个线程)

3.快速写一个测试代码 --- 重点验证线程同步机制

4.生产消费模型

5.生产消费模型 + 条件变量