## synchronized
### synchronized 关键字的用法
1. 修饰代码块，指定加锁对象，对给定对象加锁，进入同步代码库前要获得给定对象的锁。
2. 修饰普通方法，对象锁，进入同步代码前要获得当前对象实例的锁
3. 修饰静态方法，类锁（Demo.class），所有对象共用一把锁，进入同步代码前要获得当前类对象的锁。 
### synchronized 内部实现 
1. 锁的状态记录在对象头的 mark word 中 
	- 001 无状态
	- 101 偏向锁 
	- 00 轻量级锁 
	- 10 重量级锁 
2. Lock Record
	- 线程在执行同步块之前，JVM会先在当前的线程的栈帧中创建一个 Lock Record
	- 包括两部分
		- Displaced Mark word：用于存储对象头中的 mark word 
		- Object Reference：指向对象的指针
1. 偏向锁
	- `mark word` 中存储的是偏向的线程ID；
	-  当来了新的线程，需要升级为轻量级锁！
	- 对象创建
		- 当新创建一个对象时，如果该对象所属的 class 没有关闭偏向锁模式，那新建的对象处于可偏向状态，此时 mark word中的 thread id 为0，表示未偏向任何线程，也叫做匿名偏向。
	- 加锁过程
		- 如果此时处于匿名偏向状态，将mark word中的thread id 由 0 改为当前线程 id。如果成功，则成功获得偏向锁。
	- 重入过程
		- 当偏向的线程再次进入同步块时，发现锁对象偏向的就是当前线程，就会往当前的栈中添加一条 Displaced Mark Word 为 null 的 Lock Record，将 Lock Record 的 obj 指向该对象。、
	- 产生竞争
		- 如果当前对象已经被其他线程获取偏向锁，则会走到`撤销偏向锁`的过程，如果占用线程还存活，则升级为轻量级锁，原偏向线程继续拥有锁，当前线程进行锁升级。
	- 解锁过程
		- 遍历栈，将栈中最近一条的 Lock Record 的 obj 字段设为 null，并不会修改对象头中的thread id。
2. 轻量级锁
	- `mark word` 中存储的是指向线程栈中的 `Lock Record` 的指针

	- 加锁过程
		- 在线程中创建一个 `Lock Record` 将其 `obj` 字段指向锁对象
		- 通过 `CAS` 指令将 `Lock Record` 的地址存储在对象头的 `mark word` 中，如果成功，获得轻量级锁，否则进入自旋状态
		- 当有线程超过10次自旋/自旋线程数超过cpu核数的一半，就需要升级为重量级锁。
	- 锁重入
		- 如果当前线程已经持有锁了，代表这是一次重入过程，添加一个指向该对象的，displaced mark word 为 null，的Lock Record。
	- 解锁过程 
		- 遍历线程栈，找到所有 obj 字段等于当前锁对象的 Lock Record。
		- 如果 Lock Record 的 Displaced Mark Word 为 null， 代表这是一次重入，将 obj 设置为 null 后 continue
		- 如果 Lock Record 的 Displaced Mark Word 不为 null，利用 CAS 指令将对象头的 mark word 恢复为 Displaced Mark Word。
3. 重量级锁
	- 通过 monitorenter 和 monitorexit两个指令来获取/释放 monitor 对象！
	- `mark word` 中存储的是指向堆中的 `monitor` 对象的指针
	- `monitor` 对象包括以下几个关键字
		- cxq：ObjectWaiter 链表结构，
		- EntryList：ObjectWaiter 链表结构
		- WaitSet：ObjectWaiter 链表结构
		- owner：指向持有锁的线程
	- 当一个线程尝试获取锁时，如果该锁已经被占用，则会将该线程封装成一个 ObjectWaiter 对象插入 cxq 队列尾部，然后暂停该线程。
	- 当持有锁的线程释放锁前，会将 cxq 中的所有元素移动到 EntryList 中去，并唤醒 EntryList 的队首线程。
	- 当一个线程在同步块中调用了 `Object::wait` 方法，会将该线程对应的 `ObjectWaiter` 从 `EntryList` 移除，并加入 `WaitSet` 中，然后释放锁。
	- 当 `wait` 的线程被 `notify` 之后，会将对应的 ObjectWaiter 从 `WaitSet` 移动到 `EntryList` 中。
