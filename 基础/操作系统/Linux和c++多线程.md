---
title: Linux和c++多线程
toc: true
date: 2022-02-22 15:26:49
tags:
- 多线程
categories:
- Linux 
---

###### Linux多线程

- 定义

  - 线程的join（联合的）和detch（分离的）；1）当使用join()函数时，主调线程(main函数里有一个主调线程)阻塞，等待被调线程终止，然后主调线程回收被调线程资源，并继续运行；上面这段话的意思就是，使用join(),线程运行完,main函数才能结束。（2）当使用detach()函数时，主调线程继续运行，被调线程驻留后台运行，主调线程无法再取得该被调线程的控制权。当主调线程结束时，由运行时库负责清理与被调线程相关的资源。上面这段话的意思就是，使用detach(),main函数不用等待线程结束才能结束。有时候线程还没运行完，main函数就已经结束了。
  - xxx

- 自旋锁

  - 定义：自旋锁的目的是将一个并行化的数据串行化，自旋锁的循环指令标识cpu忙等待，在等待的内核路径上无事可做的时候，其也会在cpu上运行（浪费cpu的时间），【volatile标识当前修改的值立马更新到内存中】自旋锁是为了保证临界区代码

  <!-- more -->

  - spin_lock相当于get_and_set判断是否获取并设置成功，

  - 这里有两个常用的方法可以使用：`TAS（test and set）`和`CAS （compare and swap）`。

    - TAS：一个TAS指令包括两个子步骤，把给定的内存地址设置为1，然后返回之前的旧值。

    - `CAS`：CAS指令需要三个参数，一个内存位置(V)、一个期望旧值(A)、一个新值(B)。过程如下：

      a. 比较内存V的值是否与A相等？
      b. 如果相等，则用新值B替换内存位置V的旧值
      c. 如果不相等，不做任何操作。
      d. 无论哪个情况，CAS都会把内存V原来的值返回。

      TAS是直接写操作，CAS是先比较，满足条件后再写。而`写`是一个相对耗时的操作，因此在高并发、频繁使用锁的场景，CAS性能会更好。

    很多语言都提供了封装后的TAS和CAS调用方法。

    1. 以C++ 11为例，`atomic`标准库提供了相关方法：[std::atomic_flag::test_and_set](https://en.cppreference.com/w/c/atomic/atomic_flag_test_and_set)和[std::atomic::compare_exchange_strong](http://www.cplusplus.com/reference/atomic/atomic/compare_exchange_strong/)
    2. GCC编译器也内置了相关方法：`__atomic_test_and_set`和`__atomic_compare_exchange_n`.
    3. Java也提供了例如`java.util.concurrent.atomic.AtomicReference.compareAndSet`等方法。

  - api：

    ```c++
    spinlock_t lock;
    spin_lock_init(spinlock_t *lock);
    spin_lock(spinlock_t *lock);
    spin_unlock(spinlock_t *lock);
    spin_trylock(spinlock_t *lock);
    ```

  - 使用：

  - 注意点：由于自旋锁，一直会去申请

- 互斥量

  - 定义：主要用来互斥访问共享资源，当一个线程拿到互斥锁，其他线程再去拿就会陷入阻塞等待状态，只有这个互斥锁释放，才会唤醒其他线程，那个线程获得互斥锁取决于线程的优先级和cpu的调度算法,一般是先等待的先获得；互斥量的状态，0、1或者其他负数，初始化为1【/* 1: unlocked, 0: locked, negative: locked, possible waiters */】，其中有个进程（线程队列等待中）。
  - api:init、lock、unlock、trylock、destory、
  - 使用：互斥锁是一种建议锁，如果进程中有多线程访问，必须按照加锁、处理、解锁的步骤，不然还是会导致数据错乱。当一个线程lock了互斥量，当另外一个线程去lock的时候，这个线程会阻塞，让出cpu；互斥量的使用流程应该是：线程占用互斥量，然后访问共享资源，最后释放互斥量。
  - 优缺点：所有的线程都去锁这个互斥量（锁），
  - 注意点：互斥锁不能进行递归锁定或者解锁；一个互斥锁对象必须通过其API初始化，而不能使用memset或复制初始化；一个任务在持有互斥锁的时候是不能结束的；互斥锁所使用的内存区域是不能被释放的。使用中的互斥锁是不能被重新初始化的。并且互斥锁不能用于中断上下文。但是互斥锁比当前的内核信号量选项更快，并且更加紧凑，

- 条件变量

  - 定义：利用线程间共享的全局变量进行同步的一种机制；包括两个动作：等待条件变量成立的而被挂起（阻塞？）；条件变量成立后唤醒（发送一个信号）其他线程；用来等待;

  - api:

    ```c++
    
    pthread_cond_wait(pthread_cond_t* cond, pthread_mutex_t* mutex);
    pthread_cond_timedwait(pthread_cond_t *__restrict __cond,
                                       pthread_mutex_t *__restrict __mutex,
                                       __const struct timespec *__restrict __abstime);
    
    pthread_cond_signal(pthread_cond_t* cond);
    pthread_cond_broadcast(pthread_cond_t* cond);
    pthread_cond_destory(pthread_cond_t* cond);
    pthread_cond_init(pthread_cond_t* cond, void* ptr);
    ```

  - 使用：在wait的时候必须用while循环（不用if)；在signal的时候必须加互斥量；wait 函数会先将线程放在线程等待队列，然后将互斥量（互斥锁）解锁。

  - 注意点：条件变量的使用必须配合互斥量；唤醒丢失;惊群现象(只给一个线程（优先级和最老的）发，不给其他线程发）；在生产端加互斥量是为了保证原子性，变量的改变和唤醒是一个原子操作；

- 信号量

  - 定义：临界区是更新数据的代码需要独占式的执行；二进制信号量和通用信号量；P(sv)：如果sv的值大于零，就给它减1；如果它的值为零，就挂起该进程的执行；V(sv)：如果有其他进程因等待sv而被挂起，就让它恢复运行，如果没有进程因等待sv而挂起，就给它加1.

  - api：

    ```c++
    int semget(key_t key,int num_sems,int sem_flags)；
    int semctl(int sem_id,int sem_num,int command,[union semun sem_union]);
    int semop(int sem_id,struct sembuf *sem_opa,size_t num_sem_ops);
    ```

  - 使用：信号量是一种特殊的变量，访问具有原子性。
    只允许对它进行两个操作：
    1)等待信号量
    当信号量值为0时，程序等待；当信号量值大于0时，信号量减1，程序继续运行。
    2)发送信号量
    将信号量值加1。

  - 注意点：信号量是不知道属主的，互斥量是知道的

