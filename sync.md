```
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <unistd.h>
#include <pthread.h>

#define LEN 100000

int num = 0;

// 线程执行的函数
void* thread_func(void* arg) {
    // 传递进来的参数是互斥锁
    pthread_mutex_t* p_mutex = (pthread_mutex_t*)arg;

    // 执行一定次数的加法操作
    for (int i = 0; i < LEN; ++i) {
        // 加锁，保证在修改共享资源时的互斥性
        pthread_mutex_lock(p_mutex);
        num += 1;
        // 解锁，释放锁
        pthread_mutex_unlock(p_mutex);
    }

    return NULL;
}

int main() {
    pthread_mutex_t m_mutex;

    // 初始化互斥锁
    pthread_mutex_init(&m_mutex, NULL);

    pthread_t tid1, tid2;

    // 创建两个线程，传入互斥锁作为参数
    pthread_create(&tid1, NULL, (void*)thread_func, (void*)&m_mutex);
    pthread_create(&tid2, NULL, (void*)thread_func, (void*)&m_mutex);

    // 等待两个线程执行完毕
    pthread_join(tid1, NULL);
    pthread_join(tid2, NULL);

    // 销毁互斥锁
    pthread_mutex_destroy(&m_mutex);

    // 输出结果，期望结果是 2 * LEN
    printf("correct result=%d, result=%d.\n", 2 * LEN, num);

    return 0;
}

```
运行结果：correct result=200000, result=200000.
