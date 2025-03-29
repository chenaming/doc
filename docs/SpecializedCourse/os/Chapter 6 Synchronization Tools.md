
我们说 **Cooperating Process** 是可以影响系统中其他运行进程或被其他进程影响的进程。

A **cooperating process** is one that can affect or be affected by other processes executing in the system.


Cooperating processes 会共同使用一些数据，可能是直接使用同一段地址空间（代码+数据），或者是通过共享的内存或信息传递来共用一些数据。对数据的同时访问 (concurrent access) 可能会导致 data inconsistency，因为数据的一致性需要 cooperating processes 有序的运行。看下面一个例子：

# 6.1 Background

- Processes can execute concurrently(进程调度程序切换进程以便提供并发执行)
	- May be interrupted at any time, partially completing execution
- Concurrent access to shared data may result in data inconsistency
- Maintaining data consistency requires mechanisms to ensure the orderly execution of cooperating processes
- Illustration of the problem:  
	- Suppose that we wanted to provide a solution to the consumer-producer problem that fills **all** the buffers. We can do so by having an integer counter that keeps track of the number of full buffers.  Initially, counter is set to 0. It is incremented by the producer after it produces a new buffer and is decremented by the consumer after it consumes a buffer.
- 我们现在回到有界缓冲区的问题。正如已指出的，原来的解决方案允许缓冲区同时最多只有BUFFER_SIZE-1项。假如我们想要修改这一算法以便弥补这个缺陷。
- 一种可能方案是，增加一个整型变量counter，并且初始化为0。每当向缓冲区增加一项时，递增counter；每当缓冲区移走一项时，递减counter。

```
/* Bounded-buffer Problem */

/* Producer Process */
while (true) {
    /* produce an item in next_produced */
    while (count == BUFFER_SIZE)
    	; /* do nothing */
    buffer[in] = next_produced;
    in = (in + 1) % BUFFER_SIZE;
    count++;
}

/* Consumer Process */
while (true) {
    while (count == 0)
    	; /* do nothing */
    next_consumed = buffer[out];
    out = (out + 1) % BUFFER_SIZE;
    count--;
    /* consume the item in next_consumed */
}
```

Race Condition

- counter++ could be implemented as  
  
     register1 = counter  
     register1 = register1 + 1  
     counter = register1

- counter-- could be implemented as  
  
     register2 = counter  
     register2 = register2 - 1  
     counter = register2

Consider this execution interleaving with “count = 5” initially:

  S0: producer execute register1 = counter         {register1 = 5}  
  S1: producer execute register1 = register1 + 1   {register1 = 6}  
  S2: consumer execute register2 = counter        {register2 = 5}  
  S3: consumer execute register2 = register2 – 1  {register2 = 4}  
  S4: producer execute counter = register1         {counter = 6 }  
  S5: consumer execute counter = register2        {counter = 4}



出现这个问题，是因为我们允许两个进程同时操控变量 `count` 。类似这样的多个进程同时操控同一个数据，因而结果取决于每一种操控的出现顺序的情形，称为 **race condition**。为了防止 race condition，我们需要保证同一时间只有一个进程可以操控某个变量。

A situation where several processes access and manipulate the same data concurrently and the outcome of the execution depends on the particular order in which the access takes place, is called a **race condition**.


Race condition 在操作系统中是常见的。Kernel code 中也包含 race condition 的可能性。如下例：


