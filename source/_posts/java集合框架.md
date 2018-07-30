---
title: java集合框架
date: 2018-07-11 07:02:56
categories: "javase笔记" 
tags:
	- JavaSE
---


### 一、迭代器
iterator是一个或者实现了Iterator,或者实现ListIterator接口的对象，可以通过循环输出类集的内容，从而获得或删除元素，

- next()方法
	- 逐个访问集合中的每个元素，经常需要与hasNext()方法搭配使用；
	- java迭代器可以认为位于两个元素之家，当调用next()时，迭代器越过下一个元素，并返回刚刚越过的那个元素的引用
- 用"for each"循环遍历类集的内容，
- remove()方法删除上次调用next()时返回的元素，因而调用remove之前没有调用next是不合法的，删除的元素依赖于迭代器的状态
- 对于实现List的类集，可以使用ListIterator，可以双向访问类集，如果在调用previous之后调用remove，则会将迭代器右边的元素删掉

----
### 二、List
有序集合，使用基于零的下标，可以用迭代器访问（顺序访问）或者用一个整数索引访问（随机访问）,有两个主要的实现类

**ArrayList**
- 基于数组，能够动态到地增加或减小其大小，当也可以调用ensureCapacity方法来进行人工地增加ArrayList的容量，从而避免再分配的消耗时间，前提是事先知道需要存储很多内容，然后往往是不知道

- 适用于需要进行随机访问时
	
**LinkedList**
- 基于双向链表，
- 对于有序集合，add方法只是添加到链表的尾部，当需要将元素添加到链表中间时，可以用迭代器的add，只有对自然有序的集合使用迭代器添加元素才有实际意义，但是这里的add不返回任何值
- 适用于顺序访问

----
### 三、Set
等同于Collection,不允许增加重复的元素，需要定义equals方法来确保元素唯一

**HashSet**
- 基于散列表的集，无序
- 存入的元素必须定义有hashCode()，以得到散列码
- 可以快速地查找，对于大的集合，add()、contains()、remove()、size()等方法的运行时间保持不变
- 可以设置装填因子来实现再散列
- 适用于不关心访问顺序，需要查找大容量容器时

**TreeSet**
- 基于红黑树排序，有序
- 存入的元素必须实现Comparable接口，或者构造集必须提供一个Comparator
- 每次添加元素时元素都会被放置到正确的位置，因而迭代器总是以排好序的顺序访问
- 适用于存储大量需要进行快速检索的排序信息的情况
	
**BitSet**
- 用于存放一个位序列

----
### 四、Queue
**双端队列**
可以同时在头部或者尾部添加或删除元素，有两个实现
- inkedList
- ArrayList

**PriorityQueue**
- 基于堆，堆是可以自我调整的二叉树，对树执行add和remove时，可以让最小的元素移动到根，不必对元素排序
- 按照任意顺序插入，却可以按照顺序进行检索
- 和TreeSet一样，存入的元素必须实现Comparable接口，或者构造器必须提供一个Comparator

----
### 五、Map
对键进行散列或排序，键必须唯一，同一个键多次调用put时，后面的会取代前面的，put返回这个键存储的上一个值。
**键不存在时**
get方法返回null，可以用getOrDefault返回默认值，test.get(id,0)
**更新映射项**
正常情况可以得到与一个键关联的值，替代原来的值，但是在需要持续更新时，例如:使用一个映射统计一个单词在文件中的频数，`counts.put(word,counts.get(word)+1)`,需要解决一个问题，就是键第一次出现时，这时可以有三种方法
- `counts.put(word,counts.getOrDefault(word,0) + 1)`
- 先调用putIfAbsent，只有原先键存在时才会放一个值,`counts.putIfAnsent(word,0)`		
	`counts.put(word,counts.get(word)+1)`			
