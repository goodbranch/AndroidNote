
## 第一章 简介

* 利用并发提高CPU使用效率。

* 并发带来资源竞争，合理处理资源竞争增加性能。

## 第二章 线程安全性

### 2.1 线程安全的定义

> 当多个线程访问某个类时，这个类始终都能表现出正确的行为，那么就称为这个类是线程安全。

### 2.2 原子性

> 同一个代码同一时间有且仅有一个在操作。

### 2.3 加锁机制

* concurrent 包中的线程安全类

* Synchronized Block

* 锁分成对象锁和类锁

* 重入

> 当某个线程请求一个由其他线程持有的锁时，发出请求的线程就会阻塞，然而，由于内置锁是可重入的，因此如果某个线程试图获得一个已经由其他自己持有的锁，那么这个这个请求不会被阻塞。例如子类重写父类Synchronized 块方法时也加了锁，并且同时调用super方法，则按照逻辑看这是一个死锁现象，当时因为重入机制而不会阻塞。

* 尽可能简化锁的范围

## 第三章 对象的共享

* 可见性

> 不同线程始终可以获取变量最新状态。

* 非原子的64位操作

> JVM允许将64位操作分成两个32位操作

* Volatile变量

> 是一种稍弱的同步机制，即volatile变量用来确保将变量的更新操作通知给其他线程。通常用作某个操作完成，发生中断或者状态的标记并且他不能保证原子性。

* 发布与逸出

> 发布:一个对象能够在当前作用域之外的代码中使用。

> 逸出:将原本不需要发布的对象也发布出去了。（例如类中的private变量有直接的get方法） 

### 线程封闭

> 仅在单线程中使用的数据就是线程封闭。

> Ad-hoc线程封闭，维护线程封闭的职责完全由程序实现来承担。

> 栈封闭，局部变量固有属性就是封闭在执行线程中。

> ThreadLocal,把值与当前线程关联起来,确保一个线程中只能有一个实例。

> 不变性，例如使用final

### 安全发布

> 使用线程安全的类可以保证线程安全

> 使用对象锁

## 第四章 对象的组合

### 4.1 设计线程安全的类

设计线程安全3要素：

* 找出构成线程状态的所有变量

* 找出约束状状态变量的不变形条件

* 建立对象状态的并发访问策略

* 将数据封装在对象内部，访问限制在对象方法上更容易确保线程在访问时持有正确的锁。

* 不可变的变量是线程安全

> 例如对象中变量为final，在构造方法中初始化后就是线程安全了。

* Collections 内部使用对象锁确保线程安全

* 在现有的线程安全类中添加功能，但需要注意避免出现增加的方法与原来类中使用了不同的锁。（锁了错误的对象）

## 第五章 基础构建模块

* 线程安全的类也可能出现意想不到的错误

* ConcurrentModiticationException,（错误时及时失败），一旦出现线程同步异常就抛出这个错误。

* 所有间接的迭代器操作都可能抛出ConcurrentModiticationException，也就是迭代过程中如果设计多线程处理需要增加锁防止异常。

* ConcurrentHashMap 使用分段锁使得任意数量的线程可以并发访问修改map，但是size和isEmpty可能不可靠，不过在大量并发中不可靠也是可以接受的，主要改进性能是在get,put,containsKey和remove等

* CopyOnWriteArrayList 用于替代List实现更好的并发性能，迭代期间也不需要加锁，这种情况主要在迭代明显大于修改插入等操作时使用。

* Deque,BlockingDeque 实现队列头和队列尾高效插入和移除，方便生产者消费者消费者。

* Thread 提供interrupt用于中断线程和查询线程是否已经被中断,告诉指定线程放弃CPU,但是并不是强制性的，所以只能是告诉正在执行的操作在某个愿意停止的地方停止。

* 闭锁，例如CountDownLatch 规定数量为0后才允许执行。FutureTask 也类似，get方法只有在执行完后才返回结果否则一直阻塞.

* 信号量，Semaphore acquire将阻塞直到有许可，release将返回一个许可。

* 栅栏，Barrier，CyclieBarrier 阻塞一组线程发生直到某个事件发生，栅栏与闭锁的关键区别在于，所有的线程必须同时到达栅栏位置才能继续执行，闭锁用于等待事件，而栅栏用于等待其他线程。

