
## java线程状态
我们都知道进程的状态有五种：创建、就绪、运行、阻塞、终止。但java线程状态不同于进程状态，java进程包括一下六种状态：
1. New（初始）：新创建了一个线程对象，但还没有调用start()方法；
2. Runnable（运行）：java将进程中的就绪(ready)和运行(running)统称为“运行”。调用start()方法之后，线程或处于就绪队列中等待调度或已经获得CPU时间片正在运行。
3. Blocked（阻塞）：阻塞于锁，synchronized关键字，线程位于锁的等待队列中，和等待的区别在于，处于阻塞状态的线程在等待获取一个排他锁，这个事件将在另外一个线程放弃这个锁时发生。
4. Waiting（等待）：调用wiat、join、park等方法，不会被分配处理器执行时间，需要等待其他线程显式唤醒。
5. Timed_Waiting（超时等待）：调用sleep等方法，也不会被分配处理器执行时间，但不需要其他线程显式唤醒，一定时间之后系统自动唤醒线程。
6. Terminated（终止）：表示该线程已经执行完毕。

线程状态变化如下图所示：
![](/media/threadstate.png)

## 线程的实现
1. 内核线程实现，也被称为1:1实现（进程和线程1:1），直接由操作系统内核支持的线程，由内核完成切换，内核通过操纵调度器对线程进行调度，这种需要操作系统配合的都是重量级线程。
2. 用户线程实现，也被称为1:N实现（进程和用户级线程1:N），完全建立在用户空间的线程库上，系统内核完全不能感知到用户线程的存在，用户线程的建立、销毁、同步、调度完全在用户态中完成，不需要内核帮助；其优点是不需要系统内核支援，其劣势也在于此：由于操作系统只把处理器资源分配到进程，那诸如“阻塞如何处理（自旋）”，“多处理器系统如何将线程映射到其他处理器上”这类问题解决起来会异常困难。
3. 混合实现，也被称为N:M实现（进程和线程N:M）。

## java线程的实现
java采用内核线程实现，即1:1实现。每一个java线程都会直接映射到一个操作系统原生线程，线程的切换、调度全权由操作系统完成。

## 线程调度
线程调度指系统为线程分配处理器使用权的过程，主要分为两种方式：协同式、抢占式；
- 协同式：线程把自己的工作执行完之后，要主动通知系统切换到另外一个线程；优点是简单，缺点是线程执行时间不可控。
- 抢占式：每个线程由系统来分配执行时间，线程的切换不由线程本身决定。优点是线程执行时间可控。java采用抢占式调度。

## java线程和协程
1. 内核线程的调度成本高：用户态与和心态之间的状态转换，响应中断、保护和回复现场。
2. 如果采用用户线程，保护和恢复现场的花销能省略掉吗？不能，但是如果把保护和恢复现场及调度的工作从操作系统交到程序员手上，那我们就可以打开脑洞，通过玩出各种花样来缩减这些开销
	- 有栈协程：用户自己模拟多线程，在内存中划分出一片额外空间来模拟调用栈。java里的纤程就是有栈协程。
	- 无栈协程：本质上是一种有限状态机，状态保存在闭包里。

## java创建线程的几种方式
1. 三种方式：
https://github.com/duanxinpeng/Java-E-xamples/blob/master/src/concurrent/ThreadCreate.java
	- 继承Thread类，重写该类的run方法
	- Runnable接口（run）+ Thread
	- Callable接口(call) + FutureTask + Thread
2. 三种方式的对比
	- Thread类的方式在继承Thread类之后就不能再继承其他类。
	- Runnable/Callable接口适合多个线程进行资源共享
	- Callable接口的call方法可以有返回值，Runnable接口的run方法不能有返回值
	- 运行一个Callable任务可以拿到一个FutureTask对象，通过FutureTask对象可以了解任务执行情况，可取消任务的执行，还可以获取线程返回值。
3. 思考
	- 多线程发起多个任务，如果所有校验结果均为true，则返回true，只要有一个返回false则返回false，尽量快，请完成isAllTrue方法。
	- 分布式事务的实现：collable？
	- jdk8之前没有线程返回之后及时反馈的机制，所以要用到guava？
	- 或者用jdk8之后才有的 CompleteableFuture、ListeningFutre实现！
