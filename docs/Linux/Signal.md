# 信号

信号 vs 信号量:两个没有任何关系!

**是什么?**

Linux系统提供的让用户(进程)给其他进程发送异步信息的一种方式

1.在没有发生的时候,我们已经知道怎么处理了

2.信号我们能够认识,很早之前,有人给我们大脑设置了识别特定信号的方式

1和2我们能识别一个信号并处理

3.信号到来的时候,我们正在处理更重要的事情,我们暂时不能处理到来的信号,我们必须暂时要将到来的信号进行临时保存(保存在进程的PCB中). 

4.信号到了,可以不立即处理,可以在合适的时候处理

5.信号的产生是随时产生的,我们无法预料,所以信号是异步发送的.(信号的产生,是由别人(用户,进程)产生的,我收到之前,我一直在忙我的事情,并发在跑的)

1-5:进程看待信号的方式

**为什么?**

停止,删掉..系统要求有随时响应信号的能力,随后做出反应

**怎么办?**

## 准备
1.常见信号: 数字和名字都可以标识信号,名字真实就是宏(没有0和32、33)、34-64:实时信号---不做处理,所以常用信号1-31

2.信号处理的方式 --- signal

a.默认动作

b.自定义处理信号 --- 捕捉

c.忽略了信号 --- 是处理了信号吗?是

更改我对信号的处理方式


## 信号的产生

1.kill 命令(-9(SIGKILL),-2(SIGINT):进程自己终止)

```cpp
#include <iostream>
#include <unistd.h>
#include <signal.h>
#include <sys/types.h>

void handler(int signo) {
    // 简单的打印一句话
    std::cout << "get a sig, number is :" << signo << std::endl;
}

int main() {
    // signal调用完了,handler方法会被立即执行吗?不是,设置对应信号的处理方法
    // 未来我们收到对应的信号才执行handler方法
    // 未来进程永远没有收到SIGINT呢?handler也就永远不会被调用
    signal(SIGINT, handler); //自定义处理
    singal(SIGINT, SIG_IGN); //忽略
    // ctrl + c为什么可以终止程序:Ctrl + c -> 解释成信号 -> 向目标进程进行发送 -> 收到进程 -> 响应进程
    while(true) {
        std::cout << "I am active...,pid: " << getpid() << std::endl;
        sleep(1); 
    }
    return 0;
}
```

2.键盘

ctrl + c / ctrl + \

3.系统调用:
```cpp
int kill(pid_t pid, int sig); // 对任意进程发送信号

int raise(int sig); // 对自己发送信号

void about(void); // 终止进程,对自己发送指定的信号 6) SIGABRT

unsigned int alarm(unsigned int seconds); 
// 1.alarm只响一次,如果要多次响应要嵌套调用
// 2.返回值是现在调用alarm时间距离上次一次调用alarm结束时间的差值(单位s)
// 3. alarm(0):取消闹钟 
```

4.软件条件

设置闹钟,其实是OS内部设定的.可能同时存在很多的闹钟.管理闹钟?先描述，再组织.
```cpp
struct alarm {
    pid_t pid;
    uint64 expired;//过期时间
}
```
// 先描述,再组织

5.异常

a.代码除0了 8)SIGEPE

b.野指针 11)SIGSEGV

**关于信号产生的各种情况的理解**

键盘产生信号

a.按键按下了

b.那些按键按下了

c.字符输入(字符设备),组合键输入(输入的是命令,abcd/ctrl + c ->键盘驱动和OS进行联合解释的)

操作系统怎么知道键盘在输入数据了?

硬件中断的技术(中断向量表)

临时保存->保存在进程的PCB中->位图结构

比特位的位置,表示信号的编号,比特位的内容(0/1)是/否收到指定的信号

给进程发送信号就是写信号!!!

task_struct是内核数据结构->只有OS有资格写入->用户也想写信号?OS提供系统调用!!!

->无论信号产生有多少种,最终都是OS动手向进程写入信号

寄存器只有一个,但是寄存器的数据可以有很多,我们把寄存器的数据叫做: 上下文数据!!!

野指针->虚拟地址->转化(OS + CPU(MMU(虚拟到物理转换的)))->只读区域要写,OS直接返回报错

## 信号的保存

Core:终止进程

Term:termination

云服务器默认将进程Core退出,进行了特定的设定,默认Core是被关闭的.

(为什么默认关闭核心转储功能:防止未知的core dump一直在进行,导致服务器磁盘被打满)
```cpp
int main() {
    int a = 10;
    a /= 0;
    return 0;
}
ulimit -c 10240 //打开Core+pid(新版没有pid)
```
现象会生成一个core文件

