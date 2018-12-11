---
title: Java并发
date: 2018-11-15 16:01:42
categories: "JavaSE笔记"
tags:
    - JavaSE
    - Java并发
copyright:
---

### 基本的线程机制
#### 定义一个线程
- 实现Runnable接口，重写run()方法，并传递给Thread构造器
- 继承Thread类，重写run()方法

> 调用Thread.start()方法时，其实是创建一个线程，并初始化之后，再去调run()方法

#### 使用Executor
> Executor从来管理Thread对象，在客户端和任务之间提供了一个间接层，可以管理一部任务的执行，无须显式地管理线程的生命周期。ExecutorService（具有服务生命周期的Executor）知道如何构建恰当的上下文来执行Runnable对象

有集中不同的Executor：
- **CacheThreadPool**：通常创建与所需数量相同的线程，然后在它回收旧线程时停止创建新线程（首先的Executor）
- **FixedThreadPool**：使用有限的线程集来执行所提交的任务，可以一次性预先执行代价高昂的线程分配
- **SingleThreadPool**：像是线程数量为1的FixedThreadPool，当希望在另一个线程中连续运行某事物来说很有用。当提交多个任务时，这些任务会排队。
示例（三种Executor都这样使用）：

``` java
public class SingleThreadPool {
    public static void main(String[] args) {
        ExecutorService exec = Executors.newSingleThreadPool();
        for (int i = 0; i < 5; i++) {
            exec.execute(new LiftOff());
        }
        exec.shutdown();
    }
}
```

#### 从任务中产生返回值
> 实现Callable接口，从call（）方法中返回值，必须使用ExecutorService.submit()方法调用。submit()方法会产生Future对象。可以用isDone()方法来查询Future是否完成，完成时可以调用get()方法获取该结果，若完成就调用get()方法将阻塞到完成。


```java
public class TaskWithResult implements Callable<String> {
    private int id ;
    public TaskWithResult(int id) {
        this.id = id;
    }
    public String call() {
        return "result of TaskWithResult " + id;
    }
}

public class CallableDemo {
    public static void main(String[] args) {
        ExecutorService exec = Executors.newCachedThreadPool();
        ArrayList<Future<String>> results = new ArrayList<Future<String>>();
        for (int i = 0; i < 5; i++) {
            results.add(exec.submit(new TaskWithResult(i)));
        }
        for(Future<String> fs: results) {
            try {
                System.out.println(fs.get());
            } catch {
                System.out.printLn(e);
                return;
            } finally {
                exce.shutdown();
            }
        }
    }
}
```

#### 休眠(sleep)
> sleep(),使任务中止执行给定的时间，但不会释放持有的锁

####  优先级
> Java有十个优先级，但不同的操作系统有不同的优先级，不能与操作系统有很好的映射。在调整优先级时只是用MAX_PRIORITY,NORM_PRIORITY,MIN_PRIORITY三个等级

#### 让步(yield)
> 暂停当前正在执行的线程对象，把执行机会让给相同或者更高优先级的线程。

#### 加入一个线(join)
> 某个线程在另一个线程t上个调用join()，此线程被挂起，知道目标线程t结束(即.isAlive()为假)才恢复，也可调用join()时带一个超时参数，超时便返回

#### 后台线程(维护线程)
- 可以在线程启动之前调用setDaemon()方法设置为后台线程
- 一个后台进程创建的任何线程被自动地设置为后台线程
- 后台线程在不执行finally子句时就终止器run()方法 

#### 编码变体
> 有时候可以使用内部类将代码隐藏在类中

- 继承Thread的两种写法：
1.扩展自Thread的匿名内部类

``` java
class InnerThread1 {
  private Inner inner;
  private class Inner extends Thread {
    Inner(String name) {
      super(name);
      start();
    }
    public void run() {
        ......
    }
  }
  public InnerThread1(String name) {
    inner = new Inner(name);
  }
}
```

2.在构造器中穿件一个匿名的Thread子类，并且向上转型为Thread引用t

