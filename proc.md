proc是linux系统下一个伪文件系统，挂载于/proc目录，它实际上是一个和内核内部数据结构交互的接口，它包含了很多系统运行的重要信息，linux的好多程序正确运行依赖了这个目录下的好多信息，同时它也可以用来在系统运行动态调整系统参数以改变系统行为。

收集系统信息
===========

系统相关
--------

/proc/meminfo

这个文件主要包含了系统内存使用的统计数据，像free命令就是读取的这个文件的数据来报告系统使用了多少内存，还有多少剩余，多少是buffered，多少是cached。文件中每一行的格式是属性名，冒号，属性值，值一般包括一个单位项（比如KB）。下面是一个属性名字的列表，说明了每一项的含义，以及格式，其中有些属性只有在内存配置了某些选项之后才会有。

MemTotal：物理内存使用总量（除掉一些保留内存和内核代码占用的）
MemFree：物理内存空闲量，两部分（LowFree + HighFree）
Buffers：相对来说临时占用的内存，不会太大，一般是文件元信息缓存
Cached：一般是读取文件在内存的缓存（Page Cache），不包括SwapCached
SwapCached：曾经被换出到交换文件，但是现在又换进内存的大小，但是同时这部分文件依然在交换文件中存在，如果内存压力大，需要再次换出的话，因为交换文件中已经存在，所以可以节省IO
Active：最近被使用到的内存
Inactive：最近未被使用的内存
HighTotal：大于860MB的内存总量
HighFree：大于860MB的内存空闲量
LowTotal：小于860MB的内存总量
LowFree：小于860MB的内存空闲量
SwapTotal：交换空间总量
SwapFree：交换空间空闲量
Dirty：需要写回到磁盘的内存量
WriteBack：正在写回磁盘的内存量
AnonPages：用户空间使用的内存页大小，但是不是文件缓存
Mapped：映射的内存大小，比如共享库
Slab：内核数据结构缓存
SReclaimable：Slab的一部分，能被回收再利用的
SUnreclaim：Slab的一部分，不能回收再利用
KernelStack：内核栈使用的大小
PageTables：页表使用的大小
NFS Unstable：网络文件系统来的数据，但还未写入到固定存储
Bounce：块设备使用的缓冲区
WritebackTmp：FUSE使用的临时写回缓冲区
CommitLimit：系统能分配的内存上限，由参数vm.overcommit_ratio控制。依据如下公式进行计算，总量＝（物理内存量－TLB内存量）* overcommit_ratio / 100 + 交换空间量。比如1GB内存，7GB交换空间，owercommit_ratio设置为30，则
CommitLimit＝1GB * 30 / 100 + 7GB＝7.3GB。
Committed AS：当前系统已经申请的内存总量，包括已经申请但未使用的量。
VmallocTotal：虚拟内存区域大小
VmallocUsed：已经使用的虚拟内存区域大小
VmallocChunk：最大连续空闲的虚拟内存区域


/proc/modules

当前被载入进内核的模块列表

/proc/cpuinfo

CPU和系统架构相关的信息，每个处理器一个列表。如果是多处理器的话，还有一个内核初始化时计算出来的常量，lscpu命令从此文件读取信息。

/proc/modules

当前被载入进内核的模块列表

进程相关
--------

对于每个进程，在/proc目录下都有一个以pid命名的子目录，保存了进程相关的一些信息。有一个self目录，这个目录会动态的指向正在读/proc目录的进程。

每个进程的目录下主要有这些项

clear_refs	清空在smpas中出现的页面引用位
cmdline		启动这个进程的命令行参数
cpu			当前或者上一次运行该进程的cpu
cwd			进程的工作目录
environ		环境变量
exe			进程的可执行文件
fd			打开的文件描述符
maps		内存映射表
mem			进程拥有的内存
root		进程的根目录
stat		进程统计信息
statm		进程内存统计信息
status		进程统计信息，可读性比stat强
wchan		进程睡眠时在内核中的位置
pagemap		页表
stack		进程运行栈
smaps		每一块内存消耗的统计

举例来说，想得到一个进程相关的信息，就读取/proc/<PID>/status

	cat /proc/self/status
	Name:	cat
	State:	R (running)
	Tgid:	1091
	Pid:	1091
	PPid:	1090
	TracerPid:	0
	Uid:	0	0	0	0
	Gid:	0	0	0	0
	FDSize:	32
	Groups:	0 1 2 3 4 6 10 19 
	VmPeak:	    4248 kB
	VmSize:	    4248 kB
	VmLck:	       0 kB
	VmPin:	       0 kB
	VmHWM:	     280 kB
	VmRSS:	     280 kB
	VmData:	     156 kB
	VmStk:	     136 kB
	VmExe:	      44 kB
	VmLib:	    1836 kB
	VmPTE:	      20 kB
	VmSwap:	       0 kB
	Threads:	1
	SigQ:	0/1879
	SigPnd:	0000000000000000
	ShdPnd:	0000000000000000
	SigBlk:	0000000000000000
	SigIgn:	0000000000000000
	SigCgt:	0000000000000000
	CapInh:	0000000000000000
	CapPrm:	ffffffffffffffff
	CapEff:	ffffffffffffffff
	CapBnd:	ffffffffffffffff
	Cpus_allowed:	1
	Cpus_allowed_list:	0
	Mems_allowed:	1
	Mems_allowed_list:	0
	voluntary_ctxt_switches:	0
	nonvoluntary_ctxt_switches:	0

这里看到的信息和用ps命令看到的差不多，实际上，ps命令就是读取/proc里面的文件，解析其中的字段来展示的。

status中的字段含义是这样的
name：当前运行的进程名
state：进程状态，可能是：运行中，睡眠中，磁盘睡眠，停止，被跟踪，僵尸，死亡
Tgid：线程组ID（进程PID）
Pid：线程ID
PPid：父进程PID
TracerPid：跟踪者进程的PID
Uid，Gid：真实的，有效的，文件系统的用户标识和组标识
FDSize：当前使用的文件描述符数目
Group：
VmPeak：用到的虚拟内存最大值
VmSize：虚拟内存大小
VmLck：锁住的内存大小
VmHWM：最大常驻内存大小
VmRSS： 常驻内存大小
VmData，VmStk，VmExe：数据段，堆栈段，代码段大小
VmLib：共享库大小
VmPTE：页表大小
Threads：进程包含的线程数
SigQ：用斜杠分隔的两个数字，一个表示当前排队等待的信号数，一个表示最大等待的信号数
SigPnd，ShdPnd：进程或者线程未处理的信号数
SigBlk，SigIgn，SigCgt：信号掩码，分别表示阻止，忽略，捕获的信号
Cpus allowd：CPU掩码，表示进程能在哪个CPU上运行
Cpus allowd list：和上面一样，不过表示能在哪些CPU上运行
Mems allowed：内存掩码，表示进程能使用哪些内存结点
Mems allowed list：和上面一样，不过表示能使用哪些内存结点
voluntary_context_switches，nonvoluntary_context_switches：自愿的上下文切换，非自愿的上下文切换