为什么?

想通过Core定位到进程为什么退出,以及执行到哪行代码退出的.

是什么?

将进程在内存中的核心数据(与调试有关)转存储中形成Core,core.pid的文件

有什么用?

协助调试(core-file core):直接定位到问题位置(事后调试)

```cpp
int main() {
    pid_t id = fork();
    if (id == 0) {
        sleep(2);
        int a = 10;
        a /= 0;
        exit(0);
    }
    int status = 0;
    pid_t rid = waitpid(id, &status, 0);
    if (rid > 0) {
        cout << "exit code " << ((status >> 8) & 0xFF) << endl;
        cout << "exit signal" << (status & 0x7F) << endl;
        cout << "code dump" << ((status >> 7) & 0x1) << endl;
    }
    return 0;
}
```

系统调用

+ 实际执行信号的处理动作称为递达(默认、忽略、自定义)

+ 信号从产生到递达之间的状态,称为信号未决

+ 进程可以选择阻塞某个信号

(0000 0000 0000 0000 0000 0000 0000 0000 --- 比特位的位置,表示信号编号,比特位的内容,是否收到指定信号)

(0000 0000 0000 0000 0000 0000 0000 0000 --- 比特位的位置,表示信号编号,比特位的内容,是否阻塞该信号)

如果一个信号被阻塞(屏蔽)，则该信号永远不会被递达处理,除非解除阻塞.

阻塞 vs 忽略区别:忽略是一种信号递达的方式,阻塞仅仅是不让信号进程递达.

阻塞一个信号,和是否收到指定信号,有关系吗?

没关系

理论:记住3张表即可.

OS->发送信号->OS想目标进程写入信号

三张表匹配的操作和系统调用

```cpp
// 有没有涉及到将数据设置进内核呢?没有
// sigset_t数据类型,int double float class没有差别
sigset_t s; //用户栈上开辟了空间
sigemptyset(&s);
sigaddset(&s, 2);
return 0;
```

```cpp
void handler(int signo) {
    cout << signo << "号信号被递达处理..." << endl;
}

void PrintSig(sigset_t &pending)
{
    cout << "Pending bitmap: ";
    for (int signo = 31; signo > 0; signo--)
    {
        if (sigismember(&pending, signo))
        {
            cout << "1";
        }
        else
            cout << "0";
    }
    cout << endl;
}

int main()
{
    // 对2号信号进行自定义捕捉 --- 不让进程因为2号信号而终止
    signal(2, handler);
    // 1.屏蔽2号信号
    sigset_t block, oblock;
    sigemptyset(&block);
    sigemptyset(&oblock);
    // 0.如果我屏蔽了所有信号呢???
    sigaddset(&block, 2); // SIGSET --- 根本就没有设置当前进程的PCB block位图中
    // 9号、19号系统无法被屏蔽;18号会被特殊处理 
    // for (int signo = 1; signo <= 31; signo++) 
    //     sigaddset(&block, signo);
    // 1.1 开始屏蔽2号信号,其实就是
    int n = sigprocmask(SIG_SETMASK, &block, &oblock);
    assert(n == 0);
    // (void)n; //骗过编译器,不要警告,因为我们后面用n,不光光是定义
    cout << "block 2 signal success" << endl;
    cout << getpid() << endl;
    int cnt = 0;
    while (true)
    {
        // 2.获取进程的pending位图
        sigset_t pending;
        sigemptyset(&pending);
        n = sigpending(&pending);
        assert(n == 0);
        // 3.打印pengding位图中收到的信号
        PrintSig(pending);
        cnt++;
        // 4.解除对2号信号的屏蔽
        if (cnt == 20) {
            cout << "解除对2号信号的屏蔽" << endl;
            n = sigprocmask(SIG_UNBLOCK, &block, &oblock); // 2号信号会被立即递达，默认处理是终止进程
            assert(n == 0);
        }

        // 我还想看到pending 2号信号1->0:递达2号信号！

        sleep(1);
    }
}
```

细节:

a.递达的时候,就一定会把对应的pending位图进行清0

b.先清0,再递达 还是 先递达,再清0(先清0,再递达)
## 信号的处理

**1.信号什么时候被处理的?**

合适的时候,什么是合适的时候呢?进程从内核态(操作系统的状态,权限级别高),切换会用户态(你自己的状态)的时候,信号检测并处理(进程是会被调度的,进程是有时间片的)

在信号处理的过程中,一共会有4次的状态切换(内核和用户态)

为什么我们在信号捕捉的时候,执行我们写的方法,还要从内核态转换回用户态?

