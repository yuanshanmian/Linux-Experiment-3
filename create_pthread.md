```
#include <stdio.h>
#include <pthread.h>

// 线程1的执行函数
void *mythread1(void)
{
    int i;
    for (i = 0; i < 10; i++)
    {
        printf("This is pthread1, i=%d\n", i);
        sleep(1); // 休眠1秒，模拟耗时操作
    }
}

// 线程2的执行函数
void *mythread2(void)
{
    int i;
    for (i = 0; i < 10; i++)
    {
        printf("This is pthread2, i=%d\n", i);
        sleep(1); // 休眠1秒，模拟耗时操作
    }
}

int main(int argc, const char *argv[])
{
    int i = 0;
    int ret = 0;
    pthread_t id1, id2;  // pthread_t一般是一个足够大的整数类型

    // 创建线程1
    ret = pthread_create(&id1, NULL, (void *)mythread1, NULL);
    if (ret)
    {
        printf("Create pthread1 error!\n");
        return 1;
    }

    // 创建线程2
    ret = pthread_create(&id2, NULL, (void *)mythread2, NULL);
    if (ret)
    {
        printf("Create pthread2 error!\n");
        return 1;
    }

    // 等待线程1和线程2结束
    pthread_join(id1, NULL);
    pthread_join(id2, NULL);
```

    return 0;
}
