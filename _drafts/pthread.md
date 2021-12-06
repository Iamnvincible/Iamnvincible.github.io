# Linux pthread

## 线程创建

使用 `pthread_create` 函数创建线程，并运行 `start_routine` 函数。

```c
#include <pthread.h>
int pthread_create (pthread_t *thread,
                    pthread_attr_t *attr,
                    void *(*start_routine)(void *),
                    void *arg)
```

- `pthread_t *thread`：线程标识符，一个 `pthread_t * ` 类型的变量，通常先定义一个 `pthread_t` 类型的变量，在函数调用时传入其指针。如果线程成功创建，该变量将会存放线程标识符，标识符为**无符号长整形**。如果线程创建失败，该变量值为**未定义**。
- `pthread_attr_t *attr`：线程属性。可以在线程创建时指定线程属性，指定为 `NULL` 时使用默认线程属性。
- ` void *(*start_routine)(void *)`：线程中运行的函数。一个函数指针，函数接受一个 `void *` 类型的参数，返回类型也是 `void *`。
- `void* arg`：线程运行函数的参数。
- 如果线程创建成功，线程返回 0；否则返回错误代码。

## 线程终止

线程在下面几种情况下会终止：

- 线程工作结束，正常返回。
- 线程调用了 `pthread_exit` 函数。
- 其他线程通过调用 `pthread_cancel` 终止当前线程。
- 进程调用了 `exec()` 或 `exit()`。
- 主函数没有调用 `pthread_exit` 而先结束。

```c
#include <pthread.h>
void pthread_exit(void *retval);
```

`pthread_exit` 函数终止调用线程，并将返回值放入 `retval`，其他线程可以通过 `retval` 获得线程返回状态。

如果在主函数（主线程）中调用该函数，那么主线程将会结束，但进程不会结束。当进程中的其他线程都结束时，进程才会结束。

```c
#include <pthread.h>
int pthread_cancel(pthread_t thread);
```

一个进程中的线程可以通过调用 `pthread_cancel`，来请求结束另一个线程。该函数参数为要结束的线程的标识符。如果结束成功返回 0，否则返回错误代码。

## 线程阻塞和分离

```c
#include <pthread.h>
int pthread_join(pthread_t thread, void **retval);
```

一个线程通过调用 `pthread_join` 函数等待 `thread` 线程结束，这个过程称为“阻塞（ join ）”。`thread` 线程运行结束后，继续当前线程。通过 `retval` 获得另一个线程的返回值。被阻塞的 `thread` 线程结束后，其所占用的资源将被释放。

- 一个线程仅能被一个线程阻塞。
- 线程不能阻塞自己。

```c
#include <pthread.h>
int pthread_detach(pthread_t thread);
```

调用 `pthread_join` 后，当前线程将等待被阻塞线程。使用 `pthread_detach` 函数可以使当前线程不等待目标线程而继续后续任务，并且在目标线程结束后，目标线程的资源将会自动释放。

- 线程可以自己分离自己。
- 线程被分离后，不能再被阻塞。

线程创建时，可以通过 `pthread_attr_t` 结构来设置线程属性，其中就有与阻塞和分离相关的属性。

```c
pthread_attr_t attr;
//初始化并设置 detached 属性
//可设置为 PTHREAD_CREATE_DETACHED 属性
//或 PTHREAD _CREATE_JOINABLE 属性
pthread_attr_init(&attr);
pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_JOINABLE);
pthread_create(thread, &attr, foo, (void *)arg);
//释放
pthread_attr_destroy(&attr);
```

>当线程需要阻塞时，考虑显示设置其可阻塞属性。并非所有情况下，线程都默认被设置为可阻塞。
>
>如果线程不会被用于阻塞，考虑在创建时指定其可分离属性，便于线程结束时自动释放资源。





## 参考链接

- [POSIX Threads Programming](https://hpc-tutorials.llnl.gov/posix/)
- [Pthreads 入门教程](https://hanbingyan.github.io/2016/03/07/pthread_on_linux/)
- [pthread_join和pthread_detach的用法](https://www.cnblogs.com/fnlingnzb-learner/p/6959285.html)
- [线程正常终止](https://www.cnblogs.com/zhangxuan/p/6430034.html)