用户可能会执行一些越权的非法操作

信号 --- 杀掉这个进程 --- SIG_DFL

**2.信号如何被处理的?**

内核态 --- 地址空间的第三讲 --- 轻一点

系统调用本质是一堆函数指针数组

1.我们使用系统调用或者访问系统数据,其实还是在我进程的地址空间内进行跳转的

2.进程无论如何切换,总能找到OS(我们访问OS,本质就是通过我的进程的地址空间的[3,4]GB)来访问即可

3.操作系统是如何正常运行的？->信号技术本来就是通过软件的方式,来模拟软件中断(非常高频率的,每个非常短的时间，就给CPU发送中断---cpu不断地进行处理中断).->谁让OS运行起来呢?

4.操作系统不相信任何用户

**3.捕捉信号还有其他方式吗?signal?**

```cpp
int sigaction(int signum, const struct sigaction *act,struct sigaction *oldact);
```
当某个信号的处理函数被调用时,内核自动将当前信号加入进程的信号屏蔽字

如果我们处理对应的信号,该信号默认也会从信号屏蔽字中进行移除

不想让信号,嵌套式进行捕捉处理

在调用信号处理函数时,除了当前信号被自动屏蔽之外,还希望自动屏蔽另外一些信号,则用sa_mask字段说明;

## 可重入函数

该函数执行流重复进入了,导致产生了问题:该函数我们叫做不可重入函数,否则叫做可重入函数

我们用到的大部分函数都是不可重入的!

```cpp
void Print(sigset_t& pending) {
    cout << "cur process pending:";
    for (int sig = 31; sig > 0; sig--) {
        if (sigismember(&pending, sig))
            cout << "1";
        else
            cout << "0";
    }
    cout << endl;
}

void handler(int signo) {
    cout << "signo: " << signo << endl;
    // 不断获取当前进程的pending信号集合并打印
    sigset_t pending;
    sigemptyset(&pending);
    while (true) {
        sigpending(&pending);
        Print(pending);
    }
```

```cpp
int g_flag = 0;

void changeflag(int signo) {
    (void)signo;
    printf("将flag,从%d->%d\n", g_flag, 1);
    g_flag = 1;
}

int main() {
    signal(2, changeflag);
    while (!g_flag); // 故意写成这个样子,编译器默认会对我们的代码进行自动优化(实际上就是把内存中的g_flag放入寄存器导致g_flag实际改变了也不会改变g_flag,volatile可以不进行优化)
    printf("process quit normal\n");
    return 0;
}
```

## SIGCHLD信号

子进程退出,不是默默退出的,会在退出的时候,向父进程发送信号,发送17)SIGCHLD信号

如何证明?

```cpp
void handler(int signo) {
    cout << "child quit, father get a signo: " << signo << endl; 
}

int main() {
    signal(SIGCHLD, handler);
    pid_t pid = fork();
    if (pid == 0) {
        //child
        int cnt = 5;
        while (cnt--) {
            cout << "I am Child process: " << getpid() << endl;
            //sleep(1);
        }
        cout << "child process died" << endl;
        exit(0);
    }
    while (true)
        sleep(1);
    return 0;
}
```
如何在多个进程下都成功释放,不变成僵尸进程
```cpp
void CleanupChild(int signo)
{
    if (signo == SIGCHLD)
    {
        while (true)
        {
            pid_t rid = waitpid(-1, nullptr, WNOHANG); // -1回收任意一个子进程
            if (rid > 0)
            {
                cout << "wait child success" << rid << endl;
            }
            else
                break;
        }
    }
    cout << "wait sub process done" << endl;
}
int main()
{
    signal(SIGCHLD, CleanupChild);
    for (int i = 1; i <= 100; ++i)
    {
        pid_t pid = fork();
        if (pid == 0)
        {
            // child
            int cnt = 5;
            while (cnt--)
            {
                cout << "I am Child process: " << getpid() << endl;
                sleep(1);
            }
            cout << "child process died" << endl;
            exit(0);
        }
    }
    while (true)
        sleep(1);
    return 0;
}
```
方法二:
```cpp
int main()
{
    signal(SIGCHLD, SIG_IGN);
    for (int i = 1; i <= 100; ++i)
    {
        pid_t pid = fork();
        if (pid == 0)
        {
            // child
            int cnt = 5;
            while (cnt--)
            {
                cout << "I am Child process: " << getpid() << endl;
                sleep(1);
            }
            cout << "child process died" << endl;
            exit(0);
        }
    }
    while (true)
        sleep(1);
    return 0;
}
```


