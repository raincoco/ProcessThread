# 进程与线程（Process And Thread）

## 1、进程在内存中分3部分：代码段、堆栈段、数据段。

## 2、getpid()函数
函数功能：获取本程序运行时的进程编号PID。<br>
函数声明：
```c++
#include<sys/types.h>  
#include<unistd.h>

pid_t getpid(void); 
```
返回值：进程编号PID。<br>

## 3、fork()函数
函数功能：创建了一个新的进程（子进程）。<br>
函数声明：
```c++
pid_t fork();
```
返回值：在父进程中返回进程编号，在子进程中返回0。<br>

if和else可同时执行。<br>
子进程与父进程使用相同的代码段，子进程拷贝了父进程的堆栈段和数据段。<br>
子进程复制了父进程的一切数据，但是两者独立运行，相互没有影响。<br>

## 4、僵尸进程
1）一个子进程调用return函数结束时并没有被真正销毁，而是留下一个僵尸进程。僵尸进程是子进程比父进程先结束，而父进程又没有回收子进程资源，释放子进程所占用的资源。<br>
2）如果父进程先退出，子进程会被系统接管，子进程退出后系统会收回其所占用的相关资源，不会成为僵尸进程。<br>
3）僵尸进程标志：<defunct><br>
4）僵尸进程危害：在僵尸进程消失前会继续占有系统资源。<br>
5）并发程序解决僵尸进程问题：使用signal忽略子进程信号。<br>

## 5、后台运行
1）程序名后加上&可以将程序放入后台。<br>
2）采用fork，主程序执行fork生成一个子进程，然后父进程退出，子进程由系统托管继续运行。<br>
3）中止后台程序，使用kill、killall杀死程序。<br>

## 6、信号signal
函数功能：signal函数库可以设置信号的处理方式。<br>
函数声明：
```c++
#include<signal.h>

sighandler_t signal(int signum, sighandler_t handler);
```
参数:<br>
>signum参数：信号编号；<br>
>handler参数：信号的处理方式（忽略、默认操作、自定义操作）。可以自己定义处理方法。<br>

## 7、发送信号——kill
函数功能：Linux提供kill命令向程序发送信号。<br>
函数声明：
```c++
int kill(pid_t, int signal);
```
>参数pid：<br>
>>1）pid>0 将信号传给进程识别码为pid的进程；<br>
>>2）pid=0 将信号传给和目前程序相同进程组的所有进程，常用于父进程给子进程发送信号。<br>
>>3）pid=-1 将信号广播传给系统内所有的进程，例如系统关机时通知所有登录窗口。<br>

>参数signal：准备发送的信号代码，0则没有任何信号送出，但是系统会执行错误检查，通常利用signal为0来检验某个进程是否仍在执行。关于signal=0，可以作为监控使用。<br>

返回值：成功，返回0；失败，返回-1，且error设置为一下值：<br>
>EINVAL：指定信号参数无效（signal不合法）；<br>
>EPERM：权限不够无法传送信号给指定程序。<br>
>ESRCH：参数pid所指定的进程或进程主不存在。<br>

常用信号表：<br>


## 8、进程通讯
进程通讯方式：<br>
1）无名管道（pipe）和有名管道（named pipe）<br>
>无名管道：父子进程之间进行通讯；<br>
>有名管道：都可以通讯。<br>

2）信号（signal）<br>
3）消息队列（message）：消息队列是消息的连接表，进程可以向队列中添加信息，其他的进程则可以读走队列中的消息。<br>
4）共享内存：多个进程访问同一块内存。<br>
5）信号灯（semaphore）：信号量，主要作为进程之间对共享资源的加锁手段。<br>
6）套接字。<br>