![](https://cdn.nlark.com/yuque/0/2020/png/641515/1606542148202-b7f2337c-060a-41af-ae79-4adb551e5149.png)

两个进程 P0 和 P1 同时 fork() 时，如果不加限制，可能会出现类似前例的情况，即在某一个进程把当前的 `next_avaliable_pid` 分配给他的 child 后，在没来得及更新 `next_avaliable_pid` 前，另一个进程使用了 `next_avaliable_pid` 来给 child 分配 PID，这就会导致两个不同的线程使用同一个 PID 的情况。

process synchronization
process coordination




# 6.2 Critical Section Problem


Critical Section Problem

- Consider system of n processes {p0, p1, … pn-1}
- Each process has critical section segment of code
	- Process may be changing common variables, updating table, writing file, etc
	- When one process in critical section, no other may be in its critical section
- Critical section problem is to design protocol to solve this
- Each process must ask permission to enter critical section in ==entry section==, may follow critical section with ==exit section==, then ==remainder section==


![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/5c8340a639b04c78b921c64f4ed8f1ab.png#pic_center)


Solution to Critical-Section Problem


1.   Mutual Exclusion - If process Pi is executing in its critical section, then no other processes can be executing in their critical sections

2.   Progress - If no process is executing in its critical section and there exist some processes that wish to enter their critical section, then the selection of the processes that will enter the critical section next cannot be postponed indefinitely

3.  Bounded Waiting -  A bound must exist on the number of times that other processes are allowed to enter their critical sections after a process has made a request to enter its critical section and before that request is granted
	- Assume that each process executes at a nonzero speed
	- No assumption concerning relative speed of the n processes





在任一给定时间点，一个操作系统可能具有多个处于内核态的活动进程。因此，操作系统的实现代码（内核代码）可能出现竞争条件。例如，有一个内核数据结构链表，用于维护打开系统内的文件。当打开或关闭一个新文件时，应更新这个链表（向链表增加一个文件，或从链表中删除一个文件）。如果两个进程同时打开文件，那么这两个独立的更新操作可能产生竞争条件。其他导致竞争条件的内核数据结构包括维护内存分配、维护进程列表及中断处理等的数据结构。内核开发人员应确保，操作系统没有这些竞争条件。

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/9b834d29c8e94cc981e2f5703cb78c52.png#pic_center)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/eede856b70194ce3ab354b8b55c85103.png#pic_center)



Critical-Section Handling in OS

对于单核系统，我们可以通过在 critical section 中禁止中断（即，在 entry section 中 disable，在 exit section 中 enable）的方式来实现上述功能（虽然可能是危险的）。但是对于多核系统，中断禁止的消息要传到所有处理器，消息传递会延迟进入临界区，会降低效率；同时也会影响时钟中断。

- Single-core system: preventing interrupts
- Multiple-processor:preventing interrupts are not feasible
- Two approaches depending on if kernel is preemptive or non-preemptive
   抢占式内核与非抢占式内核
	- Preemptive – allows preemption of process when running in kernel mode
	- Non-preemptive – runs until exits kernel mode, blocks, or voluntarily yields CPU
		- Essentially free of race conditions in kernel mode


显然，非抢占式内核的数据结构基本不会导致竞争条件，因为在任一时间点只有一个进程处于内核模式。然而，对于抢占式内核，就不这样简单了；这些抢占式内核需要认真设计，以便确保内核数据结构不会导致竞争条件。对于SMP体系结构，抢占式内核更难设计，因为在这些环境下两个处于内核态的进程可以同时运行在不同处理器上。

那么，为何会有人更喜欢抢占式内核而不是非抢占式内核呢？抢占式内核响应更快，因为处于内核模式的进程在释放CPU之前不会运行任意长的时间。（当然，在内核模式下持续运行很长时间的风险可以通过设计内核代码来最小化。）再者，抢占式内核更适用于实时编程，因为他能允许实时进程抢占在内核模态下运行的其他进程。本章后面将探讨多种操作系统如何管理抢占式内核。

# 6.3 Peterson’s Solution

- software-based solution
- Good algorithmic  description of solving the problem
- Two process solution
- Assume that the load and store machine-language instructions are atomic; that is, cannot be interrupted
- The two processes share two variables:
	- int turn;
	- Boolean flag[2]
- The variable turn indicates whose turn it is to enter the critical section
- The **flag** array is used to indicate if a process is ready to enter the critical section. ==flag[i] = true==  implies that process Pi is ready!


Algorithm for Process Pi
```c
while(true){
	flag[i] = true;
	turn = j;
	while (flag[j] && turn == j)
		;
		
		critical section
		
	flag[i] = false;
	
		remainder section
}
```


Provable that the three  CS requirement are met:

        1.   Mutual exclusion is preserved
                Pi enters CS only if:
                      either flag[j] = false or turn = i
        2.   Progress requirement is satisfied
        3.   Bounded-waiting requirement is met



咸鱼暄


```
int turn;			// Who is allowed to enter
boolean flag[2];	// Ready to enter its CS

void foo() {
    while (true) {
        flag[i] = true; 	// Mark self ready
        turn = 1 - i;		// Assert that if the other process wishes to 
        					// enter its CS, it can do so.
        while (flag[1 - i] && turn == 1 - i);	// Wait
        /* critical section */
        flag[i] = false;	// Set ready to false
        /* remainder section */
    }
}
```

其中， `i` 是 0 或 1，表示第 i 个进程； `turn` 是当前有权进入 critical section 的进程（0 或 1）； `flag[i]` 是第 i 个进程是否准备好进入 critical section，初始值均为 FALSE。

To enter the critical section, process Pi first sets `flag[i]` to be true and then sets `turn` to the value `1-i` (the other process), thereby asserting that if the other process wishes to enter the critical section, it can do so. If both processes try to enter at the same time, `turn` will be set to both `0` and `1` at roughly the same time. Only one of these assignments will last; the other will occur but will be overwritten immediately. The eventual value of `turn` determines which of the two processes is allowed to enter its critical section first.

我们可以通过简易的分类讨论证明 Peterson's Solution 满足 6.2 中提到的三个性质：Mutual exclusion, process and bounded waiting。

但实际上，Peterson's solution 在现代计算机体系结构上不一定适用，因为现代的处理器和编译器有可能会为了优化性能而对一些读写操作进行重排。在优化中，处理器或编译器会考虑其重排的合理性，即保证了在单线程程序中结果值是稳定且正确的。但是这不能保证其在多线程共用数据时的正确性，重排可能会导致不稳定或者不期望的输出。

![](https://cdn.nlark.com/yuque/0/2020/png/641515/1606573574517-4bfd5351-5ded-480b-992f-f048d520f027.png)

![](https://cdn.nlark.com/yuque/0/2020/png/641515/1606573588613-5d1f5ac4-e5ca-41a8-846c-ee30a57a97f0.png)

Note that reordering of memory accesses can happen even on processors that don't reorder instructions ([Src](https://en.wikipedia.org/wiki/Peterson%27s_algorithm#Note))

[N process Peterson algorithm - GeeksforGeeks](https://www.geeksforgeeks.org/n-process-peterson-algorithm/)



# 6.4 Hardware Support for Synchronization

如上文，software-based 解决方案（它没有使用操作系统或某些硬件操作来保证 mutual exclusion）不能保证在现代计算机体系结构上适用。因此，我们试图寻找一些基于硬件的操作来帮助解决 critial-section problem。

## 6.4.0 Memory Models

没看懂

How a computer architecture determines what memory guarantees it will provide to an application program is known as its **memory model**. In general, a memory model falls into one of two categories:

1. **Strongly ordered**, where a memory modification on one processor is immediately visible to all other processors.
2. **Weakly ordered**, where modifications to memory on one processor may not be immediately visible to other processors.

**Memory model** are the memory guarantees a computer architecture makes to application programs.

## 6.4.1 Memory Barriers

部分内容参考 [https://blog.csdn.net/caoshangpa/article/details/78853919](https://blog.csdn.net/caoshangpa/article/details/78853919)。

如我们之前所说，编译器和处理器会对代码的结构进行 reorder，以达到最佳效果。例如：

```
int x = 1;
int y = 2;
int a1 = x * 1;
int b1 = y * 1;
int a2 = x * 2;
int b2 = y * 2;
// 可能会优化为：
int x = 1;
int y = 2;
int a1 = x * 1;
int a2 = x * 2;
int b1 = y * 1;		// a2, b1 的顺序进行了重排
int b2 = y * 2; 
```

对 a2 和 b1 进行重排，使得不需反复读取交替 x 和 y 值。

在运行时，CPU 虽然会乱序执行指令，但是在单个 CPU 的上，硬件能够保证程序执行时所有的内存访问操作看起来像是按程序代码编写的顺序执行的，这时候 Memory Barrier 没有必要使用（不考虑编译器优化的情况下）。这里我们了解一下 CPU 乱序执行的行为。在乱序执行时，一个处理器真正执行指令的顺序由可用的输入数据决定，而非程序员编写的顺序。

早期的处理器为有序处理器（In-order processors），有序处理器处理指令通常有以下几步：

- 指令获取
- 如果指令的输入操作对象（input operands）可用（例如已经在寄存器中了），则将此指令分发到适当的功能单元中。如果一个或者多个操作对象不可用（通常是由于需要从内存中获取），则处理器会等待直到它们可用
- 指令被适当的功能单元执行
- 功能单元将结果写回寄存器堆（Register file，一个 CPU 中的一组寄存器）

相比之下，乱序处理器（Out-of-order processors）处理指令通常有以下几步：

- 指令获取
- 指令被分发到指令队列
- 指令在指令队列中等待，直到输入操作对象可用（一旦输入操作对象可用，指令就可以离开队列，即便更早的指令未被执行）
- 指令被分配到适当的功能单元并执行
- 执行结果被放入队列（而不立即写入寄存器堆）

只有所有更早请求执行的指令的执行结果被写入寄存器堆后，指令执行的结果才被写入寄存器堆（执行结果重排序，让执行看起来是有序的）

从上面的执行过程可以看出，乱序执行相比有序执行能够避免等待不可用的操作对象（有序执行的第二步）从而提高了效率。现代的机器上，处理器运行的速度比内存快很多，有序处理器花在等待可用数据的时间里已经可以处理大量指令了。

现在思考一下乱序处理器处理指令的过程，我们能得到几个结论：

- 对于单个 CPU 指令获取是有序的（通过队列实现）
- 对于单个 CPU 指令执行结果也是有序返回寄存器堆的（通过队列实现）

由此可知，在单 CPU 上，不考虑编译器优化导致乱序的前提下，多线程执行不存在内存乱序访问的问题。

  

诸如此类的优化使得程序在运行时的实际内存访问与程序代码编写的访问顺序不一定一致。但是如 6.3 中所提到的，这种重排可能使得在多核运行时出现与期望不同的结果。为了解决这个问题，我们引入 **Memory Barrier**：它用来保证其之前的内存访问先于其后的完成。即，我们保证在此前对内存的改变对其他处理器上的进程是可见的。如 6.3 提出的简单例子：

![](https://cdn.nlark.com/yuque/0/2020/png/641515/1606579394097-e92e7a8a-2787-4780-a9fa-b1ad04587e7a.png)

Note that memory barriers are considered very low-level operations and are typically only used by kernel developers when writing specialized code that ensures mutual exclusion.


## 6.4.2 Hardware Instructions

许多现代系统提供硬件指令，用于检测和修改 word 的内容，或者用于 atomically（uniterruptably，不可被打断地） 交换两个 word。这里，我们不讨论特定机器的特定指令，而是通过指令 `test_and_set()` 和 `compare_and_swap()` 抽象了解这些指令背后的主要概念。

Synchronization Hardware

- Many systems provide hardware support for implementing the critical section code.
- All solutions below based on idea of ==locking==
	- Protecting critical regions via locks
- Uniprocessors – could disable interrupts
	- Currently running code would execute without preemption
	- Generally too inefficient on multiprocessor systems
		- Operating systems using this not broadly scalable
- Modern machines provide special atomic hardware instructions
		- ==Atomic== = non-interruptible
	- Either test memory word and set value
	- Or swap contents of two memory words

---

Solution to Critical-section Problem Using Locks

```c
do {
	acquire lock
		critical section
	release lock
		remainder section
} while (TRUE);

```

---

### 6.4.2.1 test_and_set()

test_and_set  Instruction

Definition:
```
boolean test_and_set(boolean* target)
{
    boolean rv = *target;
    *target = TRUE;
    return rv:
}
```

1.Executed atomically
2.Returns the original value of passed parameter
3.Set the new value of passed parameter to “TRUE”.

---

Solution using test_and_set()

- Shared Boolean variable lock, initialized to FALSE

Solution:

```
while (true) {
    /* Entry Section */
    while (test_and_set(&lock)) 	
        ; /* do nothing */
   	
    /* Critical Section */
    
    /* Exit Section */
    lock = false;
    
    /* Remainder Section */
}
```

可见，如果 `lock` 在 Entry Section 时为 true，那么 `test_and_set(&lock)` 将返回 true，因此会始终在 while 循环中询问。直到某个时刻 `lock` 为 false，那么 `test_and_set(&lock)` 将返回 false 同时将 `lock` 置为 true，进程进入 Critical Section，同时保证其他进程无法进入 Critical Section。当持锁的进程完成 Critical Section 的运行，它在 Exit Section 中释放 `lock` ，从而允许其他进程进入 Critical Section。

如果某个时刻 `lock` 为 false，而有两个或多个进程几乎同时调用了 `test_and_set(&lock)` 。但由于它是 atomic 的，因此只有一个进程可以返回 false。

  

但是，如上所示的控制不能满足 bounded waiting 条件：

![](https://cdn.nlark.com/yuque/0/2020/png/641515/1606583543590-e3e70d63-2cf9-4a6f-8d4f-61b6e4dabbd5.png)

我们可以作如下更改以满足 bounded waiting：

```
while (true) {
    /* Entry Section */
    waiting[i] = true;
    while (waiting[i] && test_and_set(&lock)) 	
        ; /* do nothing */
   	waiting[i] = false;
    
    /* Critical Section */
    
    /* Exit Section */
    j = (i + 1) % n;
    while ((j != i) && !waiting[j]))
        j = (j + 1) % n;
    if (j == i)
        lock = false;
    else
        waiting[j] = false;
    
    /* Remainder Section */
}
```

我们引入了 bool 数组 `waiting[]` 。在 Entry Section 中，我们首先置 `waiting[i]` 为 true；当 `waiting[i]` 或者 `lock` 中任意一个被释放时，进程可以进入 Critical Section。初始时， `lock` 为 false，第一个请求进入 CS 的进程可以获许运行。在 Exit Section 中，进程从下一个进程开始，遍历一遍所有进程，发现正在等待的进程时释放它的 `waiting[j]` ，使其获许进入 CS，当前进程继续 Remainder Section 的运行；如果没有任何进程在等待，那么它释放 `lock` ，使得之后第一个请求进入 CS 的进程可以直接获许。

这样的方式可以保证每一个进程至多等待 n-1 个进程在其前面进入 CS，满足了 bounded waiting 条件。



---

### 6.4.2.2 Compare_and_Swap()

compare_and_swap Instruction

Definition:
```
int compare_and_swap(int* value, int expected, int new_value) {
    int temp = *value;

    if (*value == expected)
        *value = new_value;
    return temp;
}
```

1.Executed atomically

2.Returns the original value of passed parameter “value”

3.Set  the variable “value”  the value of the passed parameter “new_value” but only if 
“value” \==“expected”. That is, the swap takes place only under this condition.


---

Solution using compare_and_swap

Shared integer  “lock”  initialized to 0;

Solution:

```
while (true) {
    /* Entry Section */
    while (compare_and_swap(&lock, 0, 1) != 0) 	
        ; /* do nothing */
   	
    /* Critical Section */
    
    /* Exit Section */
    lock = 0;
    
    /* Remainder Section */
}
```

可见，`compare_and_swap()` 和 `test_and_set()` 没有本质区别。上例 `compare_and_swap()` 的使用方法同样无法保证 bounded waiting，我们可以使用与 `test_and_set()` 同样的方式来解决。

  

在 Intel x86 架构中，汇编指令 `cmpxchg`  用于实现 `compare_and_swap()` 指令；但是不保证是 atomic 的（因为最初用于单核）。我们可以增加前缀 `lock`  来强制实现 atomic。

`lock cmpxchg <destination operand>, <source operand>` 

  

## 6.4.3 Atomic Variables

如我们先前所说，之前介绍的指令常被用来作为同步工具的组成部分而不是直接使用，我们可以使用 `compare_and_swap()` 指令来实现一些工具。其中一个工具就是 **Atomic Variable**。

如同我们在 6.1 开始提到的问题那样，一个变量在更新的过程中可能会导致一个 race condition。Atomic Variable 可以为数据提供 atomic updates。例如，我们使用不可打断的 `increment(&count);` 指令来代替可被打断的 `count++` 指令就可以解决 6.1 中的 Bounded-buffer Problem。

我们可以如下设计 `increment()` 函数：

```
void increment(atomic_int *v) {
    int temp;
    do {
        temp = *v;
    } while (temp != compare_and_swap(v, temp, temp+1));
}
```

注意到，程序循环尝试将 `v` 赋值为 `temp+1` ，当赋值成功时返回。由于 CAS 指令是 atomic 的，因此它不会在运行过程中被打断；在程序其他运行过程中 `v` 的值都没有发生改变。

但是需要注意的是，如果 buffer 有两个 consumer 在同时等待读取，那么当 `count` 由 0 变成 1 的时候两个 consumer 可能会同时进入来读取，但是实际上只有 1 个值在 buffer 中。即，Atomic Variable 并不能解决所有 race condition，因为它解决的问题仅是变量更新过程中的 race condition。

# 6.5 Mutex Locks


（MUTEX - MUTual EXclusion）

我们尝试设计软件工具来解决 CS problem。我们考虑让进程在 Entry Section 申请 `acquire()` 一个锁，然后在 Exit Section `release()` 一个锁。对于这个锁，我们用一个布尔变量来表示它是否 `avaliable` ：


- Previous solutions are complicated and generally inaccessible to application programmers
- OS designers build software tools to solve critical section problem
- Simplest is mutex lock
- Protect a critical section  by first ==acquire()== a lock then ==release()== the lock
	- Boolean variable indicating if lock is available or not
- Calls to ==acquire()== and ==release()== must be atomic
	- Usually implemented via hardware atomic instructions
- But this solution requires busy waiting
	- This lock therefore called a spinlock


acquire() and release()


```
while (true) {
    acquire();
    /* critical section */
    release();
    /* remainder section */
}

/* ------- */
void acquire() {
    while (!available)
        ; /* busy waiting */
    avaliable = false;
}

void release() {
    avaliable = true;
}
```


我们需要保证 `acquire()` 和 `release()` 是 atomic 的。我们可以使用 `test_and_set()` 和 `compare_and_swap()` 来实现：

```
void acquire() {
    while (compare_and_swap(&available, 1, 0) != 1)
        ; /* busy waiting */
}

void release() {
    available = true;
}
```

但是这种实现的缺点是，它需要 **busy waiting**，即当有一个进程在临界区中时，其他进程在请求进入临界区时在 acquire() 中持续等待，

例如当两个进程同时使用一个 CPU 时：

T0 acquires lock -> INTERRUPT-> T1 runs, spin, spin spin … (till time's out) -> INTERRUPT-> T0 runs -> INTERRUPT->T1 runs, spin, spin spin … -> INTERRUPT-> T0 runs, release locks -> INTERRUPT -> T1 runs, enters CS

可以发现，T1 在它的 CPU 时间内不断循环等待，直到 T0 释放锁。因此这种锁也成为 **spinlock**。可以想象，如果有 N 个进程同时使用一个 CPU，那么将有大约 ![](https://cdn.nlark.com/yuque/__latex/5cc07649359b61b52158b63470984d21.svg) 的时间被浪费。我们称一个锁 **contended**（被争夺），如果有进程在企图 acquire 它时被阻止；反之我们称它 **uncontended**。如我们所述，highly contended locks 会降低当前运行程序的整体性能。

但是，spinlocks 也有其优势：当进程在等待锁时，不需要 context switch，而 context switch 通常需要不短的时间。

我们还可以考虑下面的解法：

```
void acquire() {
    while (compare_and_swap(&avaliable, 1, 0) != 1)
        yield(); 
}

void release() {
    avaliable = true;
}
```

其中 `yield()` 会使程序从 running 转为 ready，从而让出 CPU。

Mutex locks 通常被认为是最简单的 synchronization tool。

# 6.6 Semaphores


- Synchronization tool that provides more sophisticated ways (than Mutex locks)  for process to synchronize their activities.
- Semaphore **S** – integer variable
- Can only be accessed via two indivisible (atomic) operations
	- **wait()** and **signal()**
		- Originally called P() and V()


- Definition of  the wait() operation
```c
wait(S) {
    while (S <= 0)
        ; // busy wait
    S--;
}
```

- Definition of  the signal() operation

```c
signal(S) {
    S++;
}
```


## 6.6.1 Semaphore Usage


- ==Counting semaphore== – integer value can range over an unrestricted domain
- ==Binary semaphore== – integer value can range only between 0 and 1
	- Same as a mutex lock
- Can solve various synchronization problems
- Consider P1  and P2 that require S1 to happen before S2

       Create a semaphore “synch” initialized to 0
		P1:
		   S1;
		   signal(synch);
		P2:
		   wait(synch);
		   S2;

- Can implement a counting semaphore S as a binary semaphore



## 6.6.2 Semaphore Implementation

- Must guarantee that no two processes can execute  the **wait()** and **signal()** on the same semaphore at the same time
- Thus, the implementation becomes the critical section problem where the **wait** and **signal** code are placed in the critical section
	- Could now have ==busy waiting== in critical section implementation
		- But implementation code is short
		- Little busy waiting if critical section rarely occupied
- Note that applications may spend lots of time in critical sections and therefore this is not a good solution

---

Semaphore Implementation with no Busy waiting

- With each semaphore there is an associated waiting queue
- Each entry in a waiting queue has two data items:
	- value (of type integer)
	- pointer to next record in the list
- Two operations:
	- ==block== – place the process invoking the operation on the appropriate waiting queue
	- ==wakeup== – remove one of processes in the waiting queue and place it in the ready queue

```c
typedef struct {
	int value;
	struct process* list;
} semaphore;

wait(semaphore* S) {
    S->value--;
    if (S->value < 0) {
        add this process to S->list;
        block();
    }
}

signal(semaphore* S) {
    S->value++;
    if (S->value <= 0) {
        remove a process P from S->list;
        wakeup(P);
    }
}

```


操作 `block()` 挂起调用它的进程，操作 `wakeup(P)` 重新启动 P 的执行，这两个操作都是由操作系统作为基本系统调用提供的。

The list of waiting processes can be easily implemented by a link field in each process control block (PCB). Each semaphore contains an integer value and a pointer to a list of PCBs. One way to add and remove processes from the list so as to ensure bounded waiting is to use a FIFO queue, where the semaphore contains both head and tail pointers to the queue. In general, however, the list can use any queuing strategy. Correct usage of semaphores does not depend on a particular queuing strategy for the semaphore lists.

需要重申的是， `wait()` 和 `signal()` 应该是 atomic 的。

In a multicore environment, interrupts must be disabled on every processing core. Otherwise, instructions from different processes (running on different cores) may be interleaved in some arbitrary way. Disabling interrupts on every core can be a difficult task and can seriously diminish performance. Therefore, SMP systems must provide alternative techniques—such as `compare_and_swap()` or spinlocks— to ensure that `wait()` and `signal()` are performed atomically.

下面这段没看懂

It is important to admit that we have not completely eliminated busy waiting with this definition of the `wait()` and `signal()` operations. Rather, we have moved busy waiting from the entry section to the critical sections of application programs. Furthermore, we have limited busy waiting to the critical sections of the `wait()` and `signal()` operations, and these sections are short (if properly coded, they should be no more than about 10 instructions). Thus, the critical section is almost never occupied, and busy waiting occurs rarely, and then for only a short time. An entirely different situation exists with application programs whose critical sections may be long (minutes or even hours) or may almost always be occupied. In such cases, busy waiting is extremely inefficient.

  

但是，semaphore 可能会导致 deadlock：

![](https://cdn.nlark.com/yuque/0/2020/png/641515/1606637016834-083d56ca-fd9f-449d-8f3b-8fd1db4b908f.png)

# 6.7 Monitors

## 6.7.1 Monitor Usage

## 6.7.2 Implementing a Monitor Using Semaphores



## 6.7.3 Resuming Processes within a Monitor



# 6.8 Liveness

## 6.8.1 Deadlock


Deadlock and Starvation

- ==Deadlock== – two or more processes are waiting indefinitely for an event that can be caused by only one of the waiting processes
- Let S and Q be two semaphores initialized to 1

          P0                              P1
            wait(S);                 wait(Q);
             wait(Q);                 wait(S);
		...       ...
             signal(S);                 signal(Q);
              signal(Q);                 signal(S);

- ==Starvation – indefinite blocking ==
	- A process may never be removed from the semaphore queue in which it is suspended
- ==Priority Inversion== – Scheduling problem when lower-priority process holds a lock needed by higher-priority process
	- Solved via ==priority-inheritance protocol==

## 6.8.2 Priority Inversion



![](https://cdn.nlark.com/yuque/0/2020/png/641515/1606637206616-b1f5d0fd-158f-4266-a80b-8e5fed3b5a09.png)

这个问题称为 **priority inversion**，即具有中等优先级的 M 的运行时间反而影响了具有较高优先级的 H 的等待时间。我们可以通过优先级继承 **priority inheritance** 来解决这一问题：所有正在访问资源（如上例中，低优先级的 L 所持的锁）的进程获得需要访问它的更高优先级进程的优先级，直到它们用完有关资源为止。（如上例中，priority inheritance 将允许 L 临时继承 H 的优先级从而防止被 M 抢占；当 L 释放锁后则回到原来的优先级，此时 H 将在 M 之前执行。）

PRIORITY INVERSION AND THE MARS PATHFINDER

![](https://cdn.nlark.com/yuque/0/2020/png/641515/1606637564619-ce1c1fdb-ae3c-4107-85e6-2c61892f712f.png)

  
