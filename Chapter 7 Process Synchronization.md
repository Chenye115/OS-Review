# Chapter 6:  Process Synchronization

在distribute system分布式系统中进程同步尤其重要

### Background

we modify the producer-consumer code by adding a variable counter-->读写冲突

The statements   counter++; counter--;  must be performed **atomically**

Atomic operation means an operation that completes in its entirety without interruption

**Race Condition**： occurs

- two or more processes/threads access and manipulate the same data concurrently
- the outcome of the execution depends on the particular order in which the access takes place

Race Condition引入了不确定性，To prevent race conditions, concurrent processes must be synchronized

### The Critical-Section Problem 临界区问题

n processes all competing to use some shared data .Each process has a code segment, called critical section, in which the shared data is accessed. 临界区是代码段

![1588838519241](C:\Users\kali_chenye\AppData\Roaming\Typora\typora-user-images\1588838519241.png)

#### Critical Section and Mutual Exclusion 相互排斥

The critical-section problem is to design a protocol that processes can use to cooperate.

#### The Critical Section Protocol

- A critical section protocol consists of two parts: an entry section and an exit section.
- Between them is the critical section that must run in a mutually exclusive way.

#### Solution to Critical-Section Problem 三个条件

- **Mutual Exclusion 互斥**：If a process P is executing in its critical section, then no other processes can be executing in their critical sections. when the process that is executing in its critical section exits, the entry protocol must be able to know this fact and allows a waiting process to enter.
- **Progress 进程 有空让进**：If no process is executing in its critical section and some processes wish to enter their critical sections-->Only those processes that are waiting to enter can participate in the competition ,No other process can influence this decision,This decision cannot be postponed indefinitely
- **Bounded Waiting 有限等待**：there exists a bound on the number of times that other processes are allowed to enter.  even though a process may be blocked by other waiting processes, it will not be waiting forever

#### Algorithm 1 

```
Shared variables:
int turn;
initially turn=i(or turn=j)

Process Pi:
do{
	while(turn!=i);
	critical section
	turn=j;
	remainder section;
}while(1);
```

***Satisfies mutual exclusion, but not progress***

#### Algorithm 2 

```
Shared variables:
boolean flag[2];
initially flag[0]=flag[1]=false;
flag [i] = true // Pi ready to enter its critical section 
Process Pi 
do {
	flag[i] = true; 
	while (flag[j]); 
	critical section flag [i] = false; 
	remainder section } while (1); 
```

***Satisfies mutual exclusion, but not progress requirement***  while flag[0],flag[1]=true

```
do {
	while (flag[j]) ;
    flag[i] = true; 
    critical section flag [i] = false; 
    remainder section } while (1);
```

***not mutual exclusion***

#### Algorithm 3 

Combined shared variables of algorithms 1 , 2

```
Process Pi 
do { 
	flag [i]= true; 
	turn = j; 
	while (flag [j] and turn == j) ;
    critical section flag [i] = false; 
    remainder section } while (1); 
```

***Meets all three requirements; solves the critical section problem for two processes.***

#### Bakery Algorithm

```
Shared data 
	boolean choosing[n]; 
	int number[n];
do { 
	choosing[i] = true; 
	number[i] = max(number[0], number[1], …, number [n – 1])+1;
    choosing[i] = false; 
    for (j = 0; j < n; j++) 
    { 
    	while (choosing[j]) ; 
    	while ((number[j] != 0) && ((number[j],j) <  (number[i],i))) ; 
    } 
    critical section number[i] = 0; 
    remainder section 
} while (1);
```

 

### Synchronization Hardware 

There are two types of hardware synchronization supports:

- Disabling/Enabling interrupts(关中断 开中断)：This is slow and difficult to implement on multiprocessor systems. （多处理系统上难以实现）
- Special machine instructions：Test and set (TS)   Swap

#### Test-and-Set

Test and modify the content of a word atomically 

```
boolean TestAndSet(boolean &target) { 
	boolean rv = target; 
	target = true; 
	return rv; 
	}
Shared data:
	boolean lock = false;
Process Pi 
	do {
    	while (TestAndSet(lock)) ; 
    	critical section 
    	lock = false; 
    	remainder section 
    }
```

#### Swap

```
Shared data (initialized to false):
	boolean lock; 
local variable 
	boolean key; 
Process Pi 
	do { 
		key = true; 
		while (key == true) 
			Swap(lock,key); 
		critical section 
		lock = false; 
		remainder section 
		}
```

该算法不保证bounded waiting

#### Bounded Waiting Mutual Exclusion with TestAndSet

![1588901743663](C:\Users\kali_chenye\AppData\Roaming\Typora\typora-user-images\1588901743663.png)

