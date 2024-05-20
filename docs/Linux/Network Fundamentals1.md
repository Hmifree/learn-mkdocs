# 网络基础(一)


## 计算机网络背景

独立模式: 计算机之间相互独立;

网络互联: 多台计算机连接在一起, 完成数据共享;

局域网LAN: 计算机数量更多了, 通过交换机和路由器链接在一起;

广域网WAN: 将远隔千里的计算机都连在一起;

所谓 "局域网" 和 "广域网" 只是一个相对的概念;

广域网, 有时也可以看做一个比较大的局域网;

## 协议

1. 操作系统要进行协议管理 -- 先描述，在组织
2. 协议本质就是软件, 软件是可以"分层"
3. 协议在设计的时候, 就是被层状划分的
4. 为什么要划分成层状结构呢? a.场景复杂 b.功能解耦,便于人们进行各种维护

通信的复杂, 本质是和距离成正相关.

复杂体现在哪里? -- 协议栈要解决的问题吗?

应用范畴:

0. 如何处理数据  --- 应用层 (系统调用)

通信范畴:

1. 丢包  --- 传输层(操作系统)
2. 定位问题 --- 网络层(操作系统)
3. 解决下一跳主机的问题 --- 数据链路层(驱动)、物理层(硬件)

0-3：TCP/IP协议

局域网中两台主机是可以直接通信的

每层都有自己的协议定制方案

每层都要有自己的协议报头

从上到下交付数据的时候,要添加报头

从下到上递交数据的时候,要去掉报头

报头类似我们取快递的快递单

**封装**:自顶向下添加报头的过程

**解包**:自底向上去掉报头,展开分析

局域网中表示主机的唯一性: MAC地址

**mac地址和ip地址的区别**:

类似A->Z(源IP->目标IP), 当你在C的时候,下一站是D(源Mac地址->下一站Mac地址)

总结:在使用tcp/ip协议的网络中,IP及其向上的协议,看到的报文都是一样的.

## OSI七层模型

+ OSI（Open System Interconnection，开放系统互连）七层网络模型称为开放式系统互联参考模型，
是一个逻辑上的定义和规范;
+ 把网络从逻辑上分为了7层. 每一层都有相关、相对应的物理设备，比如路由器，交换机;
+ OSI 七层模型是一种框架性的设计方法，其最主要的功能使就是帮助不同类型的主机实现数据传输; 
+ 它的最大优点是将服务、接口和协议这三个概念明确地区分开来，概念清楚，理论也比较完整. 通过七
个层次化的结构模型使不同的系统不同的网络之间实现可靠的通讯;
+ 但是, 它既复杂又不实用; 所以我们按照TCP/IP四层模型来讲解.

## TCP/IP五层(或四层)模型

TCP/IP是一组协议的代名词，它还包括许多协议，组成了TCP/IP协议簇. 
TCP/IP通讯协议采用了5层的层级结构，每一层都呼叫它的下一层所提供的网络来完成自己的需求.

+ 物理层: 负责光/电信号的传递方式. 比如现在以太网通用的网线(双绞 线)、早期以太网采用的的同轴电缆
(现在主要用于有线电视)、光纤, 现在的wifi无线网使用电磁波等都属于物理层的概念。物理层的能力决
定了最大传输速率、传输距离、抗干扰性等. 集线器(Hub)工作在物理层.
+ 数据链路层: 负责设备之间的数据帧的传送和识别. 例如网卡设备的驱动、帧同步(就是说从网线上检测
到什么信号算作新帧的开始)、冲突检测(如果检测到冲突就自动重发)、数据差错校验等工作. 有以太
网、令牌环网, 无线LAN等标准. 交换机(Switch)工作在数据链路层.
+ 网络层: 负责地址管理和路由选择. 例如在IP协议中, 通过IP地址来标识一台主机, 并通过路由表的方式规
划出两台主机之间的数据传输的线路(路由). 路由器(Router)工作在网路层.
+ 传输层: 负责两台主机之间的数据传输. 如传输控制协议 (TCP), 能够确保数据可靠的从源主机发送到目标
主机.
+ 应用层: 负责应用程序间沟通，如简单电子邮件传输（SMTP）、文件传输协议（FTP）、网络远程访问
协议（Telnet）等. 我们的网络编程主要就是针对应用层.

物理层我们考虑的比较少. 因此很多时候也可以称为 TCP/IP四层模型.

一般而言

> + 对于一台主机, 它的操作系统内核实现了从传输层到物理层的内容;
> + 对于一台路由器, 它实现了从网络层到物理层;
> + 对于一台交换机, 它实现了从数据链路层到物理层;
> + 对于集线器, 它只实现了物理层;

但是并不绝对. 很多交换机也实现了网络层的转发; 很多路由器也实现了部分传输层的内容(比如端口转发);

## 数据包封装和分用

+ 不同的协议层对数据包有不同的称谓,在传输层叫做段(segment),在网络层叫做数据报 (datagram),在链
路层叫做帧(frame).
+ 应用层数据通过协议栈发到网络上时,每层协议都要加上一个数据首部(header),称为封装
(Encapsulation). 
+ 首部信息中包含了一些类似于首部有多长, 载荷(payload)有多长, 上层协议是什么等信息.
+ 数据封装成帧后发到传输介质上,到达目的主机后每层协议再剥掉相应的首部, 根据首部中的 "上层协议
字段" 将数据交给对应的上层协议处理

## 网络中的地址管理

**认识IP地址**

IP协议有两个版本, IPv4和IPv6. 我们整个的课程, 凡是提到IP协议, 没有特殊说明的, 默认都是指IPv4

+ IP地址是在IP协议中, 用来标识网络中不同主机的地址;
+ 对于IPv4来说, IP地址是一个4字节, 32位的整数;
+ 我们通常也使用 "点分十进制" 的字符串表示IP地址, 例如 192.168.0.1 ; 用点分割的每一个数字表示一个
字节, 范围是 0 - 255;

**认识MAC地址** 

+ MAC地址用来识别数据链路层中相连的节点;
+ 长度为48位, 及6个字节. 一般用16进制数字加上冒号的形式来表示(例如: 08:00:27:03:fb:19)
+ 在网卡出厂时就确定了, 不能修改. mac地址通常是唯一的(虚拟机中的mac地址不是真实的mac地址, 可
能会冲突; 也有些网卡支持用户配置mac地址)

中国是最多使用ipv6的国家