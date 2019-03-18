---
title: Java并发之对象的组合
date: 2019-02-25 21:13:36
categories: "Java并发"
tags:
    - Java并发
copyright:
---
### 一、设计线程安全的类
- 设计线程安全类的过程中,需要包含以下三个基本要素:
1. 找出构成对象状态的所有变量
2. 找出约束状态变量的不变性条件
3. 建立对象状态的并发访问管理策略

- 同步策略定义了如何在不违背对象不变性条件或后验条件的情况下对其状态的访问操作进行协同.同步策略规定了如何将不可变性,线程封闭与加锁机制等结合起来以维护线程的安全性,并且还规定了哪些变量由哪些锁来保护

#### 收集同步需求
- 要确保类的线程安全性,就需要确保它的不变性条件不会在并发访问的情况下被破坏,这就需要对其状态进行推断
- 如果在一个不变性条件中包含多个变量， 那么在执行任何访问相关变量的操作时， 都必须持有保护这些变量的锁。
- 如果不了解对象的不变性条件与后验条件，那么就不能确保线程安全性。要满足在状态变量的有效值或状态转换上的各种约束条件，就需要借助于原子性与封装性。

#### 依赖状态的操作
- 什么是依赖状态的操作：
    如果在某个操作中包含有基千状态的先验条件，那么这个操作就称为依赖状态的操作。
- 在单线程程序中，如果某个操作无法满足先验条件，那么就只能失败。 
- 但在并发程序中， 先验条件可能会由于其他线程执行的操作而变成真。在并发程序中要一直等到先验条件为真，然后再执行该操作。

----

### 二、实例封闭
- 封装简化了线程安全类的实现过程，它提供了一种实例封闭机制(Instance Confinement), 通常也简称为 “封闭" 。当一 个对象被封装到另一 个对象中时，能够访问被封装对象的所有代码路径都是已知的。与对象可以由整个程序访问的情况相比，更易于对代码进行分析。通过将封闭机制与合适的加锁策略结合起来， 可以确保以线程安全的方式来使用非线程安全的对象。
- 将数据封装在对象内部，可以将数据的访问限制在对象的方法上，从而更容易确保线程访问数据时总能持有正确的锁。
- 被封闭对象一定不能超出它们既定的作用域。对象可以封闭在类的一 个实例（例如作为类 的一个私有成员）中， 或者封闭在某个作用域内（例如作为一个局部变量），再或者封闭在线程内（例如在某个线程中 将对象从一 个方法传递到另一 个方法， 而不是在多个线程之间共享该对象 ）。
#### Java监视器模式
- 遵循Java监视器模式的对象会把对象的所有可变状态都封装起来，并由对象自己的内置锁来保护。（在许多类中都使用了Java监视器模式， 例如Vector和Hashtable）
- 使用私有的锁对象而不是对象的内置锁（或任何其他可通过公有方式访问的锁）， 有许多优点：
 1. 私有的锁对象可以将对象锁封装起来，使客户代码无法得到锁， 但客户代码可以通过公有方法来访问锁， 以便（正确或者不正确地）参与到它的同步策略中。
 2. 如果客户代码错误地获得了另一个对象的锁， 那么可能会产生活跃性问题。此外， 要想验证某个公有访问的锁在程序中否被正确地使用， 则需要检查整个程序， 而不是单个的类。

----

 ### 三、线程安全性的委托
在某些 情况下， 通过多个线程安全类组合而成的类是线程安全的， 而在某些情况下， 这仅仅是一个好的开端。
#### 将线程安全委托给单个线程安全的状态变量
以下示例将线程安全性委托给AtomicLong：
```java
public class CountingFactorizer{
    private final AtomicLong count = new AtomicLong(0);

    public long getCount() { return count.get(); }

    public void add(ServletRequest req, ServletResponse resp) {
        count.incrementAndGet();
    }
}
```

#### 将线程安全性委托给多个状态变量
- 可以将线程安全性委托给多个状态变量,只要这些变量是彼此独立的,即组合而成的类并不会在其包含的多个状态变量上增加任何不变性条件：

```java
public class VisualComponent {
    private final List<KeyListener> keyListeners
            = new CopyOnWriteArrayList<KeyListener>();
    private final List<MouseListener> mouseListeners
            = new CopyOnWriteArrayList<MouseListener>();

    public void addKeyListener(KeyListener listener) {
        keyListeners.add(listener);
    }

    public void addMouseListener(MouseListener listener) {
        mouseListeners.add(listener);
    }

    public void removeKeyListener(KeyListener listener) {
        keyListeners.remove(listener);
    }

    public void removeMouseListener(MouseListener listener) {
        mouseListeners.remove(listener);
    }
}
```
> VisualComponent为鼠标和键盘备有一个注册监听器列表，因此当某个事件发生时，就会调用相应的监听器，然而鼠标事件监听器和键盘事件监听器之间不存在任何关联，二者是彼此独立的而因此VisualComponent可以将其线程安全性委托给这两个线程安全的监听器列表。