## 9、共享内存
### 1）共享内存操作函数头文件：
```c++
#include <sys/types.h> 
#include <sys/ipc.h> 
#include <sys/shm.h> 
```
### 2）shmget函数：
函数功能：创建或者获取一块共享内存，返回获得共享内存区域的ID，如果不存在指定的共享区域就创建相应的区域。<br>
函数声明：
```c++
int shmget(key_t key, int size, int flag); 
```
参数:<br>
>key：共享内存的标识符，一个整数（用十六进制方便看），是共享内存在系统中的编号。如果是父子关系的进程间通信的话，这个标识符用   >IPC_PRIVATE来代替。 如果两个进程没有任何关系，所以就用ftok()算出来一个标识符（或者自己定义一个）使用了。<br>
>size：指定共享内存大小，字节为单位。 <br>
>flag：内存的模式(mode)以及权限标识。  <br>
>>模式可取如下值：<br>
>>IPC_CREAT：新建（如果已创建则返回目前共享内存的id） 。 <br>
>>IPC_EXCL：与 IPC_CREAT结合使用，如果已创建则返回错误。<br>
>>将“模式” 和“权限标识”进行或运算，做为第三个参数。如：IPC_CREAT | IPC_EXCL | 0640 。<br>

返回值：成功时返回共享内存的ID，失败时返回-1。<br>

创建共享内存时，shmflg参数至少需要 IPC_CREAT | 权限标识，如果只有IPC_CREAT 则申请的地址都是k=0xffffffff，不能使用；<br>

### 3）shmat函数：
函数功能：用来允许本进程访问一块共享内存的函数。<br>
>第一次创建共享内存时，它不能任何进程访问，要想启用对该共享内存的访问，必须将其连接到一个进程的地址空间中。shmat函数就是用来完成此工作的。<br>

函数声明：
```c++
void* shmat(int shmid,  const void *addr, int flag);
```
参数：<br>
>shmid：是由shmget函数返回的共享内存标识。<br>
>shmaddr：共享内存连接到进程中的起始地址，通常为NULL，由系统选择共享内存地址。<br>                                                 
>shmflag：本进程对该内存的操作模式，可以由两个取值：<br>
>>SHM_RND和SHM_RDONLY。<br>
>>SHM_RDONLY是只读模式。<br>
>需要注意的是，共享内存的读写权限由它的属主、它的访问权限和当前进程的属主共同决定。如果当shmflg & SM_RDONLY为true时，即使该共享内存的访问权限允许写操作，它也不能被写入。该参数通常会被设为0。<br>

返回值：成功时，返回共享内存的起始地址；失败，返回-1。<br>

### 4）shmdt函数：（shmat反操作）
函数功能：对共享内存进行控制。将共享内存从当前进程中分离。<br>
函数声明：
```c++
int shmdt(char *shmaddr);
```
参数：<br>
>shmaddr：shmat函数返回的地址。<br>

返回值：成功，放回0；失败，返回-1。<br>

### 5）shmctl函数
函数功能：删除共享内存<br>
函数声明：
```c++
int shmctl(int shm_id, int command, struct shmid_ds *buf);
```
参数；<br>
>shm_id：shmget函数返回的共享内存标识符。<br>
>command：控制命令：<br>
>>IPC_STAT：得到共享内存的状态：把shmid_ds结构中的数据设置为共享内存的当前关联值。<br>
>>IPC_SET：改变共享内存的状态：把共享内存的当前关联值设置为shmid_ds结构中给出的值。<br>
>>IPC_RMID：删除共享内存段。<br>
>shmid_ds结构至少包含以下成员：<br>

```c++
struct shmid_ds {
	uid_t shm_perm.uid;
	uid_t shm_perm.gid;
	uid_t shm_perm.mode;
}
```
>buf：一个结构体指针。IPC_STAT的时候，取得的状态放在这个结构体中。平时填0；<br>

返回值：成功，放回0；失败，返回-1。<br>

### 6）查看共享内存命令：ipcs
删除共享内存命令：ipcrm

