# Chapter 5 ：CPU　Scheduling



Basic Concepts: Maximum CPU utilization obtained with multiprogramming 

### CPU-bound and I/O-bound

A CPU-bound process might have a few very long CPU bursts. 

An I/O-bound process typically has many short CPU bursts

### CPU Scheduler 

The CPU scheduler selects a process from the **ready queue**, and allocates the CPU to it. 

### Circumstances that scheduling may take place

1. A process switches from the running state to the wait state (e.g., doing for I/O)
2.  process switches from the running state to the ready state (e.g., an interrupt occurs) 
3.  A process switches from the wait state to the ready state (e.g., I/O completion) 
4.  A process terminates

![1587083721108](C:\Users\kali_chenye\AppData\Roaming\Typora\typora-user-images\1587083721108.png)

### Preemptive vs. Non-preemptive

Non-preemptive scheduling: scheduling occurs when a process voluntarily enters the wait state (case 1) or terminates (case 4). Simple, but very inefficient 

Preemptive scheduling: scheduling occurs in all possible cases.  Mutual exclusion may be violated.  Mutual exclusion may be violated. 

实时OS一定要实现可抢占调度

### Dispatcher 派发器

Dispatcher module gives control of the CPU to the process selected by the short-term scheduler; this involves:

◆switching context 

◆switching to user mode

 ◆jumping to the proper location in the user program to restart that program

Dispatch latency – time it takes for the dispatcher to stop one process and start another running.上下文切换的时间,与硬件有关

### Scheduling Criteria

◆CPU utilization : Normally 40% is lightly loaded and 90% or higher is heavily loaded

◆Throughput :The number of processes completed per time unit ,Higher throughput means more jobs get done.对同样的进程集较为客观

◆Turnaround time :The time period between job submission to completion ,From a user’s point of view, turnaround time is more important than CPU utilization and throughput

◆Waiting time : the sum of the periods that a process spends waiting in the ready queue

◆Response time: The time from the submission of a request (in an interactive system) to the first response is called response time

### Test

![1587086738819](C:\Users\kali_chenye\AppData\Roaming\Typora\typora-user-images\1587086738819.png)

Answer: ABDE

判断process的类型是观察CPU burst的长度,Long CPU burst 是有较长的CPU时间,有很多Short CPU burst 的进行频繁的IO,IO burst的长度取决于设备,并不是因为是IO bound burst而变得很长

Jitter 抖动, 抖动越小可预测性越大

### Scheduling Algorithms

◆First-Come, First-Served (FCFS) 

◆Shortest-Job-First (SJF) 

◆Priority 

◆Round-Robin 

◆Multilevel Queue 

◆Multilevel Feedback Queue

#### FCFS

The process that requests the CPU first is allocated the CPU first. 

Using a queue. 

FCFS is not preemptive

**Convoy effect **:short process behind long process, CPU utilization may be low. 护航效应

#### SJF

When a process must be selected from the ready queue, the process with the smallest next CPU burst is selected. Thus, the processes in the ready queue are sorted in CPU burst length.

**SJF can be non-preemptive or preemptive**!!!

◆nonpreemptive 

◆preemptive – if a new process arrives with CPU burst length less than remaining time of current executing process, preempt.  This scheme is know as the **Shortest-Remaining-Time-First (SRTF). **

SJF is provably optimal(最优的) – gives minimum average waiting time for a given set of processes

#### How do we know the Next CPU Burst?

Without a good answer to this question, SJF cannot be used for CPU scheduling. 

We try to predict the next CPU burst  by using the length of previous CPU bursts, using exponential averaging.

how to do: 

![1587089504467](C:\Users\kali_chenye\AppData\Roaming\Typora\typora-user-images\1587089504467.png)



#### SJF Problems

some long jobs may not have a chance to run at all. **This is starvation.**

#### Priority Scheduling 

Priority may be determined internally or externally: 

◆internal priority: determined by time limits,memory requirement, # of files, and so on. (内部性质决定的)

◆external priority: not controlled by the OS (e.g., importance of the process) (人为设定的)

The scheduler always picks the process **(in ready queue) **with the highest priority to run

- Priority scheduling can be **non-preemptive or preemptive. **if the newly arrived process has a higher priority than the running one, the latter is preempted

Indefinite block (or starvation) may occur: a low priority process may never have a chance to run-->solution: Aging

#### Aging

Aging: gradually increases the priority of processes that wait in the system for a long time. 

#### Round Robin (RR) 时间片轮转

All processes in the ready queue is a FIFO list. When the CPU is free, the scheduler picks the first and lets it run for one time quantum. 

![1587090041429](C:\Users\kali_chenye\AppData\Roaming\Typora\typora-user-images\1587090041429.png)

- If time quantum is too large, RR reduces to FCFS 
- If time quantum is too small, RR becomes processor sharing 
- Context switching may affect RR’s performance .Shorter time quantum means more context switches 
- Turnaround time also depends on the size of time quantum. In general, 80% of the CPU bursts should be shorter than the time quantum 
- Typically, higher average turnaround than SJF, but better response.

#### Multilevel Queue 

Each process is assigned permanently to one queue based on some properties of the process .

Each queue has its own scheduling algorithm,

- foreground – RR 
- background – FCFS

![1587090988024](C:\Users\kali_chenye\AppData\Roaming\Typora\typora-user-images\1587090988024.png)

Fixed priority scheduling:  Possibility of starvation. 

Time slice:  each queue gets a certain amount of CPU time which it can schedule amongst its processes( i.e., 80% to foreground in RR, 20% to background in FCFS )

#### Multilevel Feedback Queue 

Multilevel queue with feedback scheduling is similar to multilevel queue; however, it allows processes to move between queues,**aging can be implemented this way **

If a process uses more (resp., less) CPU time, it is moved to a queue of lower (resp., higher) priority. 

That is to say : ,I/O-bound (resp., CPU-bound) processes will be in higher (resp., lower) priority queues.

![1587091212736](C:\Users\kali_chenye\AppData\Roaming\Typora\typora-user-images\1587091212736.png)

更确切的,Multilevel Feedback Queue 可以说是一种调度的框架

### Multiple-Processor Scheduling

CPU scheduling more complex when multiple CPUs are available.不做深入讨论

- Homogeneous processors 
- Load sharing 
- Asymmetric multiprocessing 

### Thread Scheduling

- Local Scheduling – threads library decides which thread to put onto an available LWP
- Global Scheduling –  kernel decides which kernel thread to run next



Classic Test：

Consider a multi-level feedback queue in a single-CPU system. The first level (queue 0) is given a quantum of 8 ms, the second one a quantum of 16 ms, the third is scheduled FCFS. Assume jobs arrive all at time zero with the following job times (in ms): 4, 7, 12, 20, 25 and 30, respectively. Assume the context switch overhead is zero unless otherwise stated. 

•Show the Gantt chart for this system.

 •Compute the average waiting and turnaround time. 

•Suppose the context switch overhead is 1 ms. Compute the average turnaround time



![1587138211994](C:\Users\kali_chenye\AppData\Roaming\Typora\typora-user-images\1587138211994.png)