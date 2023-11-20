```
#include <sys/types.h>
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <pthread.h>
#include <semaphore.h>

#define NUM 5

int queue[NUM];
sem_t psem, csem;

void producer(void* arg) {
    int pos = 0;
    int num, count = 0;
    for (int i = 0; i < 12; ++i) {
        num = rand() % 100;
        count += num;

        // 等待生产者信号量
        sem_wait(&psem);

        // 生产
        queue[pos] = num;

        // 发送消费者信号量
        sem_post(&csem);

        printf("Producer: %d\n", num);

        // 更新位置
        pos = (pos + 1) % NUM;

        // 随机休眠，模拟生产过程
        sleep(rand() % 2);
    }
    printf("Producer count = %d\n", count);
}

void consumer(void* arg) {
    int pos = 0;
    int num, count = 0;
    for (int i = 0; i < 12; ++i) {
        // 等待消费者信号量
        sem_wait(&csem);

        // 消费
        num = queue[pos];

        // 发送生产者信号量
        sem_post(&psem);

        printf("Consumer: %d\n", num);

        // 更新位置
        pos = (pos + 1) % NUM;

        // 随机休眠，模拟消费过程
        sleep(rand() % 3);

        count += num;
    }
    printf("Consumer count = %d\n", count);
}

int main() {
    // 初始化信号量
    sem_init(&psem, 0, NUM);  // 初始值为队列大小，表示可以生产的数量
    sem_init(&csem, 0, 0);    // 初始值为0，表示初始时没有可以消费的物品

    // 创建线程
    pthread_t tid[2];
    pthread_create(&tid[0], NULL, (void*)producer, NULL);
    pthread_create(&tid[1], NULL, (void*)consumer, NULL);

    // 等待线程结束
    pthread_join(tid[0], NULL);
    pthread_join(tid[1], NULL);

    // 销毁信号量
    sem_destroy(&psem);
    sem_destroy(&csem);

    return 0;
}

```