- 读写锁

  - 定义：读模式下加锁、写模式下加锁、不加锁

  - api：pthread_rwlock_init；pthread_rwlock_destory;

    ```c++
    
    /* 初始化读写锁属性对象 */
    int pthread_rwlockattr_init (pthread_rwlockattr_t *__attr);
     
    /* 销毁读写锁属性对象 */
    int pthread_rwlockattr_destroy (pthread_rwlockattr_t *__attr);
     
    /* 获取读写锁属性对象在进程间共享与否的标识*/
    int pthread_rwlockattr_getpshared (__const pthread_rwlockattr_t * __restrict __attr,
                                              int *__restrict __pshared);
    /* 设置读写锁属性对象，标识在进程间共享与否  */
    int pthread_rwlockattr_setpshared (pthread_rwlockattr_t *__attr, int __pshared);
     /*返回值：成功返回0，否则返回错误代码*/
    ```

    ```c++
    
    /* 读模式下加锁  */
    int pthread_rwlock_rdlock (pthread_rwlock_t *__rwlock);
     
    /* 非阻塞的读模式下加锁  */
    int pthread_rwlock_tryrdlock (pthread_rwlock_t *__rwlock);
     
    # ifdef __USE_XOPEN2K
    /*  限时等待的读模式加锁 */
    int pthread_rwlock_timedrdlock (pthread_rwlock_t *__restrict __rwlock,
                                           __const struct timespec *__restrict __abstime);
    # endif
     
    /* 写模式下加锁  */
    int pthread_rwlock_wrlock (pthread_rwlock_t *__rwlock);
     
    /* 非阻塞的写模式下加锁 */
    int pthread_rwlock_trywrlock (pthread_rwlock_t *__rwlock);
     
    # ifdef __USE_XOPEN2K
    /* 限时等待的写模式加锁 */
    int pthread_rwlock_timedwrlock (pthread_rwlock_t *__restrict __rwlock,
                                           __const struct timespec *__restrict __abstime);
    # endif
     
    /* 解锁 */
    int pthread_rwlock_unlock (pthread_rwlock_t *__rwlock);
     /*返回值：成功返回0，否则返回错误代码*/
    ```

  - 使用：只要没有写模式下的加锁，任意线程都可以进行读模式下的加锁；只有读写锁处于不加锁状态时，才能进行写模式下的加锁；

  - 注意点

- 死锁

  - 定义
  - 解决方案
  - 其他

