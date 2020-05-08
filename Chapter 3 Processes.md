# Chapter 3 Processes

引入进程---描述CPU的活动---研究CPU的活动---提高CPU的利用率

What is a Process-->程序的一次执行过程

Process – a program in execution; process execution must progress in sequential fashion. 

A process includes: 

◆program counter  程序执行到哪一步了

◆contents of the processor’s registers 状态

◆stack 局部变量

◆data section 全局变量

Process in Memory 进程空间

 ![img](C:\Users\kali_chenye\Desktop\96EBA89C2491F02B369B9C7CC1B05889.png) 

![1586341272971](C:\Users\kali_chenye\AppData\Roaming\Typora\typora-user-images\1586341272971.png)

text：程序（二进制）

heap：动态分配的内存

### Process State （进程的生命周期）

![1586341678517](C:\Users\kali_chenye\AppData\Roaming\Typora\typora-user-images\1586341678517.png)

### Process Control Block (PCB)

PCB存储的地方：内存 内核段

Information associated with each process：

 Process state

 Program counter 程序运行到哪 （context）

CPU registers  （**context**）

CPU scheduling information 

Memory-management information  程序访问的相关地址

Accounting information 审计CPU时间

 I/O status information

![1586342358312](C:\Users\kali_chenye\AppData\Roaming\Typora\typora-user-images\1586342358312.png)

### Process Scheduling Queues

Job queue – set of all processes in the system. 

Ready queue – set of all processes residing in main memory, ready and waiting to execute.所有能在CPu上执行的进程，被调度器调度的进程

 Device queues – set of processes waiting for an I/O device.处于waiting状态的 进程

== Process migration between the various queues.==

![1586347873354](C:\Users\kali_chenye\AppData\Roaming\Typora\typora-user-images\1586347873354.png)

## Schedulers

Long-term scheduler (or job scheduler) – selects which processes should be loaded into memory for execution. 决定进入队列的进程  (may be slow)The long-term scheduler controls **the degree of multiprogramming**.

Short-term scheduler (or CPU scheduler) – selects which process should be executed next and allocates CPU. 决定CPU执行的进程 (must be fast).

Processes can be described as either:

- I/O-bound process 
- CPU-bound process 

## Context Switch

What is a process context? 

The context of a process includes the values of CPU registers, the process state, the program counter, and other memory/file management information.

After the CPU scheduler selects a process (from the ready queue) and before allocates CPU to it, the CPU scheduler must :

- save the context of the currently running process
- put it into a queue
- load the context of the selected process
- let it run

## Operations on Processes 

### Process Creation

Parent process create children processes, which, in turn create other processes, forming a tree of processes. 

- Resource sharing 
- Execution
- **Address space**  子进程和父进程的地址空间不一样

![1586349412502](C:\Users\kali_chenye\AppData\Roaming\Typora\typora-user-images\1586349412502.png)

![1586349674675](C:\Users\kali_chenye\AppData\Roaming\Typora\typora-user-images\1586349674675.png)

```c
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>

int main(int argc,char *argv[])
{
	int i,id1,id2;
	for(i=1;i<2;i++)
	{
		id1=fork();
		id2=fork();
		if(id1==0||id2==0)
			fork();
	}
	printf(" I am %d\n",getpid());
}

```

```
 I am 3696
 I am 3697
 I am 3698
 I am 3699
 I am 3701
 I am 3702
 I am 3700
```

### Process Termination 

Process executes last statement and asks the operating system to delete it (exit).

- Output data from child to parent (via wait).
- Process’ resources are deallocated by OS.

Parent may terminate execution of children processes (abort). 

- Child has exceeded allocated resources.
- Task assigned to child is no longer required. 
- Parent is exiting. (Operating system does not allow child to continue if its parent terminates)

## Cooperating Processes

Advantages of process cooperation :

◆Information sharing

◆Computation speed-up

◆Modularity 

◆Convenience

**Producer-Consumer Problem**