``` java
class InnerThread2 {}
  private Thread t;
  public InnerThread2(String name) {
    t = new Thread(name) {
      public void run() {
        ......
      }
    };
    t.start();
  }
}
```

- 实现Runnable的两种写法（同上两种情况）

```java
class InnerRunnable1 {
    private Inner inner;
    private class Inner implements Runnable {
    Thread t;
    Inner(String name) {
      t = new Thread(this, name);
      t.start();
    }
    public void run() {
      ......
    }
  }
  public InnerRunnable1(String name) {
    inner = new Inner(name);
  }
}
```

```java
class InnerRunnable2 {
  private Thread t;
  public InnerRunnable2(String name) {
    t = new Thread(new Runnable() {
      public void run() {
        ......
    }, name);
    t.start();
  }
}
```

- 在方法内部创建线程

```java
class ThreadMethod {
  private Thread t;
  private String name;
  public ThreadMethod(String name) { this.name = name; }
  public void runTask() {
    if(t == null) {
      t = new Thread(name) {
        public void run() {
          ......
        }
      };
      t.start();
    }
  }
}
```

#### 捕获异常
> 线程的本质特征使得我们態捕获从线程中逃逸的异常，而是直接向外传播到控制台，就算用try-catch语句也没有作用。因而要捕获异常，需要修改Executor产生线程的方式，为每个新创建的Thread附着一个Thread.UnCaughtExceptionHandler()，Thread.UnCaughtExceptionHandler.uncaughtException()会在线程因在线程因未捕获异常而临近死亡时被调用

实例：

```java
class ExceptionThread2 implements Runnable {
  public void run() {
    Thread t = Thread.currentThread();
    System.out.println("run() by " + t);
    System.out.println(
      "eh = " + t.getUncaughtExceptionHandler());
    throw new RuntimeException();
  }
}

//创建一个uncaughtExceptionHandler
class MyUncaughtExceptionHandler implements
Thread.UncaughtExceptionHandler {
  public void uncaughtException(Thread t, Throwable e) {
    System.out.println("caught " + e);
  }
}

class HandlerThreadFactory implements ThreadFactory {
  public Thread newThread(Runnable r) {
    System.out.println(this + " creating new Thread");
    Thread t = new Thread(r);
    System.out.println("created " + t);
    //设置uncaughtExceptionHandler
    t.setUncaughtExceptionHandler(
      new MyUncaughtExceptionHandler());
    System.out.println(
      "eh = " + t.getUncaughtExceptionHandler());
    return t;
  }
}

public class CaptureUncaughtException {
  public static void main(String[] args) {
    ExecutorService exec = Executors.newCachedThreadPool(
      new HandlerThreadFactory());
    exec.execute(new ExceptionThread2());
  }
} 
```

以上代码将输出：
  HandlerThreadFactory@de6ced creating new Thread
  created Thread[Thread-0,5,main]
  eh = MyUncaughtExceptionHandler@1fb8ee3
  run() by Thread[Thread-0,5,main]
  eh = MyUncaughtExceptionHandler@1fb8ee3
  caught java.lang.RuntimeException

#### 线程安全性
> 当多个线程访问某个类时，这个类始终都能表现出正确的行为，那么称这个类是线程安全的。

- **无状态对象**: 无状态对象一定是线程安全的
- **竞态条件**： 由于不恰当的执行时序而出现不正确的结果的情况，大多数竞态条件的本质是———基于一种可能失效的观察结果做出判断或执行某个计算，比如“先检查后执行”操作
- **复合操作**： 包含了一组必须以原子方式执行的操作以确保线程安全性
- **内置锁**：每个Java对象都可以用作一个实现同步的锁，这些锁称为内置锁
  i.同步代码块分为两部分：一个作为锁的对象的引用，一个作为这个锁保护的代码块
  ii. 对象锁：用来控制实例方法之间的同步，包括用synchronized修饰的方法和代码块均是对象锁
  iii. 类锁： 即以Class对象作为锁，用来控制静态方法之间的同步，包括静态的synchronized方法
- **重入**： 某个线程试图获得一个已经由它自己持有的锁，那么这个请求会成功