### Monitor
1. Java虚拟机为每个对象和class字节码都设置了一个监听器Monitor，用于检测并发代码的重入。
2. Monitor结构 
	- wait set：存放处于wait状态的线程的ObjectWaiter类型的队列
	- entry set：存放等待锁的线程的ObjectWaiter类型的队列 
	- owner：用于指向获得 monitor 的线程 
3. 获取锁
	- 如果monitor对象的owner为空，那么将owner设置为当前线程
	- 如果monitor对象的owner是当前线程，那么是重入操作，将 recursions的值+1
	- 如果owner不是当前线程，先通过多次自旋尝试获取锁（非公平的体现），失败后将当前线程插入 cxq队列，并调用 park 挂起
3. 三个队列 
	- cxq ：等待竞争锁的队列 （cxq也被称为ContentionList，并不是真实存在的队列，而是由 node 和 next 指针组成的逻辑队列）
	- entryset：竞争锁的队列
	- waitset：等待队列！
3. Object.wait()
	- 将当前线程封装成 ObjectWaiter对象node
	- 将 node 加入到 wait set队列末尾
	- 通过 monitorexit释放 monitor 
	- 底层 park 挂起线程
4. Object.notify()
	- 从 wait set 中取出一个线程 
	- 唤醒该线程之后，根据策略选择将其放入 cxq队列或者entryset（entrylist）队列。
	- notify 并不会释放monitor
	- 如果 notify 时，没有线程处于等待状态，那么notify不会起作用（这点和 LockSupport.unpart 不同，LockSupport会保留一次 unpark的效果！condition.await和condition.signal 也是这样吗？）
### synchronized 常见问题
1. synchronized 会造成死锁吗
	- 两个线程在分别获取自己的锁之后再尝试获取对方的锁就会造成死锁
	https://github.com/duanxinpeng/Java-Examples/blob/master/src/concurrent/SynchronizedTest.java
	- 但是，同一个方法内的不同 synchronized 方法之间相互调用并不会造成死锁，因为 synchronized 是可重入的！
2. 用字符串作为 synchronized 锁住的对象时，一旦对字符串做了修改，字符串对象就发生了改变，所以 synchronized 锁就失效了！
3. synchronized 和 ReentrantLock 的区别
	- Synchronized是JVM层次的锁实现，ReentrantLock是JDK层次的锁实现；
	- Synchronized的锁状态是无法在代码中直接判断的，但是ReentrantLock可以通过ReentrantLock#isLocked判断；
	- Synchronized是非公平锁，ReentrantLock是可以是公平也可以是非公平的；
	- Synchronized是不可以被中断的，而ReentrantLock#lockInterruptibly方法是可以被中断的；
	- 在发生异常时Synchronized会自动释放锁（由javac编译时自动实现），而ReentrantLock需要开发者在finally块中显示释放锁；
	- ReentrantLock获取锁的形式有多种：如立即返回是否成功的tryLock(),以及等待指定时长的获取，更加灵活；
	- Synchronized在特定的情况下对于已经在等待的线程是后来的线程先获得锁（FILO），而ReentrantLock对于已经在等待的线程一定是先来的线程先获得锁（FIFO）？
4. 为什么要设计偏向锁？
	- 大部分情况下都是只有一个线程在运行
	- 去掉了线程的加锁过程，减轻了线程负担
5. 轻量级锁一定比重量级锁效率高吗？
	- 不一定，轻量级锁消耗 CPU 资源，
	- 当一个线程自旋次数过多，或者有过多的线程处于自旋状态的时候，升级锁为重量级锁更好一些。
6. 为什么 JVM 启动四秒之后才会启动偏向锁？
	- 因为在 JVM 启动过程中肯定有锁竞争。
## AQS
### 继承关系
1. AbstractQueuedSynchronizer
	- Sync 
		- FairSync
		- NofairSync 
### state
1. AQS底层维护一个由 volatile 修饰的 int 类型的变量 state 来表示当前的同步状态。
2. 关于 state 主要有以下三个方法
	- getState： 获取当前同步状态
	- setState： 设置当前同步状态
	- compareSetAndSetState：使用 `CAS` 设置当前同步状态。
### AQS提供的模板方法（模板方法设计模式）
1. 独占式地获取和释放锁
	- acquire()
		- tryAcquire(arg)需要子类实现
	- release()
		- tryRelease(arg)需要子类实现