- `counts.merge(word,1,Interger::sum)`(推荐）,当word关联的值为空（即键值对不存在），将word与1关联；不为空时，将sum应用于word和1，sum返回的结果与word关联

**映射视图**
集合框架不认为映射本身是一种集合，不过可以通过调用Map一些方法得到映射的视图
- 键集：`Set<K> keySet()`，
- 值集：`Collection<V> values()`，
- 键/值对集：`Set<Map.Entry<K,V>> entrySet()`
- 这些集合不能添加元素，但可以删除这些集合中的远思，键和相关联的的值也将从映射中删除

**散列**
- 散列码，由对象的实例域产生的一个整数，由hashCOde()产生
- 散列表，用链表数组实现，每个列表称为桶
- 散列实现，Map中用数组保存键，查找表中对象的位置时，先算出散列码，然后与桶的总数取余，结果为保存此元素的桶的索引，即数组的下标，查询时通过equals()方法对List中的值进行查询
> 新版jdk使用红黑树和List（数据多用树，少时用List)

- 散列冲突，桶被占满时，可以设置装填因子（0~1），对散列表满时进行再散列，即创建桶数更多的表

**HashMap**
- 比较快，适用于不需要按照排列顺序访问键时

**TreeMap**
- 按排序顺序存储键值对，允许快速检索，保证了元素按关键字升序排序

**WeakHashMap**
- 使用弱引用（weak reference)保存key，WeakReference对象将引用保存到另外一个对象中，就是散列键
- 如果垃圾回收器发现某个特定的对象已经没有人引用，就将其回收，而如果某个对象只由WeakReference引用，垃圾回收器仍然回收它，并将这个对象的弱引用放入队列，WeakHashMap将周期性地检查队列，一边找出新添加的弱引用，并将删除对应的条目
- 适用于需要缓存时

**LinkedHashMap**
- 链接散列集与映射，用访问顺序，对映射条目进行迭代，每次调用get或put时，收到影响的条目从当前位置删除，并放到条目链表的尾部，不过条目仍然在原来散列码对应的桶中，只是改变了条目在链表中的位置
- 用于实现高速缓存的“最近最少使用”原则
 
----
六、视图与包装器
**轻量级集合包装器**
例如Arrays类的asList返回一个包装了普通java数组的List包装器，可以将数组传递给一个期望得到列表或集合参数的方法
```java
	Card[] cardDeck = new Card[52];
	List<Card> cardList = Arrays.asList(cardDeck);
```
> 返回的是一个视图对象，可以调用底层数组的get和set，但不可改变数组的大小,例如Collections类的一些使用方法，`nCopied(100,"KKK")`,`single(anObject)`等，返回一个不可修改的试图对象

**子范围**
- 相当于返回几个的一个“子集合”，例如返回列表staff的第10-19个元素，可以用`List group2 = staff.subList(10,20);`,类似于String类中获取子串
- 可以将任何操作应用于子范围，如`group2.clear()``,元素会从staff中清除
- 对于有序集或者映射，可以使用排序顺序建立子范围，如SortedSet声明的三个方法，返回大于等于from小于to的所有元素的子集，
> SortedSet<E> subSet(E from, E to)
> SortedSet<E> headSet(E to)
> SortedSet<E> tailSet(E from)

**不可修改的视图**
- 只能对现有集合增加了一个运行时的检查，试图修改会抛出异常，不过仍然可以通过原始引用修改
- 访问器方法从原始集合对象中获取值
- 视图只是包装了接口而不是实际的集合对象，因而只能访问接口中定义的方法，例如：
> `unmodifiableCollection`方法将返回一个集合，但他的`equals()`方法不调用底层集合的，而是调用它继承了`Object类的equals()`方法，这个方法只是检测两个对象是否是同一个对象，视图就是以这种方式运行的

**同步视图**
- 实现多线程访问，确保集合的线程安全，例如`Collections.synchronizedMap`

**受查视图**
- 用来对泛型类型进行检测，例如`Collections.checkedList`

> 参阅：
  [java核心技术 卷I：基础知识](http://product.dangdang.com/24035306.html)