- 反例：当委托失效时

```java
public class NumberRange {
    // INVARIANT: lower <= upper
    private final AtomicInteger lower = new AtomicInteger(0);
    private final AtomicInteger upper = new AtomicInteger(0);

    public void setLower(int i) {
        // Warning -- unsafe check-then-act
        if (i > upper.get())
            throw new IllegalArgumentException("can't set lower to " + i + " > upper");
        lower.set(i);
    }

    public void setUpper(int i) {
        // Warning -- unsafe check-then-act
        if (i < lower.get())
            throw new IllegalArgumentException("can't set upper to " + i + " < lower");
        upper.set(i);
    }

    public boolean isInRange(int i) {
        return (i >= lower.get() && i <= upper.get());
    }
}
```
> NumberRange使用两个AtomicInteger来管理状态，并且两个状态变量之间存在一个不变形条件：即第一个数值要小于或等于第二个数值。因而NumberRange不是线程安全的。

- 如果某个类含有符合操作，例如NumberRange,那么仅靠委托并不足以实现线程安全性。这种情况下,这个类必须提供自己的加锁机制以保证这些复合操作都是原子操作,除非整个复合操作都可以委托给状态变量
- 如果一个类是由多个独立且线程安全的状态变量组成，并且在所有的操作中都不包含无效状态转换，那么可以将线程安全性委托给底层的状态变量。
- 如果一个状态变量是线程安全的，并且没有任何不变性条件来约束它的值，在变量的操作上也不存在任何不允许的状态转换，那么就可以安全地发布这个变量。
----

### 四、在现有的线程安全类中添加功能
#### 将新方法添加到类中
要添加一个新的原子操作， 最安全的方法是修改原始的类，如果直接将新方法添加到类中， 那么意味着实现同步策略的所有代码仍然处于一个源代码文件中， 从而更容易理解与维护。**但这通常无法做到。**
#### 扩展这个类
```java
public class BetterVector <E> extends Vector<E> {
    static final long serialVersionUID = -3963416950630760754L;

    public synchronized boolean putIfAbsent(E x) {
        boolean absent = !contains(x);
        if (absent)
            add(x);
        return absent;
    }
}
```
> “扩展” 方法比直接将代码添加到类中更加脆弱， 因为现在的同步策略实现被分布到多个单独维护的源代码文件中。

#### 使用客户端加锁
扩展类的功能， 但并不是扩展类本身， 而是将扩展代码放人一个“ 辅助类” 中。
```java
class BadListHelper <E> {
    public List<E> list = Collections.synchronizedList(new ArrayList<E>());

    public synchronized boolean putIfAbsent(E x) {
        boolean absent = !list.contains(x);
        if (absent)
            list.add(x);
        return absent;
    }
}
```
> 这种方法是线程不安全的，问题在于在错误的锁上进行了同步。无论List使用哪一个锁来保护它的状态，可以确定的是，这个锁并不是ListHelper上的锁。要使用客户端加锁， 你必须知道对象X使用的是哪一个锁。

以下的实现才是正确的：
```java
class GoodListHelper <E> {
    public List<E> list = Collections.synchronizedList(new ArrayList<E>());

    public boolean putIfAbsent(E x) {
        synchronized (list) {
            boolean absent = !list.contains(x);
            if (absent)
                list.add(x);
            return absent;
        }
    }
}
```

> 通过添加一个原子操作来扩展类是脆弱的， 因为它将类的加锁代码分布到多个类中。然而，客户端加锁却更加脆弱， 因为它将类 C的加锁代码放到与C完全无关的其他类中。

#### 组合
```java
public class ImprovedList<T> implements List<T> {
    private final List<T> list;

    public ImprovedList(List<T> list) { this.list = list; }

    public synchronized boolean putIfAbsent(T x) {
        boolean contains = list.contains(x);
        if (contains)
            list.add(x);
        return !contains;
    }

    public synchronized void clear() {list.clear();}
    // ...按照类似的方式委托给List的其他方法
}
```
> ImprovedList通过自身的内置锁增加了一层额外的加锁。它并不关心底层的List是否是线程安全的， 即使List不是线程安全的或者修改了它的加锁实现，ImprovedList也会提供一致的加锁机制来实现线程安全性。
