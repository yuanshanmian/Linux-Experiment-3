### pthread_join()：
```
int pthread_join(pthread_t thread, void **retval);
```
thread: 要等待的线程的标识符，即由 pthread_create 返回的 pthread_t 类型的标识符。

retval: 用于存储被等待线程的返回值的指针，如果不关心返回值，可以传递 NULL。

pthread_join 函数会将调用线程（通常是主线程）阻塞，直到指定的线程（通过 thread 参数指定）终止。一旦被等待的线程终止，pthread_join 将返回，并且如果提供了 retval 参数，将把被等待线程的返回值存储在 retval 指向的地址中。

## pthread_create()：
```
int pthread_create(pthread_t *thread, const pthread_attr_t *attr,
                   void *(*start_routine)(void *), void *arg);
```
thread: 用于存储新创建线程的标识符的指针，即 pthread_t 类型的变量的地址。
attr: 用于设置线程属性的参数，通常为 NULL，表示使用默认属性。
start_routine: 是一个指向线程主函数的指针，该函数的参数和返回值都是 void* 类型。这个主函数是新线程启动时将要执行的函数。
arg: 是传递给 start_routine 函数的参数。

# pthread_mutex_t 结构体：
```
typedef struct {
    // 互斥锁的属性，通常为默认值 NULL，表示使用默认属性
    pthread_mutexattr_t *__m_mutexattr;
    
    // 互斥锁的数据结构，具体实现由系统决定
    // 一般包含一个锁状态和等待队列等信息
    void *__m_reserved[2];
} pthread_mutex_t;
```
互斥锁属性 (__m_mutexattr)： 互斥锁可以有不同的属性，例如递归锁、错误检查锁等。在上述代码中，使用默认属性，通常传递 NULL 表示使用默认值。

互斥锁的数据结构： 该结构体包含了用于实现互斥锁的具体数据。这些数据由系统决定，包括锁状态、等待队列等。

在上述代码中，pthread_mutex_t 结构体被用于实现条件变量，结合了条件变量 pthread_cond_t 来实现线程的同步和互斥。通过 pthread_mutex_lock 和 pthread_mutex_unlock 操作，可以在关键代码段（临界区）加锁，确保同一时刻只有一个线程可以执行该段代码，从而保护共享资源的完整性。

### pthread_mutex_lock()：
是 POSIX 线程库中用于锁定互斥量（Mutex）的函数。互斥量是一种用于控制多个线程对共享资源访问的同步机制。互斥量的目的是防止多个线程同时访问共享资源，从而避免数据竞争和不确定的行为。
```
#include <pthread.h>
int pthread_mutex_lock(pthread_mutex_t *mutex);
```
mutex：指向互斥量对象的指针。

pthread_mutex_lock() 函数的作用是锁定互斥量，如果互斥量已经被其他线程锁定，调用线程将被阻塞，直到互斥量可用。如果互斥量当前没有被锁定，调用线程将成功锁定互斥量，并可以继续执行临界区代码。

使用互斥量的目的是确保在任意时刻只有一个线程可以访问共享资源，以避免并发访问导致的问题。

### pthread_mutex_unlock()： 
是 POSIX 线程库中用于释放互斥锁（Mutex）的函数。在多线程编程中，互斥锁用于保护共享资源，确保在任何时刻只有一个线程能够访问这些资源，从而防止数据竞争和一致性问题。
```
int pthread_mutex_unlock(pthread_mutex_t *mutex);

```
mutex: 一个指向互斥锁对象的指针。

pthread_mutex_unlock() 函数用于解锁已经被线程锁定的互斥锁。如果当前线程拥有这个互斥锁，那么该锁就会被释放。如果当前线程没有锁定这个互斥锁，pthread_mutex_unlock() 的行为是未定义的。


# pthread_cond_t 结构体：
```
typedef struct {
    // 条件变量的属性，通常为默认值 NULL，表示使用默认属性
    pthread_condattr_t *__c_condattr;
    
    // 条件变量的数据结构，具体实现由系统决定
    // 一般包含一个等待队列等信息
    void *__c_reserved[2];
} pthread_cond_t;
```
条件变量属性 (__c_condattr)： 条件变量可以有不同的属性，例如时钟类型、进程共享等。在上述代码中，使用默认属性，通常传递 NULL 表示使用默认值。

条件变量的数据结构： 该结构体包含了用于实现条件变量的具体数据。这些数据由系统决定，通常包含一个等待队列等信息。

在上述线程池程序中，pthread_cond_t 结构体用于实现条件变量，通过 condition_t 结构体封装了互斥锁 pthread_mutex_t 和条件变量 pthread_cond_t。这种结构体的组合常见于多线程编程中，用于实现复杂的同步机制，确保多个线程能够正确地协同工作。

关于条件变量的常见操作有：
```
pthread_cond_init：初始化条件变量。
pthread_cond_wait：等待条件变量满足。
pthread_cond_timedwait：带有超时的等待条件变量满足。
pthread_cond_signal：唤醒等待条件变量的一个线程。
pthread_cond_broadcast：唤醒等待条件变量的所有线程。
pthread_cond_destroy：销毁条件变量。
```




