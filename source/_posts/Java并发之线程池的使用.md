---
title: Java并发之线程池的使用
date: 2019-03-18 15:36:00
categories: "Java并发"
tags:
    - Java并发
copyright:
---


### 无限制创建线程的不足
- **线程生命周期的开销非常高**

> 线程的创建与销毁并不是没有代价的。根据平台的不同， 实际的开销也有所不同， 但线程的创建过程都会需要时间， 延迟处理的请求， 并且需要JVM和操作系统提供一些辅助操作。

- **资源消耗**

> 活跃的线程会消耗系统资源， 尤其是内存。如果可运行的线程数量多于可用处理器的数量， 那么有些线程将闲置。大量空闲的线程会占用许多内存， 给垃圾回收器带来压力， 而且大量线程在竞争CPU资源时还将产生其他的性能开销。如果你已经拥有足够多的线程使所有CPU保持忙碌状态， 那么再创建更多的线程反而会降低性能。

- **稳定性**

> 在可创建线程的数批上存在一个限制。这个限制值将随着平台的不同而不同， 并且受多个因素制约， 包括JVM的启动参数、Thread构造函数中请求的栈大小， 以及底层操作系统对线程的限制等。如果破坏了这些限制， 那么很可能抛出OutOfMemoryError 异常，想从这种错误中恢复过来是非常危险的。

---

### 什么是线程池
线程池就是提前创建若干个线程，如果有任务需要处理，线程池里的线程就会处理任务，处理完之后线程并不会被销毁，而是等待下一个任务。

### 使用线程池的好处
- **降低资源消耗**。通过重复利用已创建的线程降低线程创建和销毁造成的消耗
- **提高响应速度**。当任务到达的时候，任务可以不需要等待线程创建就能立即执行。
- **提高线程的可管理性**。线程是稀缺资源，如果无限制地创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一分配、调优、监控。

### 线程池的类图
![](/images/java_concurrency_threadpool_class.png)


### 线程池的状态：运行、关闭、已终止。
![](/images/java_concurrency_threadpool_status.png)

---

### 线程池的使用：ThreadPoolExecutor
#### 线程池的创建

`new ThreadPoolExecutor(corePoolSize, maximumPoolSize, keepAliveTime,milliseconds,runnableTaskQueue, handler);`

创建一个线程池时需要输入几个参数，如下：

- corePoolSize：线程池的基本大小
- runnableTaskQueue：任务队列，用于保存等待执行的任务的阻塞队列。可以选择以下几个阻塞队列。
	- ArrayBlockingQueue：是一个基于数组结构的有界阻塞队列，此队列按FIFO（先进先出）原则对元素进行排序。
	- LinkedBlockingQueue：一个基于链表结构的阻塞队列，此队列按FIFO排序元素，吞吐量通常要高于ArrayBlockingQueue。静态工厂方法Executors.newFixedThreadPool()使用了这个队列。
	- SynchronousQueue：一个不存储元素的阻塞队列。每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，吞吐量通常要高于Linked-BlockingQueue，静态工厂方法Executors.newCachedThreadPool使用了这个队列。
	- PriorityBlockingQueue：一个具有优先级的无限阻塞队列。
- maximumPoolSize：线程池允许创建的最大线程数。如果队列满了，并
且已创建的线程数小于最大线程数，则线程池会再创建新的线程执行任务。

> 值得注意的是，如果使用了无界的任务队列这个参数就没什么效果。

- ThreadFactory：用于设置创建线程的工厂，可以通过指定一个线程工厂方法，定制线程池的配置信息。

> 如：使用开源框架guava提供的ThreadFactoryBuilder可以快速给线程池里的线程设置有意义的名字，代码如下:`new ThreadFactoryBuilder().setNameFormat("XX-task-%d").build();`

- RejectedExecutionHandler：饱和策略,当队列和线程池都满了，说明线程池处于饱和状态，那么必须采取一种策略处理提交的新任务。在JDK 1.5中Java线程池框架提供了以下4种策略。
	- AbortPolicy：直接抛出异常。(默认情况下是这种策略)
	- CallerRunsPolicy：只用调用者所在线程来运行任务。
	- DiscardOldestPolicy：丢弃队列里最近的一个任务，并执行当前任务。
	- DiscardPolicy：不处理，丢弃掉。

> 当然，也可以根据应用场景需要来实现RejectedExecutionHandler接口自定义策略。如记录日志或持久化存储不能处理的任务。

- keepAliveTime：线程活动保持时间,线程池的工作线程空闲后，保持存活的时间。所以，如果任务很多，并且每个任务执行的时间比较短，可以调大时间，提高线程的利用率。

- TimeUnit：线程活动保持时间的单位,可选的单位有天（DAYS）、小时（HOURS）、分钟（MINUTES）、毫秒（MILLISECONDS）、微秒（MICROSECONDS，千分之一毫秒）和纳秒（NANOSECONDS，千分之一微秒）。

