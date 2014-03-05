proc是linux系统下一个伪文件系统，挂载于/proc目录，它实际上是一个和内核内部数据结构交互的接口，它包含了很多系统运行的重要信息，linux的好多程序正确运行依赖了这个目录下的好多信息，同时它也可以用来在系统运行动态调整系统参数以改变系统行为。

收集系统信息
===========

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


