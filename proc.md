proc是linux系统下一个伪文件系统，挂载于/proc目录，它实际上是一个和内核内部数据结构交互的接口，它包含了很多系统运行的重要信息，linux的好多程序正确运行依赖了这个目录下的好多信息，同时它也可以用来在系统运行动态调整系统参数以改变系统行为。

收集系统信息
===========

系统相关
--------

/proc/meminfo

这个文件主要包含了系统内存使用的统计数据，像free命令就是读取的这个文件的数据来报告系统使用了多少内存，还有多少剩余，多少是buffered，多少是cached。文件中每一行的格式是属性名，冒号，属性值，值一般包括一个单位项（比如KB）。下面是一个属性名字的列表，说明了每一项的含义，以及格式，其中有些属性只有在内存配置了某些选项之后才会有。

MemTotal    ：物理内存使用总量（除掉一些保留内存和内核代码占用的）
MemFree     ：物理内存空闲量，两部分（LowFree + HighFree）
Buffers     ：相对来说临时占用的内存，不会太大，一般是文件元信息缓存
Cached      ：一般是读取文件在内存的缓存（Page Cache），不包括SwapCached
SwapCached  ：曾经被换出到交换文件，但是现在又换进内存的大小，但是同时这部分文件依然在交换文件中存在，如果内存压力大，需要再次换出的话，因为交换文件中已经存在，所以可以节省IO
Active      ：最近被使用到的内存
Inactive    ：最近未被使用的内存
HighTotal   ：大于860MB的内存总量
HighFree    ：大于860MB的内存空闲量
LowTotal    ：小于860MB的内存总量
LowFree     ：小于860MB的内存空闲量
SwapTotal   ：交换空间总量
SwapFree    ：交换空间空闲量
Dirty       ：需要写回到磁盘的内存量
WriteBack   ：正在写回磁盘的内存量
AnonPages   ：用户空间使用的内存页大小，但是不是文件缓存
Mapped      ：映射的内存大小，比如共享库
Slab        ：内核数据结构缓存
SReclaimable：Slab的一部分，能被回收再利用的
SUnreclaim  ：Slab的一部分，不能回收再利用
KernelStack ：内核栈使用的大小
PageTables  ：页表使用的大小
NFS Unstable：网络文件系统来的数据，但还未写入到固定存储
Bounce      ：块设备使用的缓冲区
WritebackTmp：FUSE使用的临时写回缓冲区
CommitLimit ：系统能分配的内存上限，由参数vm.overcommit_ratio控制。依据如下公式进行计算，总量＝（物理内存量－TLB内存量）* overcommit_ratio / 100 + 交换空间量。比如1GB内存，7GB交换空间，owercommit_ratio设置为30，则CommitLimit＝1GB * 30 / 100 + 7GB＝7.3GB。
Committed AS：当前系统已经申请的内存总量，包括已经申请但未使用的量。
VmallocTotal：虚拟内存区域大小
VmallocUsed ：已经使用的虚拟内存区域大小
VmallocChunk：最大连续空闲的虚拟内存区域

/proc/modules

当前被载入进内核的模块列表

/proc/cpuinfo

CPU和系统架构相关的信息，每个处理器一个列表。如果是多处理器的话，还有一个内核初始化时计算出来的常量，lscpu命令从此文件读取信息。

/proc/loadavg

前三个数是1分钟，5分钟和15分钟的系统负载，是通过计算在运行队列和IO等待队列的进程得到的，和uptime命令的输出一样的。第四个值是当前运行的任务和系统总任务数，第五个是系统最常运行的进程标识（PID）。

/proc/modules

当前被载入进内核的模块列表

/proc/kallsyms

所有被内核导出的符号，modules系列命令会用到，2.5.47以前叫ksyms。

/proc/filesystems

列出了内核支持的文件系统列表，就是被编译进内核的，或者被编译成内核模块但是目前载入进内核的。如果一个文件系统有nodev标志，那表示这个文件系统不需要一个块设备，一般是用于虚拟文件系统，或者网络文件系统。

这个文件有可能会被mount命令用到，当mount执行的时候没有指明文件系统名字，mount无法判断是哪个文件，就会尝试这个文件中列出的文件系统。

/proc/config.gz

这个文件包含了当前运行的内核的配置选项，和编译内核时的.config文件格式一样，只有配置了内核选项CONFIG_IKONFIG_PROC才有这个文件。这个文件是压缩过的,可以使用zcat或者zgrep来查看其内容。这个文件和这个命令得到的内容一致，cat /lib/modules/$(uname -r)/build/.config。

进程相关
--------

对于每个进程，在/proc目录下都有一个以pid命名的子目录，保存了进程相关的一些信息。有一个self目录，这个目录会动态的指向正在读/proc目录的进程。

每个进程的目录下主要有这些项

clear_refs	清空在smaps中出现的页面引用位  
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
limits		资源限制  
oom_score_adj		OOM的时候计算分数调整limits		资源限制  

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

进程统计信息还有一个文件，/proc/[pid]/stat，ps命令会用到这个文件。这个文件是一组数字，各个字段的含义如下：

pid：进程ID
comm：可执行文件名字，如果被交换出去的话，则名字不可见
state：进程目前所处的状态，取值是“RSDZTW”这几个字母之一一。R是运行，S是睡眠可中断，D是不可中断等待磁盘，Z是僵尸，T是被跟踪或者停止态。
ppid：父进程ID
pgrp：父进程组ID
session：会话ID
tty_nr：进程的控制终端
tpgid：进程控制终端的前台进程组ID
flags：进程的内核标记
minflt：
cminflt：
majflt：
cmajflt：
utime：进程在用户态消耗的时间
stime：进程在内核态消耗的时间
cutime：进程的子进程在用户态消耗的时间
cstime：进程的子进程在内核态消耗的时间
priority：进程调度优先级。实时策略，-2到-100；非实时策略，0到39
nice：进程优先级调整，-20到19
num_threads：进程包含的线程数
itrealvalue：2.6.17版本后固定为0
starttime：进程启动时间
vsize：进程使用的虚拟内存大小
rss：进程占用的在内存中的资源大小，包含代码段，数据段，堆栈，不包括换出的。
rsslim：进程的资源软限制
startcode：进程代码可执行起始地址
endcode：进程代码可执行结束地址
startstack：堆栈的起始地址（栈底）
kstkesp：ESP寄存器的当前值
kstkeip：EIP寄存器的当前值
signal：未处理的信号（已经废弃）
blocked：阻止的信号（已经废弃）
sigignore：忽略的信号（已经废弃）
sigcatch：捕捉到的信息（已经废弃）
wchan：进程等待的内核地址
nswap：换出的页面数(不再维护)
cnswap：子进程累计的换出页面数(不再维护)
exit_signal：进程死掉时发送给父进程的信号
processor：上一次运行的CPU
rt_priority：实时调度策略下的优先级，1到99的数字。
policy：调度策略
delayacct_blkio_ticks：累计IO延迟
guest_time：为guest os运行花费的时间
cguest_tie：子进程为guest_os运行花费的时间