## 第一部分总结

* 可变状态是至关重要的，所有的并发问题都可以归结为如何协调并发状态的访问，可变状态越少，就越容易确保线程安全性。

* 尽量将域声明为final类型，除非需要他们是可变的。

* 不可变对象一定是线程安全。

* 封装有助于管理复杂性。

* 用锁来保护每一个可变变量。

* 当保护同一个不变形条件中的所有变量时，要使用同一个锁。

* 在执行符合操作期间，要持有锁。

* 如果多个线程访问同一个可变变量而没有同步机制，那么程序出出现问题。

* 不要故作聪明推断出不需要使用同步。

## 第六章 任务执行

* 线程开启和关闭时需要耗费资源的，并且过多的线程也会占用大量的内存。

* Executors 管理线程池

* Future 的get会一直等待线程执行完成的结果，线程完成分成正常执行完成或者是抛出异常（超时或者）

	> newFixedThreadPool 创建一个固定长度的线程池，每当提交一个任务就创建一个线程直到达到线程池最大数量，这时线程池的规模将不再变化。
	> newCachedThreadPool 创建一个可缓存的线程池，如果线程池的当前规模超过了处理需求时那么将回收空闲线程，而当需求增加时，则可以添加新的线程，线程池的规模不存在任何限制。
	> newSingleThreadExcutor 是一个单线程Executor，确保线程串行执行
	> newScheduledThreadPool 创建固定长度可以以延迟或定时的方式来执行。

* Timer 执行定时任务存在缺陷，所有的Timer任务都在一个线程中执行，则会导致某一次占用时间过长而丢失了其他timer任务的及时性或完全丢失。

* ExecutorCompletionService 封装了Executors 和BlockingQueue 可以做到一旦有完成的就可以通过take或poll获取，而Future队列实现中需要根据迭代的顺序开始一个一个等待执行完成。

* invokeAll 方法的参数为一组任务，并返回一组Future，按照任务迭代器执行顺序返回结果，如果都执行成功则返回列表，如果超时或中断则任务都会被取消。

## 第七章 取消与关闭

* Thread 并没有实际可以取消线程的方法，只能是协调机制。

* 只有实现了中断策略的代码可以屏蔽中断请求，在常规的任务和库代码中都不应该屏蔽中断请求。（InterruptedException）

* 对于持有线程的服务，只要服务的存在时间大于创建线程的方法的存在时间，那么就应该提供生命周期方法。

* 毒丸方法，当任务栈中发现毒丸则任务结束，但是毒丸之前的任务都会被执行，毒丸之后的任务不会被添加。

* JVM提供了一个未处理异常毁掉，通过实现Thread.UncaughtException

* 关闭钩子，JVM关闭时的回调

* 守护线程，不重要的线程，不会阻塞JVM的关闭。

## 第八章 线程池的使用

* 警惕线程依赖，警惕线程依赖中可能导致的线程死锁。

* 线程饥饿死锁，当线程池的中任务需要等待当前线程池的任务的资源或条件，那么除非这个线程池足够大否则都将死锁。

* 当线程池执行的线程中出现长短不一时间时会导致整个线程池性能降低。

* 线程池的大小，当任务是密集CPU操作则线程池的大小应该在avilibleProcessCount+1时性能最大，而但线程池有大量的io操作，线程池就应该更大，因为io操作中大多数耗时并且等待中，也就并不会密集占用CPU

* 当线程池中的任务是独立的情况下，有界队列才有意义，否则将导致线程饥饿。

* 对于非常大的线程池或无界线程池时应该使用无界队列来保证队列不被堵塞。

* 线程池队列饱和策略，默认使用拒绝方式，当线程队列慢了则将抛出RejectException,客户端可以捕获异常做相应的处理。

* 线程工厂，可以指定线程UnCaughtExceptionHandler

* ThreadPoolExecutor 生命周期包含beforeExecute，afterExecute和terminated

* 使用Execute 的invokeAll 实现并发递归执行。

## 第九章 图形用户界面应用程序

* GUI都是在单一线程中

* 主线程中有事件分发总线

## 第三部分 活跃性，性能与测试

## 第十章 避免活跃性危险

* 过多的锁可能导致死锁，jvm并没有主动处理死锁的机制。

