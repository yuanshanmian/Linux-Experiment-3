```
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <errno.h>
#include <time.h>
#include <pthread.h>

// 通过 condition_t 结构体封装了互斥锁 pthread_mutex_t 和条件变量 pthread_cond_t。这种结构体的组合常见于多线程编程中，用于实现复杂的同步机制，确保多个线程能够正确地协同工作。
typedef struct condition
{
    pthread_mutex_t pmutex;
    pthread_cond_t pcond;
} condition_t;

typedef struct task
{
    void *(*run)(void *arg);
    void *arg;
    struct task *next;
} task_t;

typedef struct threadpool
{
    condition_t ready;
    task_t *first;
    task_t *last;
    int counter;
    int idle;
    int max_threads;
    int quit;
} threadpool_t;

// 线程同步相关操作
int condition_init(condition_t *cond)
{
    int status;
    if ((status = pthread_mutex_init(&cond->pmutex, NULL)))
        return status;
    if ((status = pthread_cond_init(&cond->pcond, NULL)))
        return status;
    return 0;
}

int condition_lock(condition_t *cond)
{
    return pthread_mutex_lock(&cond->pmutex);
}

int condition_unlock(condition_t *cond)
{
    return pthread_mutex_unlock(&cond->pmutex);
}

int condition_wait(condition_t *cond)
{
    return pthread_cond_wait(&cond->pcond, &cond->pmutex);
}

int condition_timewait(condition_t *cond, const struct timespec *abstime)
{
    return pthread_cond_timedwait(&cond->pcond, &cond->pmutex, abstime);
}

int condition_signal(condition_t *cond)
{
    return pthread_cond_signal(&cond->pcond);
}

int condition_broadcast(condition_t *cond)
{
    return pthread_cond_broadcast(&cond->pcond);
}

int condition_destroy(condition_t *cond)
{
    int status;
    if ((status = pthread_mutex_destroy(&cond->pmutex)))
        return status;
    if ((status = pthread_cond_destroy(&cond->pcond)))
        return status;
    return 0;
}

// 线程执行的任务函数
void *thread_routine(void *arg)
{
    struct timespec abstime;
    int timeout;
    printf("Thread 0x%0x is starting\n", (int)pthread_self());
    threadpool_t *pool = (threadpool_t *)arg;
    while (1)
    {
        timeout = 0;
        condition_lock(&pool->ready);
        pool->idle++;

        // 等待队列有任务到来或者线程池销毁的通知
        while (pool->first == NULL && !pool->quit)
        {
            printf("Thread 0x%0x is waiting\n", (int)pthread_self());
            clock_gettime(CLOCK_REALTIME, &abstime);
            abstime.tv_sec += 2;
            int status = condition_timewait(&pool->ready, &abstime);
            if (status == ETIMEDOUT)
            {
                printf("Thread 0x%0x is wait timed out\n", (int)pthread_self());
                timeout = 1;
                break;
            }
        }

        // 等到到条件，处于工作状态
        pool->idle--;
        if (pool->first != NULL)
        {
            task_t *t = pool->first;
            pool->first = t->next;

            // 需要先解锁，以便添加新任务。其他消费者线程能够进入等待任务。
            condition_unlock(&pool->ready);

            t->run(t->arg);
            free(t);

            condition_lock(&pool->ready);
        }

        // 等待线程池销毁的通知
        if (pool->quit && pool->first == NULL)
        {
            pool->counter--;
            if (pool->counter == 0)
            {
                condition_signal(&pool->ready);
            }
            condition_unlock(&pool->ready);
            // 跳出循环之前要记得解锁
            break;
        }

        if (timeout && pool->first == NULL)
        {
            pool->counter--;
            condition_unlock(&pool->ready);
            // 跳出循环之前要记得解锁
            break;
        }
        condition_unlock(&pool->ready);
    }
    printf("Thread 0x%0x is exiting\n", (int)pthread_self());
    return NULL;
}

// 初始化线程池
void threadpool_init(threadpool_t *pool, int threads)
{
    condition_init(&pool->ready);
    pool->first = NULL;
    pool->last = NULL;
    pool->counter = 0;
    pool->idle = 0;
    pool->max_threads = threads;
    pool->quit = 0;
}

// 向线程池中添加任务
void threadpool_add_task(threadpool_t *pool, void *(*run)(void *arg), void *arg)
{
    task_t *new_task = (task_t *)malloc(sizeof(task_t));
    new_task->run = run;
    new_task->arg = arg;
    new_task->next = NULL;

    condition_lock(&pool->ready);

    // 将任务添加到队列中
    if (pool->first == NULL)
    {
        pool->first = new_task;
    }
    else
        pool->last->next = new_task;
    pool->last = new_task;

    // 如果有等待线程，则唤醒其中一个
    if (pool->idle > 0)
    {
        condition_signal(&pool->ready);
    }
    else if (pool->counter < pool->max_threads)
    {
        pthread_t tid;
        pthread_create(&tid, NULL, thread_routine, pool);
        pool->counter++;
    }

    condition_unlock(&pool->ready);
}

// 销毁线程池
void threadpool_destroy(threadpool_t *pool)
{
    if (pool->quit)
    {
        return;
    }
    condition_lock(&pool->ready);
    pool->quit = 1;

    if (pool->counter > 0)
    {
        if (pool->idle > 0)
            condition_broadcast(&pool->ready);

        while (pool->counter > 0)
        {
            condition_wait(&pool->ready);
        }
    }

    condition_unlock(&pool->ready);
    condition_destroy(&pool->ready);
}

// 线程池中任务的示例函数
void *my_task(void *arg)
{
    printf("Thread 0x%0x is working on task %d\n", (int)pthread_self(), *(int *)arg);
    sleep(1);
    free(arg);
    return NULL;
}

int main()
{
    threadpool_t pool;
    threadpool_init(&pool, 3);

    int i;
    for (i = 0; i < 10; i++)
    {
        int *arg = (int *)malloc(sizeof(int));
        *arg = i;
        threadpool_add_task(&pool, my_task, arg);
    }

    sleep(15);
    threadpool_destroy(&pool);

    return 0;
}

```
