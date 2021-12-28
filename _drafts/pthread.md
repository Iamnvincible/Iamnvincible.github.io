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

一个线程通过调用 `pthread_join` 函数等待 `thread` 线程结束，这个过程称为阻塞（ join ）。`thread` 线程运行结束后，继续当前线程。通过 `retval` 获得另一个线程的返回值。被阻塞的 `thread` 线程结束后，其所占用的资源将被释放。

- 一个线程仅能被一个线程阻塞。
- 线程不能阻塞自己。

```c
#include <pthread.h>
int pthread_detach(pthread_t thread);
```

调用 `pthread_join` 后，当前线程将等待被阻塞线程。

使用 `pthread_detach` 函数可以使当前线程不等待目标线程而继续后续任务，并且在目标线程结束后，目标线程的资源将会自动释放。

- 线程可以自己分离自己。
- 线程被分离后，不能再被阻塞。
- 线程设置了阻塞后，可以将其分离，但被分离后的线程不能被设置为阻塞。

线程创建时，可以通过 `pthread_attr_t` 结构来设置线程属性，其中就有与阻塞和分离相关的属性。

```c
pthread_attr_t attr;
//初始化并设置 detached 属性
//可设置为 PTHREAD_CREATE_DETACHED 属性
//或 PTHREAD_CREATE_JOINABLE 属性
pthread_attr_init(&attr);
pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_JOINABLE);
pthread_create(thread, &attr, foo, (void *)arg);
//释放
pthread_attr_destroy(&attr);
```

>当线程需要阻塞时，考虑显式设置其可阻塞属性。因为并非所有情况下，线程都默认被设置为可阻塞。
>

### 线程资源释放
线程运行完毕后，就需要考虑资源释放。使用 `pthread_create` 创建线程后，线程会立即开始运行。一般地，线程与线程之间不会通信，线程运行结束后不会将资源释放，而是仍由其所在的进程持有。如果不使用 `pthread_join` 或 `pthread_detach` 设置线程，那么线程在结束后不会会释放资源。

- 如果线程不会被用于阻塞，考虑在创建时为线程指定 `PTHREAD_CREATE_DETACHED` 属性，便于线程结束时自动释放资源。
- 用于阻塞的线程一定要在创建后使用 `pthread_join` 函数将其阻塞。

## 互斥锁
多线程编程时，多个线程常常需要访问同一个数据源，为确保访问的数据有效，必须使用锁。互斥锁（ Mutex ）可以用来保护被多个线程访问的资源，防止多个线程同时更新一个数据时出现同步错误。

使用互斥锁的一般步骤是：
1. 创建互斥锁。
2. 多个线程尝试锁定互斥锁。
3. 成功锁定互斥锁的线程成为互斥锁的拥有者，并在这之后执行一些代码。
4. 互斥锁拥有者解锁。
5. 其他线程尝试锁定互斥锁，并重复 3-5 步。
6. 不再需要互斥锁时释放互斥锁资源。

初始化互斥锁有两种方式：
1. 使用常量 `PTHREAD_MUTEX_INITIALIZER` 来初始化互斥锁变量。
   - 例如 `pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;`。
   - 但定义和初始化需要写在一起，不能分开。
2. 使用函数 `pthread_mutex_init(mutex,attr)` 初始化互斥锁变量，这种方式可以设置互斥锁属性。
   - 互斥锁属性初始化函数 `pthread_mutexattr_init (attr)`。
   - 互斥锁属性释放函数 `pthread_mutexattr_destroy (attr)`。

互斥锁初始化后，默认为解锁状态。当不再需要互斥锁时，使用 `pthread_mutex_destroy (pthread_mutex_t *mutex)` 释放互斥锁资源。

线程可以使用三种方式尝试锁定互斥锁：
1. `pthread_mutex_lock(mutex)`。当互斥锁已经被其他线程锁定时，该函数会阻塞，直到互斥锁被解锁。
2. `pthread_mutex_trylock(mutex)`。当互斥锁已经被其他线程锁定时，该函数会立即返回一个表示“忙碌 busy”的错误码。

线程执行完需要的代码后，需要使用 `pthread_mutex_unlock (mutex)` 函数解锁。

互斥锁的使用没有什么特别的地方，实践中更像是线程间的“君子协定”。程序员需要保证所有访问同一数据源的线程都使用了互斥锁机制，否则可能产生非预期结果。

## 条件变量

互斥锁通过控制线程访问数据的方式来实现同步，而条件变量能够在特定条件下进行数据同步。在使用互斥锁时，没有获得锁的线程会持续查询锁的状态，这种方式显然会浪费 CPU 资源。
通过使用条件变量，一个线程可以在数据达到某个条件时通知另一个线程。

> 条件变量需要与互斥锁同时使用，使用条件变量的同时也需要初始化互斥锁。

初始化条件变量有两种方式：
1. 使用常量初始化条件变量。
   例如： `pthread_cond_t myconvar = PTHREAD_COND_INITIALIZER;`。
2. 使用函数 `pthread_cond_init(cond,attr)` 初始化条件变量，这种方式可以设置条件变量属性。
   - 条件变量属性初始化函数 `pthread_condattr_init (attr)`。
   - 条件变量属性释放函数 `pthread_condattr_destroy (attr)`。

释放条件变量函数：
- `pthread_cond_destroy (condition)`。

使用函数 `pthread_cond_wait (condition,mutex)` 来等待条件变量成立的通知。这个函数需要在已经获得互斥锁的前提下使用，所以在使用这个函数前必须有一个 `pthread_mutex_lock(mutex)` 存在。

条件变量使用的一般步骤：
1. 线程 t 通过 `pthread_mutex_lock(mutex)` 获得互斥锁。
2. 线程 t 调用 `pthread_cond_wait(condition,mutex)` 等待条件成立。
3. 条件成立通知尚未到来时，`pthread_cond_wait(condition,mutex)` 函数将会阻塞，同时自动解锁互斥锁。
4. 其他线程获得互斥锁。
5. 其他线程通过 `pthread_cond_signal(condition)` 函数发送条件成立通知，随后使用 `pthread_mutex_unlock(mutex)` 函数解锁互斥锁。
6. 线程 t 收到条件变量通知被唤醒，同时自动锁定互斥锁。
   1. 如果此时互斥锁不可用？
7. 线程 t 执行后续代码，完成后使用 `pthread_mutex_unlock(mutex)` 函数解锁互斥锁。

> 在使用条件变量时，线程 t 往往需要和其他线程访问同一数据。但在开始时该数据尚未达到线程 t 的执行条件，在判断这个执行条件时，推荐使用 `while` 循环而非 `if` 以避免下面的问题。
> - 当有多个线程等待条件变量时，多个等待线程会先后获得互斥锁，一些线程可能会修改共同访问的数据使得条件判断条件不成立，所以在获得通知后要用 `while` 再进行一次判断。
> - 线程接收条件通知时可能会出现错误，使用 `while` 能再次尝试。
> - ...

如果需要通知多个处于等待条件状态的线程，可以使用 `pthread_cond_broadcast(condition)`。







## 参考链接

- [POSIX Threads Programming](https://hpc-tutorials.llnl.gov/posix/)
- [Pthreads 入门教程](https://hanbingyan.github.io/2016/03/07/pthread_on_linux/)
- [pthread_join和pthread_detach的用法](https://www.cnblogs.com/fnlingnzb-learner/p/6959285.html)
- [线程正常终止](https://www.cnblogs.com/zhangxuan/p/6430034.html)
- [Using Condition Variables](https://docs.oracle.com/cd/E19455-01/806-5257/6je9h032r/index.html)

