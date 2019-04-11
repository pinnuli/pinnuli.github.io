---
title: 深入ArrayList看fast-fail机制
date: 2019-04-10 21:23:05
categories: "JavaSE"
tags:
    - JavaSE
copyright:
---

![](/images/JavaSE_fail-fast.png)
### fail-fast机制简介
#### 什么是fail-fast
fail-fast 机制是java集合(Collection)中的一种错误机制。它只能被用来检测错误，因为JDK并不保证fail-fast机制一定会发生。当多个线程对同一个集合的内容进行操作时，就可能会产生fail-fast事件。例如：当某一个线程A通过iterator去遍历某集合的过程中，若该集合的内容被其他线程所改变了；那么线程A访问集合时，就会抛出ConcurrentModificationException异常，产生fail-fast事件。
> 这种“ 及时失败” 的迭代器井不是一种完备的处理机制，而只是“ 善意地” 捕获并发错误，因此只能作为并发问题的预警指示器。

#### fail-fast示例
```java
public class FailFastTest {

    static final List<Integer> list = new ArrayList<>();

    public static void main(String[] args) {
        ExecutorService exec = Executors.newCachedThreadPool();
        for (int i = 0; i < 5; i++) {
            exec.execute(() -> {
                add((int)(Math.random() * 10));
                print();
            });
        }
    }

    private static void add(int number) {
        list.add(number);
    }

    private static void print() {
        Iterator iterator = list.iterator();
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
    }
}

```

> 以上代码会抛出`java.util.ConcurrentModificationException`异常

#### fail-fast的解决办法
使用java.util.concurrent.*下的工具类


### 深入ArrayList源码看fast-fail的原理
先放上
首先看下ArrayList中的内部类Itr的域
```java
private class Itr implements Iterator<E> {
    int cursor;                         //1
    int lastRet = -1;                   //2
    int expectedModCount = modCount;    //3
}
```
> 1. 表示迭代器下一个元素的索引; 
> 2. 表示迭代器上一个元素的索引;
> 3. 在创建一个迭代器时，将当前ArrayList的修改次数赋值给expectedModCount保存。

在上面的示例中，我们可以看到一般的迭代过程是
```java
Iterator iterator = list.iterator();
    while (iterator.hasNext()) {
        System.out.println(iterator.next());
    }
```
分别看下`iterator()`、`hasNext()`、`next()`三个方法：

- iterator()：没有做任何处理，不过构造时三个域会进行初始化

```java
Itr() {}
```

- hasNext()：判断下一个元素索引是否等于ArrayList的大小，等于说明没有元素了

```java
public boolean hasNext() {
    return cursor != size;
}
```

- 接下来重点看next()方法

```java
public E next() {
    checkForComodification();                               //1
    int i = cursor;                                         //2
    if (i >= size)
        throw new NoSuchElementException();
    Object[] elementData = ArrayList.this.elementData;
    if (i >= elementData.length)
        throw new ConcurrentModificationException();
    cursor = i + 1;
    return (E) elementData[lastRet = i];
}
```

在获取下一个元素之前，先调用checkForComodification()进行了检查，检查当前集合的修改次数是不是跟之前保存的相同，如果相同则表示没有被其他线程修改， 
```java
final void checkForComodification() {
    if (modCount != expectedModCount)
        throw new ConcurrentModificationException();
}
```


> modCount：modCount 是 AbstractList 的属性值：`protected transient int modCount = 0; 他是一个修改次数计数器，实例化一个集合之后，每次修改（源码的注释成为结构性修改），比如set，add，clear等，计数器都会加1。

**这里其实就是fail-fast机制的实现原理了，将修改计数器的变化与容器关联起来：首先在构造迭代器的时候，将当前的修改计数器的值保存，之后进行遍历的时候，每访问一个数据，都要检查当前集合的修改次数是否合法，如果有其他线程修改了集合，那么modCount就会被修改，当前修改计数器的值与之前保存的值（即期望值）不同，那么将抛出ConcurrentModificationException。**


### fail-fast与fail-safe的区别
待补充