## 线程池
1. 作用：通过维护一定数量的线程来达到多个线程的复用！
2. 好处：线程复用节省了线程创建消耗掉的资源
3. 本质：本质就是一个生产者消费者模型，系统不断生成task（本质就是一个Runnbale接口的实现类），线程不断从阻塞队列中取任务。
3. 线程池的几种重要参数
	- corePoolSize：核心线程池的大小
	- maximumPoolSize：线程池最大线程数
	- keepAliveTime：（核心）线程的最大存活时间
	- unit：keepAliveTime的时间单位
	- workQueue：阻塞队列，用来存储等待执行的任务
	- threadFactory：创建线程的工厂
	- handler：拒绝任务时的策略
4. 线程池工作流程
	- 向线程池提交一个任务，如果此时线程池中线程数小于corePoolSize，线程池会创建一个线程去处理提交的任务
	- 如果线程数大于等于corePoolSize， 新提交的任务会被放入任务队列中。
	- 若线程数等于corePoolSize且任务队列已满，则进一步判断线程数是否达到最大线程数（maximumPoolSize）如果还没达到，则继续创建一个线程执行任务。
	- 如果线程数已经达到最大线程数，对新来的任务采用拒绝策略。
5. 线程池中的常用阻塞队列：（大概和生产者消费者的实现类似，由很多种方式比如Synchronized关键字+wait/notify，可重入锁+condition）
	- ArrayBlockingQueue：基于数组的阻塞队列，有界队列，FIFO
	- LinkedBlockingQueue：基于链表的阻塞队列，无界队列，FIFO
	- SynchronousBlockingQueue：容量为0，不存储任务，有一个处理一个
	- PriorityBlockingQueue：基于堆的阻塞队列，无界有序!
6. 常见线程池（利用Executors直接生成）
	- newCachedThreadPool：corePoolSize=0，maximumPoolSize=Max_value，workQueue=synchronoutQueue，适用于很多短期异步的小程序。
	- newFixedThreadPool：corePoolSize=maximumPoolSize=nThread，workQueue=LinkedBlockingQueue，keepAlivetimie=0（不限时），适用于长期任务
	- newSingleThreadExecutor
	- newScheduledThreadPool
7. 拒绝策略
	- 抛异常
	- 丢弃
	- 丢弃最老的任务
8. 如何提交一个线程
	- execute：没有返回值
	- submit：返回一个Future对象，有返回值
9. 如何关闭线程池
	- shutdown：不接受新任务，将线程池内任务执行完毕
	- shutdownNow：不接受新任务，并试图停止线程池中的任务。
10. 使用Executors构建线程池的缺点，应该使用(ThreadPoolExecutor)
	- Executors提供的很多方法都是默认无界的，无界队列容易导致OOM。
	- FixedThreadPool和SingleThreadPool：阻塞队列无界！
	- CachedThreadPool和Scheduled THreadPool：maximumPoolSize=Integer.MAX_VALUE；
11. https://github.com/duanxinpeng/Java-E-xamples/blob/master/src/concurrent/TheadPoolTest.java	
12. 线程池状态（五种）
	- RUNNING：接受新任务并且处理阻塞队列中的任务
	- SHUTDOWN：拒绝新任务但是处理阻塞队列中的任务（shutdown())
	- STOP：拒绝新任务并且抛弃阻塞队列中的任务，同时中断正在处理的任务(shutdownNow())
	- TIDYING：所有任务都执行完
	- TERMINATED
13. 源码细节
	- 成员变量ctl是AtomicInteger类型，高三位用于表示线程池状态，后面29位用来记录线程池线程个数。
	- 实际上是一个生产者消费者模型，用户提娜佳任务到线程池时相当于生产者生产元素，workers线程工作集中的线程直接执行任务或者从任务队列里面获取任务相当于消费元素。
	- 当线程数小于coresize或者阻塞队列满时，任务都不会经过阻塞队列，而是会直接被新创建的线程执行，因为任务会直接与worker绑定
	- Worker
		- 继承了AQS，并且实现了Runnable
		- final Thread thread 
		- Runnable firstTask 
		- volatile long completedTasks
	- Worker每次创建时都会和一个task绑定，执行时优先执行firstTask，然后再选择从阻塞队列中取任务
	- takeTask（）就是从阻塞队列中取任务，就是一个消费者。如果线程数大于核心线程数，就用poll方法，可以设置超时时间，一旦超时就销毁线程
	- 如果线程数小于核心线程数，会用take方法，不设置超时时间