* 可以通过定时的方式避免线程持续等待。

* 要避免使用线程优先级，实际上大部分平台并没有处理优先级，加入优先级反而会增加平台的依赖性。

* 活锁，过多的使用还原机制导致出错后重新处理，可以增加策略降低重试次数。

## 第十一章 性能与可伸缩性

* 可升缩性知的是：当增加计算资源时，程序的吞吐量或者吹能力也能相应的增加。

* 多线程调度切换消耗资源

* 降低锁的竞争程度

	+ 减少锁的持有时间
	+ 降低锁的请求频率
	+ 使用带有协调机制的独占锁，这些机制允许更高的并发性。

## 第十二章 并发性测试

### 性能衡量

* 吞吐量：指一组并发任务中已完成任务所占的比例。
* 响应性：指请求从开始到完成之间的时间。
* 可伸缩性：指在增加更多的资源情况下（CPU），吞吐量的提升情况。
	
* Semaphore acquire 请求一个许可，release 释放一个许可

### 常见的错误

* 不一致的同步：如果一个类频繁被访问，但是不是同一个锁那么可能有问题。

* 调用Thread.run：Thread.start才是启动线程

* 未被释放的锁：通常锁需要在finally{}中释放，否则抛出异常导致锁没有释放。

* 空的同步块:

* 双重检查加锁：if(xx==null){syxx(this){if(xx==null){}}},这种情况下可以通过Volatile解决。

* 在构造函数中启动一个线程：可能导致this持有

* 通知错误：notify,notifiAll

* 条件等待中的错误： 

* 对Lock和Condition的误用：将Lock作为同步带吗块来使用通常是一种错误的用法.

* 在休眠或等待的同时持有一个锁

* 自旋循环：浪费CPU,(while(){...})

## 第四部分 高级主题

## 第十三章 显示锁

* ReentrantLock锁，需要在finally中关闭锁，可定时，可轮询，可中断的锁操作。

* ReadWriteLock 允许多个线程读操作，只能允许一个线程写，但是读写不能并存。

## 第十四章 构建自定义的同步工具

### 状态依赖性的管理：线程是否能继续执行需要等待其他地方标记位状态改变。

* 轮询休眠方式等待状态，条件不足时使用Thread.sleep 休眠一段时间。

* 条件队列：Object 的wait，notifyAll 方式比轮询休眠方式更加高效，可以做到当条件达到时立即唤醒。

* 条件谓词：是使某个操作成为状态依赖操作的前提条件，例如take方法的条件谓词就是“列表不能为空”，

* 当使用条件等待时（例如Object.wait 或Condition.await）

	* 通常都有一个条件谓词-包括一些对象状态的测试，线程在执行钱必须首先通过这些测试。

	* 在调用wait之前测试条件谓词并且从wait中返回时在此进行测试。

	* 在一个循环中调用wait

	* 确保使用与条件队列相关的锁来保护构成条件谓词的各个状态变量。

	* 当调用wait，notify，notifyAll等方法时，一定要持有与条件队列相关的锁。

	* 在检查条件谓词之后以及开始执行相应的操作之前，不要释放锁。

* 丢失的信号：线程必须等待一个成真的条件，当时条件在等待之前已经发生，也就是线程在等待一个已经发生的信号。

* 通知：notify，notifyAll，notify在阻塞队列中唤起某一个，而notifyAll则是唤醒所有，建议使用notifyAll，如果使用notify可能出现丢失信号。

* 阀门类：闭锁，等待知道阀门打开线程才能通过。

* 使用多个显示锁能更好扩展性能。

* Synchronizer剖析：

## 第十五章 原子变量与非阻塞同步机制

## 第十六章内存模型

* 每个处理器都有自己的缓存，并且定期地与主内存进行协调。

* 双重检验加锁：最主要的是使用volatile使对象可见。

* 对于final值来说能保证一开始的可见性，而非final需要使用同步来确保可见性。

* 多个线程操作变量时，如果没有保证顺序执行，而jvm会进行重新排序，也就不能明确执行的顺序。

## 并发标注

### 类的标注

* @Immutable 不可变的，@ThreadSafe 线程安全 @NotThreadsafe 线程不安全。

* 域和方法的标注，@GuardedBy("..") 由哪个保护。

