```
#include <pthread.h>
#include <stdio.h>
#include <sys/time.h>
#include <string.h>
#include <unistd.h> // 添加头文件以使用 sleep 函数

#define MAX 20

pthread_t thread[2];  // 两个线程
pthread_mutex_t mut;
int number = 0;
int i;

void *thread1()
{
    printf("thread1: This is thread1\n");
    for (i = 0; i < MAX; i++)
    {
        printf("thread1: number = %d\n", number);
        pthread_mutex_lock(&mut);
        number++;
        pthread_mutex_unlock(&mut);
        sleep(2); // 模拟线程执行时间
    }
    printf("thread1: Is the main function waiting for me to complete the task?\n");
    pthread_exit(NULL);
}

void *thread2()
{
    printf("thread2: This is thread2\n");
    for (i = 0; i < MAX; i++)
    {
        printf("thread2: number = %d\n", number);
        pthread_mutex_lock(&mut);
        number++;
        pthread_mutex_unlock(&mut);
        sleep(3); // 模拟线程执行时间
    }
    printf("thread2: Is the main function waiting for me to complete the task?\n");
    pthread_exit(NULL);
}

void thread_create(void)
{
    int temp;
    memset(&thread, 0, sizeof(thread)); // 清空线程数组
    /* 创建线程 */
    if ((temp = pthread_create(&thread[0], NULL, thread1, NULL)) != 0)
        printf("Thread 1 creation failed\n");
    else
        printf("Thread 1 created\n");

    if ((temp = pthread_create(&thread[1], NULL, thread2, NULL)) != 0)
        printf("Thread 2 creation failed\n");
    else
        printf("Thread 2 created\n");
}

void thread_wait(void)
{
    /* 等待线程结束 */
    if (thread[0] != 0)
    {
        pthread_join(thread[0], NULL);
        printf("Thread 1 has ended\n");
    }
    if (thread[1] != 0)
    {
        pthread_join(thread[1], NULL);
        printf("Thread 2 has ended\n");
    }
}

int main()
{
    /* 用默认属性初始化互斥锁 */
    pthread_mutex_init(&mut, NULL);

    printf("Create threads\n");
    thread_create();
    printf("Waiting for threads to complete tasks\n");
    thread_wait();

    return 0;
}

```