### Semaphores

Synchronization tool that does not require busy waiting

**Semaphore S – integer variable but can only be accessed via two indivisible (atomic) operations** 

![1588902142250](C:\Users\kali_chenye\AppData\Roaming\Typora\typora-user-images\1588902142250.png)

信号量的第一种用法：解决互斥 Critical Section of n Processes

```
 Shared data: semaphore mutex; //initially mutex = 1
 Process Pi: 
	do { 
		wait(mutex); 
		critical section 
		signal(mutex); 
		remainder section 
	} while (1);
```

mutex : mutual exclusion

信号量的第二种用法：同步 Semaphore as a General Synchronization Tool

![1588902455121](C:\Users\kali_chenye\AppData\Roaming\Typora\typora-user-images\1588902455121.png)

#### **Semaphore Implementation**

```
Define a semaphore as a record 
typedef struct { 
	int value;
    struct process *L; 
   } semaphore;
   
```

![1588902700681](C:\Users\kali_chenye\AppData\Roaming\Typora\typora-user-images\1588902700681.png)

**block** suspends the process that invokes it.

**wakeup(P)** resumes the execution of a blocked process **P**.

```
Semaphore operations now defined as
	wait(S): 
		S.value--; 
		if (S.value < 0) { 
			add this process to S.L; 
			block;
            } 
        signal(S): 
        	S.value++; 
        	if (S.value <= 0) {
            remove a process P from S.L;
            wakeup(P);
            }
```

#### Example

Explain why spinlocks are not appropriate for single-processor systems,yet are not used in multiprocessor systems.

```
在单处理机环境中可以使用硬件提供的swap指令或test_and_set指令实现进程互斥，这些指令涉及对同一存储单元的两次或两次以上操作，这些操作将在几个指令周期内完成，但由于中断只能发生在两条机器指令之间，而同一指令内的多个指令周期不可中断，从而保证swap指令或test_and_set指令的执行不会交叉进行.
但在多处理机环境中情况有所不同，例如test_and_set指令包括“取”、“送”两个指令周期，两个CPU执行test_and_set(lock)可能发生指令周期上的交叉，假如lock初始为0, CPU1和CPU2可能分别执行完前一个指令周期并通过检测(均为0)，然后分别执行后一个指令周期将lock设置为1，结果都取回0作为判断临界区空闲的依据，从而不能实现互斥. 
```

### Classical Problems of Synchronization

***Bounded-Buffer Problem***

```Shared data
semaphore full, empty, mutex;
Initially:
full = 0, empty = n, mutex = 1
```

```
do { … 
	produce an item in nextp
    … 
    wait(empty);
    wait(mutex);
    … 
    add nextp to buffer
    … 
    signal(mutex); 
    signal(full); 
    } while (1);
```

```
do {
	wait(full) 
	wait(mutex);
	… 
	remove an item from buffer to nextc
	…
	signal(mutex);
	signal(empty);
	… 
	consume the item in nextc …
	} while (1);
```

***Readers-Writers Problem***

读者优先  写者优先

```Shared data
semaphore mutex, wrt;
Initially
mutex = 1, wrt = 1, readcount = 0
```

```
wait(mutex);
readcount++; 
if (readcount == 1) 
	wait(wrt);
signal(mutex);
	…
reading is performed
	… 
wait(mutex);
readcount--;
if (readcount == 0) 
signal(wrt);
signal(mutex):
```



***Dining-Philosophers Problem***

```
Philosopher i:
	do { 
		wait(chopstick[i])
        wait(chopstick[(i+1) % 5])
        … 
        eat
        … 
        signal(chopstick[i]);
        signal(chopstick[(i+1) % 5]);
        …
        think
        … 
      } while (1);
```

### Monitors 

High-level synchronization construct that allows the safe sharing of an abstract data type among concurrent processes. 

```
monitor monitor-name
{ 	shared variable declarations
procedure body P1 (…) { . . .} 
procedure body P2 (…) { . . .} 
procedure body Pn (…) { . . .} 
{ initialization code}
}
```

![1588906784823](C:\Users\kali_chenye\AppData\Roaming\Typora\typora-user-images\1588906784823.png)

Monitors: Mutual Exclusion

-  No more than one process can be executing within a monitor. Thus, mutual exclusion is guaranteed within a monitor. 

-  When a process calls a monitor procedure and enters the monitor successfully, it is the only process executing in the monitor. 

- When a process calls a monitor procedure and the monitor has a process running, the caller will be blocked outside of the monitor.

![1588923437769](C:\Users\kali_chenye\AppData\Roaming\Typora\typora-user-images\1588923437769.png)