
# 7 Synchronization Examples

## 7.1 Classic Problems of Synchronization

### 7.1.1 The Bounded-Buffer Problem

我们讨论用 Semaphores 解决 Bounded-Buffer Problem / Producer-Consumer Problem：

Two processes, the producer and the consumer share n buffers, the producer generates data, puts it into the buffer; the consumer consumes data by removing it from the buffer.

The problem is to make sure: the producer won't try to add data into the buffer if it is full; the consumer won't try to remove data from an empty buffer.

  

首先我们尝试使用一个 `lock`  和一个 `eslot` (empty slot，空闲 buffer 的个数) 来解决：

```
Semaphore lock = 1;
Semaphore eslot = N;

// Producer:
do {
    wait(eslot);	// will make eslot--
    wait(lock);
    produce();
    signal(lock);
} while (true);
    
// Consumer:
do {
    signal(eslot);	// will make eslot++
    wait(lock);
    consume();
    signal(lock);
} while (true);
```

我们会发现，这种方式不能够满足要求，因为当 buffer 为空，即 eslot 为 N 时，consumer 并不会不运行，因为 signal 并不作判断。因此，我们需要一个额外的 semaphore `fslot`  (full slot) 来解决这个问题：

```
Semaphore lock = 1;
Semaphore eslot = N;
Semaphore fslot = 0;

// Producer:
do {
    wait(eslot);	// will make eslot--
    wait(lock);
    produce();
    signal(lock);
    signal(fslot);	// will make fslot++
} while (true);
    
// Consumer:
do {
    wait(fslot);	// will make fslot--
    wait(lock);
    consume();
    signal(lock);
    signal(eslot);	// will make eslot++
} while (true);
```

  

### 7.1.2 The Readers-Writers Problem

对一些数据，readers 只能读取，而 writers 可以读和写。设计方案保证：多个 readers 可以同时读取，但是 writer 进行写时不能有其他 writers 和 readers。

解决方法：

```
Semaphore rcnt_mutex = 1;
Semaphore rw_mutex = 1;
int reader_count = 0;

// Writer
do {
    wait(rw_mutex);
    
    read_and_write();
    
    signal(rw_mutex);
} while (true);

// Reader
do {
    wait(rcnt_mutex);			// 保证 reader_count 的同步
    reader_count++;
    if (reader_count == 1)
        wait(rw_mutex);			// 第一个 reader 出现时拿走 rw_mutex
    signal(rcnt_mutex);
    
    read();
    
    wait(rcnt_mutex);			// 保证 reader_count 的同步
    reader_count--;
    if (reader_count == 0)
        signal(rw_mutex);		// 全部 reader 退出时释放 rw_mutex
    signal(rcnt_mutex);
}
```

这种解决策略可能会导致 writer 的 starvation。

  

### 7.1.3 Dining-Philosophers Problem

每两个哲学家之间有一根筷子，每个人一次可以拿起来一根筷子，拿到两根筷子的就可以吃一段时间。吃完思考一段时间。

![](https://cdn.nlark.com/yuque/0/2021/png/641515/1610672853220-24ba826e-8a4b-4bdc-9879-cd88cc9c5504.png)

![](https://cdn.nlark.com/yuque/0/2021/png/641515/1610673950624-d50a7587-9ae1-497d-8907-398adfc80030.png)![](https://cdn.nlark.com/yuque/0/2021/png/641515/1610673972039-98f38896-c8bc-4e8d-bfe7-679d60428bcd.png)

![](https://cdn.nlark.com/yuque/0/2021/png/641515/1610674117857-602b10b2-6205-4649-b147-7e937cdb60e7.png)

![](https://cdn.nlark.com/yuque/0/2021/png/641515/1610674110296-0fdb6b00-8c5d-4bb7-96be-ada474710173.png)

![](https://cdn.nlark.com/yuque/0/2021/png/641515/1610674130285-51ceb2f1-39f5-423e-92be-e6d8ad925f5e.png)

![](https://cdn.nlark.com/yuque/0/2021/png/641515/1610674139875-9ea82099-e2f3-4c63-af34-a206ca2cab68.png)

# 8 Deadlock

## 8.3 Deadlock Characterization

### 8.3.1 Necessary Conditions

当下面四个条件同时成立时，系统会出现死锁：

1. **Mutual exclusion** : 至少一个资源处于非共享模式；
2. **Hold and wait** : 一个进程应 **占有** 至少一个资源，并 **等待** 另一个为其他进程占有的资源；
3. **No preemption** : 资源不能被抢占，只能在进程结束后主动释放；
4. **Circular wait** : 有一组等待进程 {T0, T1, ..., Tn}，T0 is waiting for a resource held by T1, T1 is waiting for a resource held by T2, ..., Tn−1 is waiting for a resource held by Tn, and Tn is waiting for a resource held by T0.

这四个条件并不完全独立。