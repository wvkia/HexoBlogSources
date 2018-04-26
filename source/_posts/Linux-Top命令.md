---
title: Linux Top命令
date: 2018-04-13 20:23:53
tags: [Linux]
---
#### Top命令

---

```aidl
top - 01:06:48 up  1:22,  1 user,  load average: 0.06, 0.60, 0.48
Tasks:  29 total,   1 running,  28 sleeping,   0 stopped,   0 zombie
Cpu(s):  0.3% us,  1.0% sy,  0.0% ni, 98.7% id,  0.0% wa,  0.0% hi,  0.0% si
Mem:    191272k total,   173656k used,    17616k free,    22052k buffers
Swap:   192772k total,        0k used,   192772k free,   123988k cached

PID     USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND
213     root      16   0  7976 2456 1980 S  0.7  1.3   0:11.03 sshd
43      root      16   0  2128  980  796 R  0.7  0.5   0:02.72 top
52      root      16   0  1992  632  544 S  0.0  0.3   0:00.90 init
12343   root      34  19     0    0    0 S  0.0  0.0   0:00.00 ksoftirqd/0
2122    root      RT   0     0    0    0 S  0.0  0.0   0:00.00 watchdog/0

```
前五行表示系统整体统计信息

1. 第一行显示：系统当前时间 系统运行时间  当前登录用户 系统负载
> 系统负载：(http://www.ruanyifeng.com/blog/2011/07/linux_load_average_explained.html)
>> 系统负载表示计算机CPU干活的能力，计算机想象成大桥，CPU想象成车道，进程相当于车，一个CPU表示只有一个车道，负载表示车在车道上的运行情况，如果
>> 没有车，负载为0，一大半车，负载为0.5，全是车，负载为1，但大桥还是可以畅通的，但如果是负载1.7，表示不仅车道的车满了，而且后续等待的车是桥面的0.7倍
>> 当系统负载大于1，后面的车就要等待。所以注意负载最好小于0.7。
>> 如果是多个CPU，表示桥有多个车道，可以容纳更多的车，负载也可以很大，比如2个CPU，意味着负载为2.0也没关系

2. 进程信息：进程总数    正在运行进程数 睡眠进程数   停止进程数   僵死进程数

3. Cpu信息：us 用户空间占CPU百分比，sy  内核空间占CPU百分比，ni 用户进程空间内改变过优先级的进程占用CPU百分比，id 空闲CPU百分比，wa 等待输入输出CPU时间百分比，hi 硬件中断百分比，si 软中断百分比，st 虚拟机占用百分比

4. 物理内存信息：total 物理内存总量，used 使用的物理内存总量，free 空闲内存总量，buffers 用作内核缓存的内存量

5. 虚拟交换内存信息：total 交换区总量，used 使用交换区量，free 空闲交换区量，cached 缓冲的交换区总量

6. 进程信息：
> PID进程id，USER运行用户，PR优先级，NI任务nice值，VIRT虚拟内存用量（=SWAP+RES），RES物理内存量，SHR共享内存，S进程状态，%CPU百分比，%MEM物理内存百分比

##### cached和buffers
Mem的buffers：缓冲区，存储速度不同步的设备之间传输的数据区域，是根据磁盘读写设计的，减少磁盘碎片和硬盘读取。
Swap的cached：高速缓存，用于CPU和内存之间的缓冲区域
简而言之：buffer是IO缓存，用于内存和硬盘缓冲；cached是高速缓存，用于CPU和内存之间的缓冲


##### 对内存进行分析
从上面Mem看的话，似乎free剩余只有17616k，已经有一大半被使用了，实际上对于Linux来说，内存是尽量使用的，尽可能的利用cached和buffer提高性能，但如果程序需要更多的内存，系统OS会把cached归还
而TOP命令反应系统OS的使用情况，对于cached和buffer它也是认为被使用，所以free是很少的

如果想要更准确的，可以使用 free 命令
#### free命令
```aidl
[root@scs-2 tmp]# free
             total       used       free     shared    buffers     cached
Mem:       3266180    3250004      16176          0     110652    2668236
-/+ buffers/cache:     471116    2795064
Swap:      2048276      80160    1968116
```
total：总计，used：已使用，free空闲，shared共享，buffersIO缓冲，cached高速缓冲
Mem表示从操作系统角度：

计算内存时：free命令是Men的free + buffers + cached
            top命令也是 free + buffers + cached