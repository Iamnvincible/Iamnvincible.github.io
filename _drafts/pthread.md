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

在主函数结束前调用 `pthread_exit` 会让主函数等待其他线程结束再终止进程。

## 参考链接

- [POSIX Threads Programming](https://hpc-tutorials.llnl.gov/posix/)
- [Pthreads 入门教程](https://hanbingyan.github.io/2016/03/07/pthread_on_linux/)