2. 共享式地获取和释放锁
	- acquireShared()
		- tryAcquireShared(arg)需要子类实现
	- releaseShared()
		- tryReleaseShared(arg)需要子类实现
3. 查询 AQS 地同步状态队列正在等待的线程情况
	
### 设计模式
1. 模板方法设计模式
2. 为什么不直接定义为抽象方法呢?
	- 因为子类只要实现模板方法中的一部分就可以实现一个同步器了，所以不需要定义成抽象方法。

### AQS如何实现线程同步
1. 通过一个 state 来记录当前同步状态
2. 通过一个同步队列来维护当前获取锁失败，进入阻塞状态的线程。这个同步队列是一个双向链表，获取锁失败的线程会被封装成一个链表节点，加入链表的尾部排队，同时调用 LockSupport.park() 方法阻塞线程。
### 独占锁的获取与释放
1. acquire() 
	- 尝试获取一次锁（tryAcquire() 需要使用者按照需求自行定义！）
	- 若获取锁失败则将当前线程封装成同步队列的节点，利用CAS将其加入等待队列中。（addWaiter）
	- 将线程阻塞，直到他成为头节点的下一个节点，才能获得锁 （acquireQueued）
2. release()
	- 尝试释放锁，（ tryRelease() 自定义）
	- 唤醒头节点的下一个节点，使其尝试去获得锁。
### 共享锁的获取与释放
1. acquireShared()
	- 尝试获取一次锁。（tryAcquireShared() 自定义）。
	- 如果获取锁失败，则执行 doAcquireShared 方法，该方法和 acquireQueued 方法类似，但有一点不同`若线程在被唤醒后，成功获取到了共享锁，还需要判断共享锁是否还能被其他线程获取，若可以，则继续向后唤醒他的下一个节点对应的线程`。
2. releaseShared()
	- 尝试释放锁（tryRelease() 自定义)
	- 调用 doReleaseShared 方法唤醒后继节点中的线程。
	
### AQS同步队列
1. AQS队列 
2. Condition队列
	- AQS还有一个很重要的内部类ConditionObject，它实现了Condition接口，主要用于实现条件锁
### ConditionObject
1. await()
	- 
2. signal()
	- 从条件队列中取出一个线程，并将其唤醒
	- 只有条件队列不为空时，才会做唤醒操作，否则什么都不用做，所以先调用signal，再调用await是无法唤醒的！
	- 上面这一点和 Object.wait()/Object.notify()是一致的，但是和 LockSupport.park/LockSupport.unpart是不一致的。
### 使用了AQS的同步工具（只需要实现tryAcquire和tryRelease，其他的逻辑都是固定的！）
1. ReentrantLock
2. ReentrantReadWriteLock
3. Semaphore
4. CountDownLatch