----

### 共享受限资源
#### 解决共享资源竞争
1.使用synchronized关键字
2.使用显式的Lock对象
示例：

```java
public class MutexEvenGenerator extends IntGenerator {
  private int currentEvenValue = 0;
  private Lock lock = new ReentrantLock();
  public int next() {
    lock.lock();
    try {
      ++currentEvenValue;
      Thread.yield(); 
      ++currentEvenValue;
      return currentEvenValue;
    } finally {
      lock.unlock();
    }
  }
  public static void main(String[] args) {
    EvenChecker.test(new MutexEvenGenerator());
  }
}
```

- 对lock()的调用必须放置在finally语句中带有unlock()的try-finally语句中
- return语句必须出现在try子句中，以确保unlock()不会过早发生
- 这里finally子句也可以同时放置其他处理，以维护系统在正确的状态，避免抛出异常
- `lock.trylock()`,ReentrantLock允许你尝试着获取但最终未获取锁，如果其他线程已经获取了这个锁，那可以离开去执行其他的任务

#### 线程本地存储
> 一种自动化机制，可以为使用相同变量的每个不同的线程都创建不同的存储，有5个线程都要使用变量x所表示的对象，那线程本地存储就会生成5个用于x的不同的存储

----

### 线程之间的协作
#### wait(),notify(),notifyAll()
> wait()使你将当前任务挂起，可以等待某个条件发生，而改变这个条件超出了当前方法的控制能力，通常这种条件由另一个任务来改变

- 调用wait()时，对象上的锁被释放，该对象上的其他synchronized方法可以在wait()期间被调用；
- 只有在notify()或notifyAll()发生时，调用wait()的任务才会被唤醒；
- wait()，notify()，notifyAll()是Object()的一部分，所以可以把wait()放在任何同步控制方法中，不需要考虑是否继承Thread或实现Runnable;
- 只能在同步控制方法或同步控制块里调用wait()，notify()，notifyAll()；
- 为了使任务从wait()中唤醒，必须首先重新获得当它进入wait()时释放的锁；
- 有两种形式的wait()：
  i. 接受毫秒数作为参数，可以通过notify()，notifyAll()，或者时间到期后恢复
  ii. 不接受任何参数，通过notify()，notifyAll()恢复

- 使用notify()，众多等待同一个锁的任务中只有一个会被唤醒，必须保证被唤醒的任务是恰当的任务，而且所有任务必须等待相同的条件
- notifyAll()将唤醒所有等待这个锁的任务

----

### 新类库中的构件
#### CountDownLatch
> **适用场景**：用来同步一个或多个任务，强制他们等待由其他任务执行的一组操作完成

- CountDownLatch对象设置一个初始计数值，任何在这个对象上调用wait()方法都将阻塞，知道这个计数值到达0；
- 其他任务结束时，可调用对象上的countDown()减小计数值
- 计数值不能被重置（需要重置计数值可以用CyclicBarrier）

#### CyclicBarrier
> **适用场景**： 希望创建一组任务，他们并行地执行工作，然后在下一个步骤之前等待，直至所有任务都完成
与CountDownLatch的区别
- 可以重用
- 可以向CyclicBarrier提供一个Runnable，当计数值到达0时自动执行

#### DelayQueue
> 一个无边界的BlockingQueue，用于放置实现了Delayed接口的对象，队列是有序的，对头对象的延迟到期的时间最长，对象只能在到期时才能取走

#### PriorityBlockingQueue
> 基础的优先级队列，具有可阻塞的读取操作

#### ScheduleExecutor
> 解决在预定时间运行的任务，使用schedule()（运行一次）或scheduleThreadFixedRate()（每隔规定的时间重复执行任务），可以将Runnable对象设置为将来的某个时刻执行

#### Semaphore
> 计数信号量，允许n个任务同时访问这个资源

#### Exchanger
> 应用于一个任务在创建对象，这些对象的生产代价很高昂，而另一个任务在消费这些对象，通过使用Exchanger，可以有更多的对象在创建的同时被消费