- 无锁编程

  - cas: compare and swap (一个旧值和一个新值，旧值相当则进行操作，不想当不操作）

  - 优点：

  - 1. 没有提升性能
    2. 避免死锁
    3. 

- 样例

  - 读者写者问题

  ```c++
  
  #include <iostream>
  #include<stdio.h>
  #include<pthread.h>
  #include<unistd.h>
  using namespace std;
  
  
  int buf = 0;  //buf全局变量就是临界资源。
  pthread_rwlock_t rwlock;
  void *reader(void *arg)
  {
      while(1) {
          /*if(pthread_rwlock_tryrdlock(&rwlock)) {
              std::cout << "continue try read lcok" << endl;
              std::cout << "reader do othrer thing!" << endl;
              continue;
  
          } else {
              buf++;
              std::cout << "read buf:" << buf << endl;
              pthread_rwlock_unlock(&rwlock);
          }*/
          pthread_rwlock_rdlock(&rwlock);
          std::cout << "read buf:" << buf << endl;
          pthread_rwlock_unlock(&rwlock);
          usleep(90);
      }
  }
  
  void *writer(void *arg) {
      while(1) {
          /*if (pthread_rwlock_trywrlock(&rwlock)) {
              std::cout << "continue try read lcok" << endl;
              std::cout << "reader do othrer thing!" << endl;
              continue;
          } else {
              buf++;
              std::cout << "write buf:" << buf << endl;
              pthread_rwlock_unlock(&rwlock);
          }*/
          pthread_rwlock_wrlock(&rwlock);
          buf++;
          std::cout << "write buf:" << buf << endl;
          pthread_rwlock_unlock(&rwlock);
          usleep(100);
      }
  }
  
  int main()
  {
      pthread_rwlock_init(&rwlock,NULL);
       pthread_t r[5],w;
       pthread_create(&r[0],NULL,reader,NULL);
       pthread_create(&r[1],NULL,reader,NULL);
       pthread_create(&r[2],NULL,reader,NULL);
       pthread_create(&r[3],NULL,reader,NULL);
       pthread_create(&r[4],NULL,reader,NULL);
       pthread_create(&w,NULL,writer,NULL);
  
       int i=0;
       for(i=0;i<5;i++)
       {
           pthread_join(r[i],NULL);
       }
       pthread_join(w,NULL);
       pthread_rwlock_destroy(&rwlock);
       return 0;
  
  
  }
  ```

- 生产者消费者

  ```c++
  #include <iostream>
  
  #include <queue>
  #include <cstdlib>
  
  #include <unistd.h>
  #include <pthread.h>
  
  using namespace std;
  
  pthread_mutex_t mutex1;
  pthread_cond_t cond;
  pthread_cond_t cond1;
  queue<int> product;
  
  void* produce(void *ptr) {
      int i = 0;
      while(1) {
          pthread_mutex_lock(&mutex1);
          while(product.size() > 100) {
              pthread_cond_wait(&cond1, &mutex1);
          }
          product.push(i);
          i++;
          cout << "produce:" << i << endl;
  //pthread_cnod_singal(&cond, &mutex);
          if (product.size() >= 20) {
              pthread_cond_signal(&cond);
          }
          pthread_mutex_unlock(&mutex1);
      }
  }
  void* consume(void* ptr) {
      int i = 0;
      while(1) {
          pthread_mutex_lock(&mutex1);
          while(product.size() < 20) {
              pthread_cond_wait(&cond, &mutex1);
          }
          /*if(product.empty()) {
              pthread_cond_wait(&cond, &mutex);
          }*/
          cout<<"consume:"<<product.front()<<endl;
          product.pop();
          if (product.size() < 100) {
              pthread_cond_signal(&cond1);
          }
          pthread_mutex_unlock(&mutex1);
      }
  }
  int main()
  {
      pthread_mutex_init(&mutex1, NULL);
      pthread_cond_init(&cond, nullptr);
      pthread_cond_init(&cond1, nullptr);
      pthread_t tid1, tid2;
      pthread_create(&tid1, NULL, consume, NULL);
      pthread_create(&tid2, NULL, produce, NULL);
      void *retVal;
      pthread_join(tid1, &retVal);
      pthread_join(tid2, &retVal);
      return 0;
  }
  ```

  ##### c++11 多线程

  ###### std::thread

  在传入线程函数的参数的时候，我们可以通过std::ref()和move来解决引用没有传递的情况；

- Lock(unique_lock和guard_lock)

  - unique_lock和guard_lock都是通过构造函数加锁，析构函数解锁

  - unique_lock比guard_lock大一点，占用的空间多并且慢一点是因为要维护一个状态（locked,unlocked）

  - unique_lock通过构造不同的构造函数来实现对mutex的所有权，具体看相关的api；unique_lock可以进行移动

    ```c++
    #include <unistd.h>
    #include <iostream>
    #include <thread>
    #include <mutex>
    #include <condition_variable>
    #include <cstdlib>
    
    static const int K_ITEMS_SIZE = 10;
    static const int K_ITEMS_MAX = 1000;
    
    struct ItemRepository {
        int items[K_ITEMS_SIZE];
        std::mutex mutex;
        std::condition_variable not_empty;
        std::condition_variable full;
        int consumer_pos;
        int producer_pos;
    };
    
    void produce_items(ItemRepository* ir, int item) {
        std::unique_lock<std::mutex> lock(ir->mutex);
        //if() 和while()的区别
        while (((ir->producer_pos + 1) % K_ITEMS_SIZE) == ir->consumer_pos) {
            std::cout << "Producer is waiting for an empty slot..." << std::endl;
            ir->full.wait(lock);
        }
        std::cout << "Producer item:" << item << ", producer pos:" << ir->producer_pos << std::endl;
        ir->items[ir->producer_pos++] = item;
    
        if(ir->producer_pos == K_ITEMS_SIZE) {
            ir->producer_pos = 0;
        }
        ir->not_empty.notify_all();
        lock.unlock();
    }
    
    void consume_items(ItemRepository* ir) {
        std::unique_lock<std::mutex> lock(ir->mutex);
        while (ir->producer_pos == ir->consumer_pos) {
            std::cout << "Producer is waiting for an empty slot...\n";
            ir->not_empty.wait(lock);
        }
        std::cout << "consumer item:" << ir->items[ir->consumer_pos] << ", consumer pos:" << ir->consumer_pos++ << std::endl;
        if(ir->consumer_pos == K_ITEMS_SIZE) {
            ir->consumer_pos = 0;
        }
        ir->full.notify_one();
        lock.unlock();
    }
    
    ItemRepository gItemRepository;
    void producer() {
        for (int i = 0; i < K_ITEMS_MAX; ++i) {
            produce_items(&gItemRepository, i);
        }
    }
    
    void consume() {
        int cnt = 0;
        while(1) {
            
            sleep(1);
            consume_items(&gItemRepository);
            if (cnt++ == K_ITEMS_MAX) {
                std::cout << "exit thread";
                break;
            }
        }
    }
    int main() {
        gItemRepository.consumer_pos = 0;
        gItemRepository.producer_pos = 0;
        std::thread consumer(consume);
        std::thread produce(producer);
    
        consumer.join();
        produce.join();
        return 0;
    }
    ```

###### 无锁队列

```c++
bool __sync_bool_compare_and_swap (type *ptr, type oldval type newval, ...)
type __sync_val_compare_and_swap (type *ptr, type oldval type newval, ...)
gcc支持的原子操作，c++1的atomic也支持一些原子操作
```

```c++
template <class T>
struct LockFreeListNode {
    T val;
	LockFreeListNode* tail;
    LockFreeListNode* head;
    LockFreeListNode* next;
};
```

入队

```c++
template <class T>
void push(LockFreeListNode* q, T data) {
    auto node = new LockFreeListNode(data);
    auto tail = q->tail;
    while (true) {
        tail = q->tail;
        auto next = tail->next;
        if (tail != q->tail) continue;
        if (next != nullptr) {
            //说明有其他线程已经加了尾结点，然后异常退出，需要其他线程的这个地方往后移动
            CAS(q->tail, tail, next);
        }
        if (CAS(tail, next, node) == true) break;
    }
    //这里可能异常退出，别的线程也能保证
    CAS(q->tail, tail, node);
}
```

出队

```
template <class T>
T pop(LockFreeListNode* q) {
    T value;
    while (true) {
        auto head = q->head;
        auto tail = q->tail;
        auto t_next = head->next;
        //已经被别的线程取走
        if ( head != q->head) continue;
        //整个队列为空
        if (head == tail && t_next == nullptr) {
            return EMPTY;
        }
        //tail是在head的后面,为啥
        if (head == tail && t_next == nullptr) {
            CAS(q->tail, tail, next);
        }
        if (CAS(q->head, head, t_next) == true) {
            value = t_next->val;
            break;         
        }
    }
    delete pre_head;
    return value;
}
```

所谓ABA（[见维基百科的ABA词条](https://en.wikipedia.org/wiki/ABA_problem)），问题基本是这个样子：

1. 进程P1在共享变量中读到值为A
2. P1被抢占了，进程P2执行
3. P2把共享变量里的值从A改成了B，再改回到A，此时被P1抢占。
4. P1回来看到共享变量里的值没有被改变，于是继续执行。

虽然P1以为变量值没有改变，继续执行了，但是这个会引发一些潜在的问题。**ABA问题最容易发生在lock free 的算法中的，CAS首当其冲，因为CAS判断的是指针的值。很明显，值是很容易又变成原样的。**

#### 解决ABA的问题

当然，我们这个队列的问题就是不想让那个内存重用，这样明确的业务问题比较好解决，这么一个方法——**使用结点内存引用计数refcnt**！
