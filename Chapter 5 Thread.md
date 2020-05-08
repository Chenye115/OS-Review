[OS Review Chapter1: Introduction](https://blog.csdn.net/qq_43721475/article/details/104834649)
[OS Review Chapter 2: Computer-System Structures](https://blog.csdn.net/qq_43721475/article/details/104836492)
[OS Review Chapter 3: Operating-System Structures](https://blog.csdn.net/qq_43721475/article/details/104846186)
[OS Review Chapter 4: Process](https://blog.csdn.net/qq_43721475/article/details/105397752)
[OS Review Chapter 5: Thread](https://blog.csdn.net/qq_43721475/article/details/105404697)

# Chapter 5 Thread

What is a thread?--A thread, also known as lightweight process (LWP),**is a basic unit of CPU execution.**

A thread has a thread ID, a program counter, a register set, and a stack--similar to a process

**However, a thread shares with other threads in the same process its code section, data section, and other OS resources (e.g., files and signals). **

![1586356695921](C:\Users\kali_chenye\AppData\Roaming\Typora\typora-user-images\1586356695921.png)

同一个进程的多个线程共享该进程的其他资源（不包括CPU、寄存器、栈）

Linux中不存在线程进程的概念，同一称作task

## Thread Usage

![1586357091134](C:\Users\kali_chenye\AppData\Roaming\Typora\typora-user-images\1586357091134.png)

## Benefits

- Responsiveness
- Resource Sharing (进程之间资源共享需要调用系统调用，需要频繁切换到kernel)
- Economy (创建时的开销)
- Utilization of MP(multi processors) Architectures

## User Threads

Thread management done by user-level threads library  --POSIX Pthreads 

用户线程阻塞会导致该进程的其他线程阻塞，kernel不知道process中的其他thread，认为这个process是阻塞的。 if one thread is blocked, every other threads of the same process are also blocked because the containing process is blocked.

User threads are supported at the user level. The kernel is not aware of user threads. 

A library provides all support for thread creation, termination, joining, and scheduling. There is no kernel intervention, and, hence, user threads are usually more efficient. 

![1586358017933](C:\Users\kali_chenye\AppData\Roaming\Typora\typora-user-images\1586358017933.png)

## Kernel Threads

Kernel threads are directly supported by the kernel. The kernel does thread creation,termination, joining, and scheduling in kernel space. 

Kernel threads are usually slower than the user threads.

通常是用户创建一个kernel thread，导致kernel需要进行系统调用陷入内核，创建完成后返回用户态

However, blocking one thread will not cause other threads of the same process to block. The kernel simply runs other threads. 

**In a multiprocessor environment, the kernel can schedule threads on different processors**

## Multithreading Models

### Many-to-One

 多对一模型：多个用户级线程线程映射到一个内核级线程。每个用户进程只对应一个内核级线程。
优点：用户级线程的切换在用户空间即可完成，不需要切换到核心态，线程管理的系统开销小，效率高。
缺点：当一个用户线程阻塞后，整个进程就会都阻塞，并发度不高。多个线程不可在多处理机上并行运行。 

### One-to-One

 一个用户进程对应一个内核进程。每个用户有与内核进程相同数量的用户进程。
优点：当一个线程被阻塞后，另外的线程还可以继续执行，并发性强。多线程可在多核处理机上并发执行
缺点：一个用户线程会占用多个内核级线程，线程切换是需要将用户态转换为核心态，因此线程管理的成本高，开销大。 

### Many-to-Many 

Allows many user level threads to be mapped to many kernel threads. Allows the  operating system to create a sufficient number of kernel threads. 

 多对多模型：n个用户级线程映射到m个内核级线程（n>=m）。每个用户进程对应m个内核级线程。
克服了多对一模型并发度不高的缺点，又克服了一对一模型中一个用户进程占用太多内核级线程，开销太大的缺点。 

## Threading Issues

Semantics of fork() and exec() system calls. ？？？

### Thread cancellation.

- **Asynchronous cancellation** terminates the target thread  immediately （实现过程简单）
- **Deferred cancellation **allows the target thread to periodically check if it should be cancelled ：The point a thread can terminate itself is a **cancellation point**（延迟，安全撤销）

Pthread supports deferred cancellation.

With asynchronous cancellation, if the target thread owns some system-wide resources, the system may not be able to reclaim all recourses. 

But for deferred cancellation,Reclaiming resources is not a problem.

## Signal Handling

## Thread Pools

Create a number of threads in a pool where they await work .

通常线程池中的线程数目是adaptive（自适应的）

Advantages: 

◆Usually slightly faster to service a request with an existing thread than create a new thread 

◆Allows the number of threads in the application(s) to be bound to the size of the pool

### Solaris 2 Threads

M:M

![1586396393345](C:\Users\kali_chenye\AppData\Roaming\Typora\typora-user-images\1586396393345.png)

## Windows XP Threads

Implements the one-to-one mapping. 

Each thread contains

- a thread id -
- register set 
- separate user and kernel stacks 
- private data storage area

![1586397994132](C:\Users\kali_chenye\AppData\Roaming\Typora\typora-user-images\1586397994132.png)

TEB: Thread Enviroment Block

## Linux Threads

linux中的进程已经十分高效，引入线程作为资源共享的一个手段

Linux refers to them as tasks rather than threads.

Thread creation is done through clone() system call. Clone() allows a child task to share the address space of the parent task (process)

## Windows Thread APIs

- CreateThread 
- GetCurrentThreadId - returns global ID 
- GetCurrentThread - returns handle 
- SuspendThread/ResumeThread 
- ExitThread 
- TerminateThread 
- GetExitCodeThread 
- GetThreadTimes

Example：

```c++
#include <iostream>
#include<Windows.h>
#include<synchapi.h>
BOOL thrdDone = FALSE;
using namespace std;

DWORD WINAPI Fibonacci()
{
    int num;
    cin >> num;
    int a = 1;
    int b = 1;
    int n = a+b;
    for (int i = 0; i < num; i++)
    {
        cout << i+1<<" : "<<a << endl;
        a = b;
        b = n;
        n = a + n;
    }
    thrdDone = TRUE;
    return 0;
}

int main()
{
    cout << "enter the number of the Fibonacci:  " <<endl;
    HANDLE hThread;
    hThread = CreateThread(NULL, 0, (LPTHREAD_START_ROUTINE)Fibonacci, NULL, 0, NULL);
    //getchar();
    while (!thrdDone);
    DWORD dw=WaitForSingleObject( hThread, 1000);
    //cout << dw;
    //std::cout << "Hello World!\n";
    system("pause");
}
```

Example Explained: Main thread is process When process goes, all threads go Need some methods of waiting for a thread to finish

### Threads states

pthread threads have two states

 ◆joinable and detached 

threads are joinable by default 

◆Resources are kept until pthread_join 

◆can be reset with attribute or API call 

detached thread can not be joined 

◆resources can be reclaimed at termination 

◆cannot reset to be joinable

```c
#include <stdio.h>
#include <pthread.h>

#define MAX_SEQUENCE 100

//一组数据，用于创建线程时作为参数传入
struct Data{
     int num; //斐波那契数列的项数
     int Fibo[MAX_SEQUENCE];//最大容量，斐波那契数列
};

void *Fibonacci(void *data){//获得斐波那契数列
    struct Data *tmp = (struct Data*)data;//转化为实际类型
    int i;

    if(  tmp->num == 0 ){
        printf("error!enter a number larger than 0");
    }
    if( tmp->num == 1 ){
        tmp->Fibo[0] = 0;
    }
    tmp->Fibo[0] = 0;
    tmp->Fibo[1] = 1;

    for( i=2; i < tmp->num; i++ ){
        tmp->Fibo[i] = tmp->Fibo[i-1] + tmp->Fibo[i-2]; 
    }
 }

int main(){
    struct Data data;
    pthread_t th;//线程标识符
    int ret; //pthread的返回值 ret = 0,创建线程成功
    int n;
    int i;

    printf("The fibonacci produce programe!\nPlease input an number within 1~200: ");
    scanf("%d", &n);
    if(n < 0 || n > 200){
        printf("Your input is error.");
        return -1;
    }
    data.num = n;//赋值


    //create a thread
    ret = pthread_create(&th, NULL, Fibonacci, (void *)&data);
    if( ret != 0 ){
        printf("Create thread error!\n");
        return -1;
    }

    //阻塞调用线程
    pthread_join( th, NULL);

    //输出斐波那契数列
    if( data.num == 0){
        printf("Nothing output.");
    }
    else{
	    for(int i=0;i<n;i++){
        printf("The Fibonacci items are:  ");
	printf("%d\n",data.Fibo[i]);
	    }
        }
        printf("\n");
 }

```

[OS Review Chapter1: Introduction](https://blog.csdn.net/qq_43721475/article/details/104834649)
[OS Review Chapter 2: Computer-System Structures](https://blog.csdn.net/qq_43721475/article/details/104836492)
[OS Review Chapter 3: Operating-System Structures](https://blog.csdn.net/qq_43721475/article/details/104846186)
[OS Review Chapter 4: Process](https://blog.csdn.net/qq_43721475/article/details/105397752)
[OS Review Chapter 5: Thread](https://blog.csdn.net/qq_43721475/article/details/105404697)