## 10、线程
同一进程中可以运行多个线程。运行于一个进程中的多线程彼此间使用相同的地址空间，共享全局数据，启动一个线程所消耗的资源比启动一个进程所消耗的资源要少。
### 1）创建线程
函数声明：
```c++
#include<pthread.h>
	int   pthread_create(
	pthread_t* thread,
	pthread_attr_t * attr, 
	void * (*start_routine)(void *),
	void * arg
	);
```
>thread：指向线程标识符的地址。pthread_t：是一个无符号长整数。<br>
>attr：设置线程属性，一般为空（详细属性需要查询）。<br>
>start_routine：线程运行函数的地址。<br>
>arg：线程运行函数的参数。新建的线程从start_routine函数开始运行，想要向start_routine传递多个函数，可以将多个参数放入一个结构体，将结构体的地址作为arg参数传入。（单个参数传入最好强制转换参数传值，不要传递地址）<br>
>pthread_t是个整数。<br>

返回值：成功，返回0；失败，返回错误号。<br>

在编译时要加上-l pthread参数调用静态链接库。pthread不是linux的系统默认库。<br>
### 2）线程的中止
被主进程或者其他线程中止被杀。<br>
线程的start_routine调用pthread_exit结束：<br>
```c++
void pthread exit(void *retval);
```
>retval填空。<br>
>
a）在线程中用exit退出的不仅是线程，还有主进程。<br>
b）线程不存在僵尸的说法。<br>
c）Linux没有真正意义上的多线程，是由进程来模拟的，属于用户级线程。所以在Linux系统下，进程与线程在性能和资源消耗方面没有本质的差别。<br>
d）Linux中进程与线程最大的区别就是线程可以共享全局数据，进程不能共享全局数据。<br>
关于进程与线程之间类传递的成员变量：


## 11、线程同步
互斥锁：
同一时刻只允许一个线程占有公共资源。
函数头文件：
```c++
#include<pthread.h>
```
##3 1）初始化锁
函数声明：
```c++
pthread_mutex_init(pthread_mutex_t *restrict mutex, 
                   const pthread_mutexattr_t *restrict attr);
```   
mutex指定锁的属性，指定为NULL则使用缺省属性。<br>
>PTHREAD_MUTEX_TIMED_NP：这是缺省值，也就是普通锁。当一个线程加锁以后，其余请求锁的线程将形成一个等待队列，并在解锁后按优先级获得锁。这种锁策略保证了资源分配的公平性。	<br>

>PTHREAD_MUTEX_RECURSIVE_NP：嵌套锁，允许同一个线程对同一个锁成功获得多次，并通过多次unlock解锁。如果是不同线程请求，则在加锁线程解锁时重新竞争。<br>
>>解析：嵌套锁也叫递归锁，在同一个线程中，由于业务问题，需要递归一块上锁代码，可以使用这个属性。	<br>

>PTHREAD_MUTEX_ERRORCHECK_NP：检错锁，如果同一个线程请求同一个锁，则返回EDEADLK，否则与PTHREAD_MUTEX_TIMED_NP类型动作相同。这样就保证当不允许多次加锁时不会出现最简单情况下的死锁。	<br>

>PTHREAD_MUTEX_ADAPTIVE_NP：适应锁，动作最简单的锁类型，仅等待解锁后重新竞争。<br>

### 2）阻塞加锁
函数功能当请求资源的锁被占据的时候，挂起等待。<br>
函数声明：
```c++
int pthread_mutex_lock(pthread_mutex *mutex);
```
### 3）非阻塞加锁
函数功能当请求资源的锁被占据的时候，立即返回EBUSY，不会挂起等待。<br>
函数声明：
```c++
int pthread_mutex_trylock(pthread_mutex_t *mutex);
```
### 4）解锁
函数功能锁是lock状态，需要由加锁线程解锁。<br>
函数声明：
```c++
int pthread_mutex_unlock(pthread_mutex_t * mutex);
```
### 5）销毁锁
函数功能：锁为unlock状态，销毁锁，否则返回EBUSY。<br>
函数声明：
```c++
int pthread_mutex_destroy(pthread_mutex_t *mutex);
```