> 在调用完ThreadPoolExecutor的构造函数后，仍然可以通过设置函数来修改大多数传递给它的构造函数的参数。如果Executor是通过Executors中的某个（newSingleThreadExecutor除外）工厂方法创建的，那么可以将结果的类型转换为ThreadPoolExecutor以访问设置器。
ps: 在Executors中包含一个unconfigurableExecutorService工厂方法，该方法对一个现有的ExecutorService进行包装，使其只暴露出ExecutorService的方法，因此不能对它进行配置。newSingleThreadExecutor返回的按这种方式封装的ExecutorService，而不是最初的ThreadPoolExecutor。可以使用这项技术防止执行策略被修改。

#### 向线程池提交任务
可以使用两个方法向线程池提交任务，分别为execute()和submit()方法。
- **execute()**：execute()方法用于提交不需要返回值的任务，所以无法判断任务是否被线程池执行成功。
- **submit()**：submit()方法用于提交需要返回值的任务。线程池会返回一个future类型的对象，通过这个future对象可以判断任务是否执行成功，并且可以通过future的get()方法来获取返回值

> get 方法的行为取决于任务的状态（尚未开始、正在运行、已完成）。如果任务已经完成，那么 get 会立即返回或者抛出一个 Exception, 如果任务没有完成， 那么 get将阻塞并直到任务 完成。 如果任务抛出了异常， 那么 get 将该异常封装为 ExecutionException 并重新抛出。 如果 任务被取消，那么 get 将抛出CancellationException。

#### 关闭线程池
可以通过调用线程池的shutdown或shutdownNow方法来关闭线程池。只要调用了这两个关闭方法中的任意一个，isShutdown方法就会返回true。当所有的任务
都已关闭后，才表示线程池关闭成功，这时调用isTerminaed方法会返回true。

#### 合理地配置线程池
想合理地配置线程池，就必须首先分析任务特性，可以从以下几个角度来分析:
- 任务的性质：CPU密集型任务、IO密集型任务和混合型任务。

> 性质不同的任务可以用不同规模的线程池分开处理。CPU密集型任务应配置尽可能小的线程，如配置Ncpu+1个线程的线程池。由于IO密集型任务线程并不是一直在执行任务，则应配置尽可能多的线程，如2*Ncpu。混合型的任务，如果可以拆分，将其拆分成一个CPU密集型任务和一个IO密集型任务，只要这两个任务执行的时间相差不是太大，那么分解后执行的吞吐量将高于串行执行的吞吐量。如果这两个任务执行时间相差太大，则没必要进行分解。

- 任务的优先级：高、中和低。

> 优先级不同的任务可以使用优先级队列PriorityBlockingQueue来处理。它可以让优先级高的任务先执行。

- 任务的执行时间：长、中和短。

> 执行时间不同的任务可以交给不同规模的线程池来处理，或者可以使用优先级队列，让执行时间短的任务先执行。

- 任务的依赖性：是否依赖其他系统资源，如数据库连接。

> 依赖数据库连接池的任务，因为线程提交SQL后需要等待数据库返回结果，等待的时间越长，则CPU空闲时间就越长，那么线程数应该设置得越大，这样才能更好地利用CPU。

#### 线程池的监控
以通过线程池提供的参数进行监控，在监控线程池的时候可以使用以下属性:
- taskCount：线程池需要执行的任务数量。
- completedTaskCount：线程池在运行过程中已完成的任务数量，小于或等于taskCount。
- largestPoolSize：线程池里曾经创建过的最大线程数量。通过这个数据可以知道线程池是否曾经满过。如该数值等于线程池的最大大小，则表示线程池曾经满过。
- getPoolSize：线程池的线程数量。如果线程池不销毁的话，线程池里的线程不会自动销
毁，所以这个大小只增不减。
- getActiveCount：获取活动的线程数。

通过扩展线程池进行监控。可以通过继承线程池来自定义线程池，重写线程池的beforeExecute、afterExecute和terminated方法，也可以在任务执行前、执行后和线程池关闭前执行一些代码来进行监控。


---

### 线程池的使用:Excutors
可以通过调用Excutors中的静态工厂方法之一来创建一个线程池

#### newFixedTheadPool
创建一个固定长度的线程池，每当提交一个任务时就创建一个线程， 直到达到线程池的最大数量， 这时线程池的规模将不再变化（如果某个线程由于发生了未预期的Exception而结束，那么线程池会补充一个新的线程）。

#### newCachedThreadPool
创建一个可缓存的线程池，如果线程池 的当前规模超过了处理需求时，那么将回收空闲的线程，而当需求增加时，则可以添加新的线程， 线程池的规模不存在任何 限制。

#### newScheduledThreadPool
创建了一个固定长度的线程池， 而 且以延迟或定时（周期性地）的方式来执行任务， 类似千Timer。

#### newSingleThreadExecutor
一个单线程的Executor, 它创建单个工作者线程来执行任务， 如果这个线程异常结束， 会创建另一个线程来替代。

> newSingleThreadExecutor 能确保依照任务在队列中的顺序来串行执行（例如FIFO、LIFO、优先级）。