### 使用AQS实现一个共享锁
`实现一个锁，它是一个共享锁，但是每次至多支持两个线程同时获取锁，若当前已经有两个线程获取了锁，则其他获取锁的线程需要等待`

	
```java
/**
 * 抽象队列同步器（AQS）使用：
 *      实现一个同一时刻至多只支持两个线程同时执行的同步器
 */

// 让当前类继承Lock接口
public class TwinLock implements Lock {

	// 定义锁允许的最大线程数
	private final static int DEFAULT_SYNC_COUNT = 2;
	// 创建一个锁对象，用以进行线程同步，Sync继承自AQS
	private final Sync sync = new Sync(DEFAULT_SYNC_COUNT);

	// 以内部类的形式实现一个同步器类，也就是锁，这个锁继承自AQS
	private static final class Sync extends AbstractQueuedSynchronizer {

		// 构造方法中指定锁支持的线程数量
		Sync(int count) {
			// 若count小于0，则默认为2
			if (count <= 0) {
				count = DEFAULT_SYNC_COUNT;
			}
			// 设置初始同步状态
			setState(count);
		}
		
		/**
		 * 重写tryAcquireShared方法，这个方法用来修改同步状态state，也就是获取锁
		 */
		@Override
		protected int tryAcquireShared(int arg) {
			// 循环尝试
			for (; ; ) {
				// 获取当前的同步状态
				int nowState = getState();
				// 计算当前线程获取锁后，新的同步状态
				// 注意这里使用了减法，因为此时的state表示的是还能支持多少个线程
				// 而当前线程如果获得了锁，则state就要减小
				int newState = nowState - arg;
				
				// 如果newState小于0，表示当前已经没有剩余的资源了
				// 则当前线程不能获取锁，此时将直接返回小于0的newState；
				// 或者newState>0，就会执行compareAndSetState方法修改state的值，
				// 若修改成功将，将返回大于0的newState；
				// 若修改失败，则表示有其他线程也在尝试修改state，此时循环一次后，再次尝试
				if (newState < 0 || compareAndSetState(nowState, newState)) {
					return newState;
				}
			}
		}

		/**
		 * 尝试释放同步状态
		 */
		@Override
		protected boolean tryReleaseShared(int arg) {
			for (; ; ) {
				// 获取当前同步状态
				int nowState = getState();
				// 计算释放后的新同步状态，这里使用加法，
				// 表示有线程释放锁后，当前锁可以支持的线程数量增加了
				int newState = nowState + arg;
				// 使用CAS修改同步状态，若成功则返回true，否则自旋
				if (compareAndSetState(nowState, newState)) {
					return true;
				}
			}
		}
		
	}


	/**
	 * 获取锁的方法
	 */
	@Override
	public void lock() {
		// 这里调用的是AQS的模板方法acquireShared，
		// 在acquireShared中将调用我们重写的tryAcquireShared方法
		// 传入参数为1表示当前线程，当前线程获取锁后，state将-1
		sync.acquireShared(1);
	}

	/**
	 * 解锁
	 */
	@Override
	public void unlock() {
		// 这里调用的是AQS的模板方法releaseShared，
		// 在acquireShared中将调用我们重写的tryReleaseShared方法
		// 传入参数为1表示当前线程，当前线程释放锁后，state将+1
		sync.releaseShared(1);
	}

	/*******************其他需要实现的方法省略***************************/

}
```
```java
public static void main(String[] args) throws InterruptedException {
	// 创建一个我们自定义的锁对象
    Lock lock = new TwinLock();

    // 启动10个线程去尝试获取锁
    for (int i = 0; i < 10; i++) {
        Thread t = new Thread(()->{
            // 循环执行
            while (true) {
                // 获取锁
                lock.lock();
                try {
                    // 休眠1秒
                    Thread.sleep(1000);
                    // 输出线程名称
                    System.out.println(Thread.currentThread().getName());
                    // 再次休眠一秒
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    // 释放锁
                    lock.unlock();
                }
            }
        });
		// 将线程设置为守护线程，主线程结束后，收获线程自动结束
        t.setDaemon(true);
        t.start();
    }

	// 主线程每隔1秒输出一个分割行
    for (int i = 0; i < 10; i++) {
        Thread.sleep(1000);
        System.out.println("********************************");
    }
}
```
## LockSupport
1. 用 LockSupport 实现一个 FIFO（公平）的 不可重入的锁！
	- 用一个 AtomicBoolean 变量来表示锁的获取状态
	- 用 LockSupport 的 park，unpark 来阻塞、唤醒线程 
	- 用一个 ConcurrentLinkedQueue 来存放被阻塞的线程。
2. LockSupport.park() 会释放锁吗？
	- 不会，condition.await() 才会释放锁
3. 先调用 LockSupport.unpark(thread1), 再调用 LockSupport.park() 会怎样？
	- LockSupport.park()会被直接跳过，（如何理解？会被直接跳过的意思是unpartk生效了，但只会生效一次，但是wait和notify就不是这样）
4. LockSupport 原理
	- LockSupport 会为每一个调用它的线程搭配一个许可（permit），这一点类似于 Semaphore，每次调用 park 时，如果许可可用，则会直接返回，否则该线程会被阻塞。
	- 调用 unpark 时，会使许可可用，但是如果许可已经可用，这个许可并不会累积，这一点和 Semaphore 不同！所以最多只会有一个许可！
## CAS
1. 本质上就是一个乐观锁
2. 无法保证代码块的原子性，只能保证变量的原子操作
2. ABA问题
	- 指的是其他线程进行了多次操作之后，将变量恢复为原来的样子，线程无法区分是否发生了改变。
	- 通过加版本号可以解决该问题，jdk1.5之后提供了 AtomicStampedReference类来解决这个问题。
3. 是否具有原子性：底层实现，通过对总线加锁！
## volatile
1. 

## Atomic
1. 


## 常见问题
