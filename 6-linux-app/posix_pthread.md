
线程thread在linux中是经常会被程序员提高的一个东西，经常说多线程，现场创建的概念，pthread也是程序员经常用到的东西, pthread是什么？

# pthread相关介绍
提到pthread, 了解下posix接口和官方对pthread的介绍

## posix接口
百度百科 https://baike.baidu.com/item/POSIX/3792413?fr=aladdin
POSIX表示可移植操作系统接口（Portable Operating System Interface of UNIX，缩写为 POSIX ），POSIX标准定义了操作系统应该为应用程序提供的接口标准，是IEEE为要在各种UNIX操作系统上运行的软件而定义的一系列API标准的总称，其正式称呼为IEEE 1003，而国际标准名称为ISO/IEC 9945。

## pthread介绍
官网  http://www.yolinux.com/TUTORIALS/LinuxTutorialPosixThreads.html

POSIX thread (pthread) libraries
The POSIX thread libraries are a standards based thread API for C/C++.

# 学习线程的创建
```javascript
int pthread_create(pthread_t *thread, const pthread_attr_t *attr,
        void *(*start_routine) (void *), void *arg);
```
      
```javascript
void* pthread_entry(void *arg)
{   
  for (int i = 0; i < 5; i++) {
    printf("run count:%d\r\n", i);
  }
}
void main()
{
   //线程变量
  pthread_t thread1;
  //通过pthread_create进行创建，
  pthread_create(&thread1, NULL, &pthread_entry, NULL );
}
```

# pthread同步机制(join)
pthread_join是阻塞等待某个线程退出，如果线程不退出，一直阻塞。

1. API介绍
```javascript
  #include <pthread.h>
  int pthread_join(pthread_t thread, void **retval);
  ```
  
2. 代码例子
```javascript
void* pthread_join_entry(void *arg)
{   
  for (int i = 0; i < 5; i++) {
    printf("run count:%d\r\n", i);
  }
}

int pthread_join_test(void)
{
  pthread_t pthread_join_test;

  pthread_create(&pthread_join_test, NULL, &pthread_join_entry, NULL );
        //等待线程退出
  printf("pthread_join_test wait exit \r\n");
  pthread_join(pthread_join_test,NULL);
        //线程退出才会继续执行
  printf("pthread_join_test exit \r\n");

  return 0;
}

```

# pthread互斥锁使用

1. 互斥锁静态申请
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
2. 互斥锁的使用
pthread_mutex_lock(&mutex );
pthread_mutex_unlock(&mutex );

```javascript

pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
int count = 0;
//消费者
void* consume( void *arg)
{
  while(count < 10)
  {
    pthread_mutex_lock(&mutex );
    printf("************************consumed%d\n",count );
    sleep(2);
    count++;
    printf("************************consumed%d\n",count);
    pthread_mutex_unlock(&mutex );
    sleep(1);
  }
}
//生产者
void* produce( void * arg )
{
  while(1 )
  {
    pthread_mutex_lock(&mutex );
    printf("Produced %d\n", count );
    pthread_mutex_unlock(&mutex );
    sleep(1);
  }

}

void pthread_mutex_test(void)
{  
  pthread_t thread1,thread2;

  pthread_create(&thread1, NULL, &produce, NULL );
  pthread_create(&thread2, NULL, &consume, NULL );

  pthread_join(thread1,NULL);
  pthread_join(thread2,NULL);

}

```



