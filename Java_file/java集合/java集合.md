# java集合

## Java集合图

Collection的集合图解

![image-20210805213030668](D:\1书本笔记\java实战项目\image-20210805213030668.png)

Map的集合图解

![image-20210807110344501](D:\1书本笔记\java实战项目\image-20210807110344501.png)

**其中大多数的集合都不是线程安全的**：

在实际编程中，会经常使用到 JDK 中 Collection 集合框架中的各种容器类如实现 List,Map,Queue 接口的容器类，但是这些容器类基本上不是线程安全的，除了使用 Collections 可以将其转换为线程安全的容器，Doug Lea 大师为我们都准备了对应的线程安全的容器，如实现 List 接口的 **CopyOnWriteArrayList**，实现 Map 接口的 **ConcurrentHashMap**，实现 Queue 接口的 **ConcurrentLinkedQueue**。

## 1 Map

### 1 HashMap

### 1 HashMap的介绍

​		HashMap 主要用来存放键值对，它基于哈希表的 Map 接口实现，是常用的 Java 集合之一，是非线程安全的。JDK1.8 之前 HashMap 由 **数组+链表** 组成的，数组是 HashMap 的主体，链表则是主要为了解决哈希冲突而存在的（“拉链法”解决冲突）。 JDK1.8 以后的 `HashMap` **在解决哈希冲突时有了较大的变化**，当链表长度大于阈值（默认为 8）（将链表转换成红黑树前会判断，如果当前数组的长度小于 64，那么会选择先进行数组扩容，而不是转换为红黑树）时，将链表转化为红黑树，以减少搜索时间。

**负载因子**是表示Hsah表中元素的填满的程度。

1. **如何有效地根据key值查找value ？**

为了利用索引来查找, 我们需要建立一个 `key -> index` 的映射关系, 这样每次我们要查找一个 key时, 首先根据映射关系, 计算出对应的数组下标, 然后根据数组下标, 直接找到对应的key-value对象, 这样基本能以o(1)的时间复杂度得到结果.

这里, 将key映射成index的方法称为hash算法, 我们希望它能将 key均匀的分布到数组中.

这里插一句,**使用Hash算法同样补足了数组插入和删除性能差的短板**, 我们知道, 数组之所以插入删除性能差是因为它是顺序存储的, 在一个位置插入节点或者删除节点需要一个个移动它的后续节点来腾出位或者覆盖位置.

使用hash算法后, 数组不再按顺序存储, 插入删除操作只需要关注一个存储桶即可, 而不需要额外的操作.

**2 . 如何解决hash冲突**

解决hash冲突的方法有很多, 在HashMap中我们选择**链地址法**, 即在产生冲突的存储桶中改为单链表存储.

![image-20210807094929629](D:\1书本笔记\java实战项目\image-20210807094929629.png)

**3. 链表长度过长怎么办？**

我们知道, 链表查找只能通过顺序查找来实现, 因此, 时间复杂度为o(n), 如果很不巧, 我们的key值被Hash算法映射到一个存储桶上, 将会导致存储桶上的链表长度越来越长, 此时, 数组查找退化成链表查找, 则时间复杂度由原来的o(1) 退化成 o(n).

为了解决这一问题, 在java8中, 当链表长度超过 8 之后, 将会自动将链表转换成红黑树, 以实现 o(log n) 的时间复杂度, 从而提升查找性能.

![image-20210807095245228](D:\1书本笔记\java实战项目\image-20210807095245228.png)



### 2 HashMap的几个构造函数

**构造函数**

HashMap 共有四个构造函数

```java
public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>, Cloneable, Serializable {

    // 默认初始大小 16
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

    // 默认负载因子 0.75
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
     
    final float loadFactor;
    
    /**
     * The next size value at which to resize (capacity * load factor).
     *
     * @serial
     */
    // (The javadoc description is true upon serialization.
    // Additionally, if the table array has not been allocated, this
    // field holds the initial array capacity, or zero signifying
    // DEFAULT_INITIAL_CAPACITY.)
    int threshold;
    
    transient Node<K,V>[] table;
     
    // 没有指定时, 使用默认值
    // 即默认初始大小16, 默认负载因子 0.75
    public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
    }
    
    // 指定初始大小, 但使用默认负载因子
    // 注意这里其实是调用了另一个构造函数
    public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }
    
    // 指定初始大小和负载因子
    public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        this.loadFactor = loadFactor;
        this.threshold = tableSizeFor(initialCapacity);
    }
    
    // 利用已经存在的map创建HashMap
    public HashMap(Map<? extends K, ? extends V> m) {
        this.loadFactor = DEFAULT_LOAD_FACTOR;
        putMapEntries(m, false);
    }
        
}
```

不知道大家发现了没有, 即使我们在构造函数中指定了`initialCapacity`, 这个值也只被用来计算 `threshold`

```ini
this.threshold = tableSizeFor(initialCapacity);
```

而 `threshold` 这个值在初始化table时, 就代表了数组的初始大小, 这个我们到后面用到的时候讲.

我们先来看看`tableSizeFor`函数干了什么事:

```java
/**
 * Returns a power of two size for the given target capacity.
 */
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

tableSizeFor这个方法用于找到大于等于initialCapacity的最小的2的幂, 这个算法还是很精妙的, 这里我稍微解释一下:
我们知道, 当一个32位整数不为0时, 32bit中至少有一个位置为1, 上面5个移位操作的目的在于, 将 *从最高位的`1`开始, 一直到最低位的所有bit* 全部设为1, 最后再加1(注意, 一开始是先`cap-1`的), 则得到的数就是大于等于initialCapacity的最小的2的幂. 读者自己找一个数算一下就明白了, 也可以参照[这一篇博客](https://link.segmentfault.com/?url=https%3A%2F%2Fblog.csdn.net%2Ffan2012huan%2Farticle%2Fdetails%2F51097331).

最后我们来看最后一个构造函数, 它调用了 `putMapEntries` 方法:

```reasonml
final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
    int s = m.size();
    if (s > 0) {
        if (table == null) { // pre-size
            float ft = ((float)s / loadFactor) + 1.0F;
            int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                     (int)ft : MAXIMUM_CAPACITY);
            if (t > threshold)
                threshold = tableSizeFor(t);
        }
        else if (s > threshold)
            resize();
        for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
            K key = e.getKey();
            V value = e.getValue();
            putVal(hash(key), key, value, false, evict);
        }
    }
}
```

我们知道, 当使用构造函数`HashMap(Map<? extends K, ? extends V> m)` 时, 我们并没有为 `table` 赋值, 所以, `table`值一定为`null`, 我们先根据传入Map的大小计算 `threshold` 值, 然后判断需不需要扩容, 最后调用 `putVal`方法将传入的Map插入table中.

`resize` 和 `putVal` 方法我们以后再细讲.

**总结**

通过上面对四个构造函数的分析我们发现, 除了最后一个构造函数, 其他三个函数:

```java
HashMap()
HashMap(int initialCapacity)
HashMap(int initialCapacity, float loadFactor)
```

的调用中, 最多只牵涉到HashMap的两个Field `loadFactor`, `threshold`, 而并不牵涉到 `table` 变量.

这说明HashMap中, `table`**的初始化或者使用不是在构造函数中进行的**, 而是在实际用到的时候, 事实上, 它是在HashMap**扩容的时候实现的**, 即`resize`函数。



### 3 HashMap的Hash算法的理解

**hash算法**

**为了利用数组索引进行快速查找, 我们需要先将 `key`值映射成数组下标.** 因为数组的下标是有限的集合, **所以我们可以先通过hash算法将`key`映射成整数**, 再将整数映射成有限的数组下标:

> Object -> int -> index

对于 `Object -> int` 部分, 使用的就是hash function, 而对于 `int -> index` 部分, 我们可以简单的使用对数组大小取模来实现.

下面我们就来看看HashMap使用了什么hash算法.

首先我们来看维基百科对于hash function的定义:

> 散列函数（英语：Hash function）又称散列算法、哈希函数，是一种从任何一种数据中创建小的数字“指纹”的方法。散列函数把消息或数据压缩成摘要，使得数据量变小，将数据的格式固定下来。该函数将数据打乱混合，重新创建一个叫做散列值（hash values，hash codes，hash sums，或hashes）的指纹。

在java中, hash函数是一个native方法, 这个定义在Object类中, 所以所有的对象都会继承.

```aspectj
public native int hashCode();
```

因为这是一个本地方法, 所以我们无法看到它的具体实现, 但是从函数签名上可以看出, 该方法将任意对象映射成一个整型值.调用该方法, 我们就完成了 `Object -> int`的映射

所以将 `key`映射成`index` 的方式可以是

```sas
key.hashCode() % table.length
```

那么HashMap是这样做的吗? 事实上, **HashMap定义了自己的散列方法**:

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

我们知道, int类型是32位的, `h ^ h >>> 16` 其实就是将hashCode的高16位和低16位进行异或, 这充分利用了高半位和低半位的信息, 对低位进行了`扰动`, 目的是为了使该hashCode映射成数组下标时可以更均匀, 详细的解释可以参考[这里](https://link.segmentfault.com/?url=https%3A%2F%2Fwww.zhihu.com%2Fquestion%2F20733617%2Fanswer%2F111577937).

链接：https://www.zhihu.com/question/20733617/answer/111577937

![image-20210807101737528](D:\1书本笔记\java实战项目\image-20210807101737528.png)

![image-20210807101757655](D:\1书本笔记\java实战项目\image-20210807101757655.png)

![image-20210807101814777](D:\1书本笔记\java实战项目\image-20210807101814777.png)



![image-20210807101823086](D:\1书本笔记\java实战项目\image-20210807101823086.png)

![image-20210807101838536](D:\1书本笔记\java实战项目\image-20210807101838536.png)



### 4  HashMap的resize()流程

参考链接：

​	https://segmentfault.com/a/1190000015812438

resize( ) :

 前提知识：

1. 首先明白每个Node节点是如何通过hash散列函数映射到Table[ ] 数组里面的，其实本质是通过一个简单的取余的算法 node.hash % newCap ,但是这样做的话，效率很低，所以源码的大佬们采用了位运算来提高效率即Table[ ] 数组的下标 i=  e.hash & (newCap - 1) (这个方法等价于前面的取余运算的方法了)，有一点值得说一下，就是一般HashMap的容量为2^m，所以一般Table[ ] 的下标值e.hash & (newCap - 1)，**本质上其实是取 hash的低位m位**。
2. 扩容后的数组可以通过 e.hash & oldCap 的方法来判断，当前的Node节点在扩容后的位置在哪里，如果e.hash & oldCap==0，那么当前节点Node保持原Table数组下标的 j 不变，但是如果e.hash & oldCap==1，说明需要扩容了，就需要将当前节点Node,放在扩容之后的 j+oldCap 的下标上面啦，具体的参考下面这张图。





![image-20210807000238785](D:\1书本笔记\java实战项目\image-20210807000238785.png)



具体的步骤方法的步骤；

1. 如果原先的Table[] 数组里面已经存在了Node节点，那么每调用一次resize( ) 方法来扩展HashMap的时候，**都会将原来的容量和阈值扩大为原来的2倍**
2.  Table[ ] 扩容的时候，各个节点的转换情况，具体进行节点操作的时候，假设现有两个链表，一个是lo链表，一个是hi链表，扩展的时候，按照上面的规律分别将Node节点移动到lo链表和 hi 链表上面。最后一步，**再将lo链表放在原始的Table[] 数组的下标 j 位置处**，**将 hi 链接放在Table[] 数组的 j+OldCap下标位置处**。如上图中的流程所示。

resize( ) 源码：

```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    
    // 原table中已经有值
    if (oldCap > 0) {
    
        // 已经超过最大限制, 不再扩容, 直接返回
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        
        // 注意, 这里扩容是变成原来的两倍
        // 但是有一个条件: `oldCap >= DEFAULT_INITIAL_CAPACITY`
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    
    // 在构造函数一节中我们知道
    // 如果没有指定initialCapacity, 则不会给threshold赋值, 该值被初始化为0
    // 如果指定了initialCapacity, 该值被初始化成大于initialCapacity的最小的2的次幂
    
    // 这里是指, 如果构造时指定了initialCapacity, 则用threshold作为table的实际大小
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    
    // 如果构造时没有指定initialCapacity, 则用默认值
    else {               // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    
    // 计算指定了initialCapacity情况下的新的 threshold
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    
    
    //从以上操作我们知道, 初始化HashMap时, 
    //如果构造函数没有指定initialCapacity, 则table大小为16
    //如果构造函数指定了initialCapacity, 则table大小为threshold, 即大于指定initialCapacity的最小的2的整数次幂
    
    
    // 从下面开始, 初始化table或者扩容, 实际上都是通过新建一个table来完成的
    @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    
    // 下面这段就是把原来table里面的值全部搬到新的table里面
    if (oldTab != null) {
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                // 这里注意, table中存放的只是Node的引用, 这里将oldTab[j]=null只是清除旧表的引用, 但是真正的node节点还在, 只是现在由e指向它
                oldTab[j] = null;
                
                // 如果该存储桶里面只有一个bin, 就直接将它放到新表的目标位置
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                
                // 如果该存储桶里面存的是红黑树, 则拆分树
                else if (e instanceof TreeNode)
                    //红黑树的部分以后有机会再讲吧
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                
                // 下面这段代码很精妙, 我们单独分一段详细来讲
                else { // preserve order
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

**关于 `(e.hash & oldCap) == 0` `j` 以及 `j+oldCap`**

上面我们已经弄懂了链表拆分的代码, 但是这个拆分条件看上去很奇怪, 这里我们来稍微解释一下:

首先我们要明确三点:

1. oldCap一定是2的整数次幂, 这里假设是2^m
2. newCap是oldCap的两倍, 则会是2^(m+1)
3. hash对数组大小取模`(n - 1) & hash` 其实就是取hash的低`m`位

### 5 HashMap的resize()流程在多线程下面的危害？

​		jdk1.7版本中多线程同时对HashMap扩容时，**会引起链表死循环**，**尽管jdk1.8修复了该问题**，但是同样在jdk1.8版本中多线程操作hashMap时仍然会引起死循环，只是**原因不一样**。

**1 先说一下jdk1.7版本中多线程同时对HashMap扩容时引发的链表死锁的问题：**

![image-20210807111233024](D:\1书本笔记\java实战项目\image-20210807111233024.png)

具体流程：

​		前提条件：当线程1走到next=e.next的时候，线程1被挂起，线程2执行了，此时线程2完成了扩容的过程但是线程1的e和next指针还是指向原来的位置。

本质是因为next指针的移动，最终next指针和e指针都移动到null节点上面去了。



**单线程扩容：**

**假设：**hash算法就是简单的key与length(数组长度)求余。

​     hash表长度为2，如果不扩容， 那么元素key为3,5,7按照计算(key%table.length)的话都应该碰撞到table[1]上

​     

**扩容：**hash表长度会扩容为4

​     重新hash，key=3 会落到table[3]上(3%4=3)， 当前e.next为key(7), 继续while循环

​     重新hash，key=7 会落到table[3]上(7%4=3), 产生碰撞， 这里采用的是头插入法，所以key=7的Entry会排在key=3前面(这里可以具体看while语句中代码)

​     当前e.next为key(5), 继续while循环

​     重新hash，key=5 会落到table[1]上(5%4=1)， 当前e.next为null, 跳出while循环， resize结束

​     

 

如题如图所示：

 

[![Image(3)](https://images2017.cnblogs.com/blog/799093/201709/799093-20170923210150181-847533613.png)](http://images2017.cnblogs.com/blog/799093/201709/799093-20170923210148900-1945296724.png)

 

 

**多线程扩容：**

这里我们先把核心代码搬出来， 方便查看

while(null != e) {

  Entry<K,V> next = e.next; //第一行

  int i = indexFor(e.hash, newCapacity); //第二行

  e.next = newTable[i]; //第三行

  newTable[i] = e; //第四行

  e = next; //第五行

}

去掉了一些冗余的代码， 层次结构更加清晰了。

**第一行：记录odl hash表中e.next**

**第二行：rehash计算出数组的位置(hash表中桶的位置)**

**第三行：e要插入链表的头部， 所以要先将e.next指向new hash表中的第一个元素**

**第四行：将e放入到new hash表的头部**

**第五行： 转移e到下一个节点， 继续循环下去**

核心代码如上所说， 下面就是多线程同时put的情况了， 然后同时进入transfer方法中：

假设这里有两个线程同时执行了`put()`操作，并进入了`transfer()`环节

```
while(null != e) {
    Entry<K,V> next = e.next; //线程1执行到这里被调度挂起了
    e.next = newTable[i];
    newTable[i] = e;
    e = next;
}
```

那么现在的状态为：

![img](http://static.oschina.net/uploads/space/2016/0511/150544_UYcT_2243330.jpg)

从上面的图我们可以看到，因为线程1的 e 指向了 key(3)，而 next 指向了 key(7)，在线程2 rehash 后，就指向了线程2 rehash 后的链表。

然后线程1被唤醒了：

1. 执行`e.next = newTable[i]`，于是 key(3)的 next 指向了线程1的新 Hash 表，因为新 Hash 表为空，所以`e.next = null`，
2. 执行`newTable[i] = e`，所以线程1的新 Hash 表第一个元素指向了线程2新 Hash 表的 key(3)。好了，e 处理完毕。
3. 执行`e = next`，将 e 指向 next，所以新的 e 是 key(7)

然后该执行 key(3)的 next 节点 key(7)了:

1. 现在的 e 节点是 key(7)，首先执行`Entry<K,V> next = e.next`,那么 next 就是 key(3)了
2. 执行`e.next = newTable[i]`，于是key(7) 的 next 就成了 key(3)
3. 执行`newTable[i] = e`，那么线程1的新 Hash 表第一个元素变成了 key(7)
4. 执行`e = next`，将 e 指向 next，所以新的 e 是 key(3)

这时候的状态图为：

![img](http://static.oschina.net/uploads/space/2016/0511/150835_WG1V_2243330.png)

然后又该执行 key(7)的 next 节点 key(3)了：

1. 现在的 e 节点是 key(3)，首先执行`Entry<K,V> next = e.next`,那么 next 就是 null
2. 执行`e.next = newTable[i]`，于是key(3) 的 next 就成了 key(7)
3. 执行`newTable[i] = e`，那么线程1的新 Hash 表第一个元素变成了 key(3)
4. 执行`e = next`，将 e 指向 next，所以新的 e 是 key(7)

这时候的状态如图所示：

![img](http://static.oschina.net/uploads/space/2016/0511/151022_MSj3_2243330.png)

 

很明显，环形链表出现了！！当然，现在还没有事情，因为下一个节点是 null，所以`transfer()`就完成了，等`put()`的其余过程搞定后，HashMap 的底层实现就是线程1的新 Hash 表了

2. **1.8以后的扩容机制，修改了环形链表的BUG但是还是存在bug**

那就是典型的ABA问题了，多个线程之间修改数据，会覆盖掉原来的线程，导致我们得不到我们想要的结果。



### 5 HashMap的put方法

HashMap 实现了Map接口, 因此必须要实现put方法:

```xquery
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
    /*final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) */
}
```

可以看到, put方法是有返回值的, 这里调用了 `putVal` 方法, 这个方法很重要, 我们将通过代码注释的方式逐行说明.

在这之前我们先看该方法的参数:

- hash

由上面的调用可知, 该值为`hash(key)`, 是key的hash值, 关于hash的概念之前已经讲过了, 这里不再赘述.

- key, value

待存储的键值对

- onlyIfAbsent

这个参数用于决定待存储的key已经存在的情况下,要不要用新值覆盖原有的`value`, 如果为`true`, 则保留原有值, `false` 则覆盖原有值, 从上面的调用看, 该值为`false`, 说明当`key`值已经存在时, 会直接覆盖原有值。

- evict

该参数用来区分当前是否是构造模式, 我们在讲解构造函数的时候曾经提到，HashMap的第四个构造函数可以通过已经存在的Map初始化一个HashMap, 如果为 `false`, 说明在构造模式下, 这里我们是用在`put`函数而不是构造函数里面, 所以为`true`。

**put ( ) 操作的源码** :

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    
    // 首先判断table是否是空的
    // 我们知道, HashMap的三个构造函数中, 都不会初始Table, 因此第一次put值时, table一定是空的, 需要初始化
    // table的初始化用到了resize函数, 这个我们上一篇文章已经讲过了
    // 由此可见table的初始化是延迟到put操作中的
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
        
    // 这里利用 `(n-1) & hash` 方法计算 key 所对应的下标
    // 如果key所对应的桶里面没有值, 我们就新建一个Node放入桶里面
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    
    // 到这里说明目标位置桶里已经有东西了
    else {
        Node<K,V> e; K k;
        // 这里先判断当前待存储的key值和已经存在的key值是否相等
        // key值相等必须满足两个条件
        //    1. hash值相同
        //    2. 两者 `==` 或者 `equals` 等
        if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k))))
            e = p; // key已经存在的情况下, e保存原有的键值对
        
        // 到这里说明要保存的桶已经被占用, 且被占用的位置存放的key与待存储的key值不一致
        
        // 前面已经说过, 当链表长度超过8时, 会用红黑树存储, 这里就是判断存储桶中放的是链表还是红黑树
        else if (p instanceof TreeNode)
            // 红黑树的部分以后有机会再说吧
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        
        //到这里说明是链表存储, 我们需要顺序遍历链表
        else {
            for (int binCount = 0; ; ++binCount) {
                // 如果已经找到了链表的尾节点了,还没有找到目标key, 则说明目标key不存在，那我们就新建一个节点, 把它接在尾节点的后面
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    // 如果链表的长度达到了8个, 就将链表转换成红黑数以提升查找性能
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                // 如果在链表中找到了目标key则直接退出
                // 退出时e保存的是目标key的键值对
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        
        // 到这里说明要么待存储的key存在, e保存已经存在的值
        // 要么待存储的key不存在, 则已经新建了Node将key值插入, e的值为Null
        
        // 如果待存储的key值已经存在
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            
            // 前面已经解释过, onlyIfAbsent的意思
            // 这里是说旧值存在或者旧值为null的情况下, 用新值覆盖旧值
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e); //这个函数只在LinkedHashMap中用到, 这里是空函数
            // 返回旧值
            return oldValue;
        }
    }
    
    // 到这里说明table中不存在待存储的key, 并且我们已经将新的key插入进数组了
    
    ++modCount; // 这个暂时用不到
    
    // 因为又插入了新值, 所以我们得把数组大小加1, 并判断是否需要重新扩容
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict); //这个函数只在LinkedHashMap中用到, 这里是空函数
    return null;
}
```

put方法的流程图：

![image-20210807110830026](D:\1书本笔记\java实战项目\image-20210807110830026.png)

**总结**

1. 在put之前会检查table是否为空，说明table真正的初始化并不是发生在构造函数中， 而是发生在第一次put的时候。
2. 查找当前key是否存在的条件是`p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k)))`
3. 如果插入的key值不存在，则值会插入到链表的末尾。
4. 每次插入操作结束后，都会检查当前table节点数是否大于`threshold`, 若超过，则扩容。
5. 当链表长度超过`TREEIFY_THRESHOLD`（默认是8）个时，会将链表转换成红黑树以提升查找性能。

### 6 HashMap为什么数组的长度要取2的整数次幂？

比如数组的长度是2^m，这样取的话，可以是HashMap里面的key刚好是在HashMap里面对应key的hash值的最后m位。

![image-20210807101426938](D:\1书本笔记\java实战项目\image-20210807101426938.png)

**性能提升**

前面我们提到, 将hash值转换成数组下标我们可以采用取模运算, 但是取模运算是十分耗时的.

另一方面, 我们知道, 当一个数是 2^n 时, 任意整数对2^n取模等效于:

```apache
h % 2^n = h & (2^n -1)
```

这样我们就将取模操作转换成了位操作, 而位操作的速度远远快于取模操作.

为此, HashMap中, table的大小都是2的n次方, 即使你在构造函数中指定了table的大小, HashMap也会将该值扩大为距离它最近的2的整数次幂的值. 这在我们下面分析构造函数的时候就能看到了.

### 7  为什么负载因子设置为0.75？

​		当负载因子是1.0的时候，也就意味着，只有当数组的8个值（这个图表示了8个）全部填充了，才会发生扩容。这就带来了很大的问题，因为Hash冲突时避免不了的。当负载因子是1.0的时候，意味着会出现大量的Hash的冲突，底层的红黑树变得异常复杂。对于查询效率极其不利。这种情况就是牺牲了时间来保证空间的利用率。

​		负载因子是0.5的时候，这也就意味着，当数组中的元素达到了一半就开始扩容，既然填充的元素少了，Hash冲突也会减少，那么底层的链表长度或者是红黑树的高度就会降低。查询效率就会增加。

​		负载因子是0.75的时候，空间利用率比较高，而且避免了相当多的Hash冲突，使得底层的链表或者是红黑树的高度比较低，提升了空间效率。

​		哈希冲突满足泊松分布的特点，当负载因子取到0.7~0.8的时候。已经可以保证在百万分之一的情况下不发生哈希冲突了、

​		作为一般规则，默认负载系数(.75)提供**了时间和空间成本之间的良好权衡**。 较高的值会减少空间开销，但会增加查找成本(反映在HashMap类的大多数操作中，包括get和put)。 在设置映射的初始容量时，应该考虑映射中预期的条目数量及其负载因子，以减少重新散列操作的数量。 如果初始容量大于最大条目数除以负载因子，则不会发生重新散列操作  

### 8 HashMap的get方法

具体的细节怕是通过key的hash值确定想要查找的node节点在table[ ]中的下标位置，然后通过 进一步判断 k在对应的桶里是否存在来判断，能否找到对应的元素。

### 9 HashMap的链表长度为啥要到达8的时候才扩展为红黑树？

​		通常如果 hash 算法正常的话，那么链表的长度也不会很长，那么红黑树也不会带来明显的查询时间上的优势，反而会增加空间负担。所以通常情况下，并没有必要转为红黑树，所以就选择了概率非常小，小于千万分之一概率，也就是长度为 8 的概率，把长度 8 作为转化的默认阈值。

​		通过查看源码可以发现，默认是链表长度达到 8 就转成红黑树，而当长度降到 6 就转换回去，这体现了时间和空间平衡的思想，最开始使用链表的时候，空间占用是比较少的，而且由于链表短，所以查询时间也没有太大的问题。可是当链表越来越长，需要用红黑树的形式来保证查询的效率。对于何时应该从链表转化为红黑树，需要确定一个阈值，这个阈值默认为 8，并且在源码中也对选择 8 这个数字做了说明，原文如下：
​		![image-20210807135236360](D:\1书本笔记\java实战项目\image-20210807135236360.png)

上面这段话的意思是，如果 hashCode 分布良好，也就是 hash 计算的结果离散好的话，那么红黑树这种形式是很少会被用到的，因为各个值都均匀分布，很少出现链表很长的情况。在理想情况下，链表长度符合泊松分布，各个长度的命中概率依次递减，当长度为 8 的时候，概率仅为 0.00000006。这是一个小于千万分之一的概率。

### 10 为什么HashMap中的String,Integer这样的包装类适合作为key?

1. String和Integer这类包装类，保证了Hash的不可更改性（这里是因为final类型，即具有不可更改性）和计算准确性（内部重写了equals和hasCode() 方法，不易出现Hash值的计算错误）
2. 有效减少了Hash碰撞的几率

### 11**HashMap 中的 key若 Object类型， 则需实现哪些方法？**

需要实现hasCode()和equals()方法

1. hasCode()方法：

计算需要存储数据的位置（实现的不恰当会导致严重的哈希冲突）

2. equals()方法

比较位置上面是否存在需要处理的节点key（保证key在哈希表中的唯一性）。

### 12 HashMap和Hashtable的区别

1. **继承的父类不同** ，HashMap继承自AbstractMap类。但二者都实现了Map接口。
   Hashtable继承自Dictionary类，Dictionary类是一个已经被废弃的类（见其源码中的注释）。父类都被废弃，自然而然也没人用它的子类Hashtable了。
2. **HashMap线程不安全,HashTable线程安全**， Hashtable 中的方法大多是Synchronize的，而HashMap中的方法在一般情况下是非Synchronize的。在多线程并发的环境下，可以直接使用Hashtable，不需要自己为它的方法实现同步，但使用HashMap时就必须要自己增加同步处理。HashTable实现线程安全的代价就是效率变低，因为会锁住整个HashTable,而ConcurrentHashMap做了相关优化,因为ConcurrentHashMap使用了分段锁，并不对整个数据进行锁定,效率比HashTable高很多。
3. **包含的contains方法不同**，HashMap是没有contains方法的，而包括containsValue和containsKey方法；hashtable则保留了contains方法，效果同containsValue,还包括containsValue和containsKey方法。
4. **是否允许null**,  Hashmap是允许key和value为null值的，用containsValue和containsKey方法判断是否包含对应键值对；HashTable键值对都不能为空，否则包空指针异常。
5. **计算hash值的方式不同**，HashMap有个hash方法重新计算了key的hash值,因为hash冲突变高，所以通过一种方法（ (h = key.hashCode()) ^ (h >>> 16)）重算hash值的方法，但是**Hashtable通过计算**key的hashCode()，来得到hash值就为最终hash值。并且计算索引的方式也不同，其中hashmap:index = (n - 1) & hash  而 hashtable:  index = (hash & 0x7FFFFFFF) % tab.length;
6. 扩容方式不同（容量不够）：HashMap 哈希扩容必须要求为原容量的2倍，Hashtable扩容为原容量2倍加1。
7. 解决hash冲突方式不同（地址冲突）：HashMap中如果冲突数量小于8，则是以链表方式解决冲突。 2.而当冲突大于等于8时，就会将冲突的Entry转换为**红黑树进行存储。** 3.而又当数量小于6时，则又转化为链表存储。而在HashTable中， **都是以链表方式存储**。

### 13 HashMap的快速失败fail-fast ( 快速失败 )

fail-fast:直接在容器上进行遍历，在遍历过程中，一旦发现容器中的数据被修改了，会立刻抛出ConcurrentModificationException异常导致遍历失败。java.util包下的集合类都是快速失败机制的, 常见的的使用fail-fast方式遍历的容器有HashMap和ArrayList等。

在使用迭代器遍历一个集合对象时,比如增强for,如果遍历过程中对集合对象的内容进行了修改(增删改),会抛出ConcurrentModificationException 异常.

### 14 HashMap和HashSet的区别

(1)HashSet实现了Set接口, 仅存储对象; HashMap实现了 Map接口, 存储的是键值对.

(2)HashSet底层其实是用HashMap实现存储的, HashSet封装了一系列HashMap的方法. 依靠HashMap来存储元素值,(利用hashMap的key键进行存储), 而value值默认为Object对象. 所以HashSet也不允许出现重复值, 判断标准和HashMap判断标准相同, 两个元素的hashCode相等并且通过equals()方法返回true.



## 2 LinkedHashMap





## 3 ConcurrentHashmap

###  1 ConcurrentHashmap的实现原理

只需要看这篇博客：http://www.justdojava.com/2019/12/18/java-collection-15.1/

ConcurrentHashMap 在 JDK1.7 和 JDK1.8 的实现方式是不同的。

**先来看下JDK1.7**

JDK1.7 中的 ConcurrentHashMap 是由 `Segment` 数组结构和 `HashEntry` 数组结构组成，即 ConcurrentHashMap 把哈希桶数组切分成小数组（Segment ），每个小数组有 n 个 HashEntry 组成。

如下图所示，首先将数据分为一段一段的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一段数据时，其他段的数据也能被其他线程访问，实现了真正的并发访问。

Segment 继承了 **ReentrantLock**，所以 Segment 是一种可重入锁，扮演锁的角色。Segment 默认为 16，也就是并发度为 16。

![image-20210807230211021](D:\1书本笔记\java实战项目\image-20210807230211021.png)

**再来看下JDK1.8**

在数据结构上， JDK1.8 中的ConcurrentHashMap 选择了与 HashMap 相同的**Node数组+链表+红黑树**结构；在锁的实现上，抛弃了原有的 Segment 分段锁，采用`CAS + synchronized`实现更加细粒度的锁。

将锁的级别控制在了更细粒度的哈希桶数组元素级别，也就是说只需要锁住这个链表头节点（红黑树的根节点），就不会影响其他的哈希桶数组元素的读写，大大提高了并发度。

![image-20210807230302128](D:\1书本笔记\java实战项目\image-20210807230302128.png)

#### 1 JDK1.8 中为什么使用内置锁 synchronized替换 可重入锁 ReentrantLock？

- 在 JDK1.6 中，对 synchronized 锁的实现引入了大量的优化，并且 synchronized 有多种锁状态，会从无锁 -> 偏向锁 -> 轻量级锁 -> 重量级锁一步步转换。
- 减少内存开销 。假设使用可重入锁来获得同步支持，那么每个节点都需要通过继承 AQS 来获得同步支持。但并不是每个节点都需要获得同步支持的，只有链表的头节点（红黑树的根节点）需要同步，这无疑带来了巨大内存浪费。





### 2 ConcurrentHashmap的put操作逻辑

前提知识：

1. **table**：默认为null，初始化发生在第一次插入操作，默认大小为16的数组，用来存储Node节点数据，扩容时大小总是2的幂次方。
2. **nextTable**：默认为null，扩容时新生成的数组，其大小为原数组的两倍。

- 其余情况：
   1、如果table未初始化，表示table需要初始化的大小。
   2、如果table初始化完成，表示table的容量，默认是table大小的0.75倍，居然用这个公式算0.75（n - (n >>> 2)）。
- **Node**：保存key，value及key的hash值的数据结构。

**put操作是通过CAS+synchronized实现的**

 **put操作**

当进行 put 操作时，流程大概可以分如下几个步骤：

- 首先会判断 key、value是否为空，如果为空就抛异常！
- 接着会判断容器数组是否为空，如果为空就初始化数组；
- 进一步判断，要插入的元素`f`，在当前数组下标是否第一次插入，如果是就通过 CAS 方式插入；
- 在接着判断`f.hash == -1`是否成立，如果成立，说明当前`f`是`ForwardingNode`节点，表示有其它线程正在扩容，则一起进行扩容操作；
- 其他的情况，就是把新的`Node`节点按链表或红黑树的方式插入到合适的位置；
- 节点插入完成之后，接着判断链表长度是否超过`8`，如果超过`8`个，就将链表转化为红黑树结构；
- 最后，插入完成之后，进行扩容判断；

源码：

![img](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-collection-15/03e9c0acf2354d17a7a6bcf04f39fdbf.jpg)

#### 2.1 initTable 初始化数组

我们再来看看源码中的第3步 `initTable()`方法，如果数组为空就**初始化数组**，源码如下：

![img](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-collection-15/52b5af5e886341658071ccdeb9e798a7.jpg)

sizeCtl 是一个对象属性，使用了volatile关键字修饰保证并发的可见性，默认为 0，当第一次执行 put 操作时，通过`Unsafe.compareAndSwapInt()`方法，俗称`CAS`，将 `sizeCtl`修改为 `-1`，有且只有一个线程能够修改成功，接着执行 table 初始化任务。

如果别的线程发现`sizeCtl<0`，意味着有另外的线程执行CAS操作成功，当前线程通过执行`Thread.yield()`让出 CPU 时间片等待 table 初始化完成。

#### 2.2 helpTransfer 帮组扩容

​		我们继续来看看 put 方法中第5步`helpTransfer()`方法，如果`f.hash == -1`成立，说明当前`f`是`ForwardingNode`节点，意味有其它线程正在扩容，则一起进行扩容操作，源码如下：

![img](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-collection-15/3da0fa6fb1f4412c88282270748a6cee.jpg)

这个过程，操作步骤如下：

- 第1步，对 table、node 节点、node 节点的 nextTable，进行数据校验；
- 第2步，根据数组的length得到一个标识符号；
- 第3步，进一步校验 nextTab、tab、sizeCtl 值，如果 nextTab 没有被并发修改并且 tab 也没有被并发修改，同时 `sizeCtl < 0`，说明还在扩容；
- 第4步，对 sizeCtl 参数值进行分析判断，如果不满足任何一个判断，将`sizeCtl + 1`, 增加了一个线程帮助其扩容;



#### 2.3 ConcurrentHashmap的扩容

 java扩容机制： https://blog.csdn.net/ZOKEKAI/article/details/90051567

```java
//调用该扩容方法的地方有：
//java.util.concurrent.ConcurrentHashMap#addCount        向集合中插入新数据后更新容量计数时发现到达扩容阈值而触发的扩容
//java.util.concurrent.ConcurrentHashMap#helpTransfer    扩容状态下其他线程对集合进行插入、修改、删除、合并、compute 等操作时遇到 ForwardingNode 节点时触发的扩容
//java.util.concurrent.ConcurrentHashMap#tryPresize      putAll批量插入或者插入后发现链表长度达到8个或以上，但数组长度为64以下时触发的扩容
private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
    int n = tab.length, stride;
    //计算每条线程处理的桶个数，每条线程处理的桶数量一样，如果CPU为单核，则使用一条线程处理所有桶
    //每条线程至少处理16个桶，如果计算出来的结果少于16，则一条线程处理16个桶
    if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
        stride = MIN_TRANSFER_STRIDE; // subdivide range
    if (nextTab == null) {            // 初始化新数组(原数组长度的2倍)
        try {
            @SuppressWarnings("unchecked")
            Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1];
            nextTab = nt;
        } catch (Throwable ex) {      // try to cope with OOME
            sizeCtl = Integer.MAX_VALUE;
            return;
        }
        nextTable = nextTab;
        //将 transferIndex 指向最右边的桶，也就是数组索引下标最大的位置
        transferIndex = n;
    }
    int nextn = nextTab.length;
    //新建一个占位对象，该占位对象的 hash 值为 -1 该占位对象存在时表示集合正在扩容状态，key、value、next 属性均为 null ，nextTable 属性指向扩容后的数组
    //该占位对象主要有两个用途：
    //   1、占位作用，用于标识数组该位置的桶已经迁移完毕，处于扩容中的状态。
    //   2、作为一个转发的作用，扩容期间如果遇到查询操作，遇到转发节点，会把该查询操作转发到新的数组上去，不会阻塞查询操作。
    ForwardingNode<K,V> fwd = new ForwardingNode<K,V>(nextTab);
    //该标识用于控制是否继续处理下一个桶，为 true 则表示已经处理完当前桶，可以继续迁移下一个桶的数据
    boolean advance = true;
    //该标识用于控制扩容何时结束，该标识还有一个用途是最后一个扩容线程会负责重新检查一遍数组查看是否有遗漏的桶
    boolean finishing = false; // to ensure sweep before committing nextTab
    //这个循环用于处理一个 stride 长度的任务，i 后面会被赋值为该 stride 内最大的下标，而 bound 后面会被赋值为该 stride 内最小的下标
    //通过循环不断减小 i 的值，从右往左依次迁移桶上面的数据，直到 i 小于 bound 时结束该次长度为 stride 的迁移任务
    //结束这次的任务后会通过外层 addCount、helpTransfer、tryPresize 方法的 while 循环达到继续领取其他任务的效果
    for (int i = 0, bound = 0;;) {
        Node<K,V> f; int fh;
        while (advance) {
            int nextIndex, nextBound;
            //每处理完一个hash桶就将 bound 进行减 1 操作
            if (--i >= bound || finishing)
                advance = false;
            else if ((nextIndex = transferIndex) <= 0) {
                //transferIndex <= 0 说明数组的hash桶已被线程分配完毕，没有了待分配的hash桶，将 i 设置为 -1 ，后面的代码根据这个数值退出当前线的扩容操作
                i = -1;
                advance = false;
            }
            //只有首次进入for循环才会进入这个判断里面去，设置 bound 和 i 的值，也就是领取到的迁移任务的数组区间
            else if (U.compareAndSwapInt(this, TRANSFERINDEX, nextIndex, nextBound = (nextIndex > stride ? nextIndex - stride : 0))) {
                bound = nextBound;
                i = nextIndex - 1;
                advance = false;
            }
        }
        if (i < 0 || i >= n || i + n >= nextn) {
            int sc;
            //扩容结束后做后续工作，将 nextTable 设置为 null，表示扩容已结束，将 table 指向新数组，sizeCtl 设置为扩容阈值
            if (finishing) {
                nextTable = null;
                table = nextTab;
                sizeCtl = (n << 1) - (n >>> 1);
                return;
            }
            //每当一条线程扩容结束就会更新一次 sizeCtl 的值，进行减 1 操作
            if (U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {
                //(sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT 成立，说明该线程不是扩容大军里面的最后一条线程，直接return回到上层while循环
                if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
                    return;
                //(sc - 2) == resizeStamp(n) << RESIZE_STAMP_SHIFT 说明这条线程是最后一条扩容线程
                //之所以能用这个来判断是否是最后一条线程，因为第一条扩容线程进行了如下操作：
                //    U.compareAndSwapInt(this, SIZECTL, sc, (rs << RESIZE_STAMP_SHIFT) + 2)
                //除了修改结束标识之外，还得设置 i = n; 以便重新检查一遍数组，防止有遗漏未成功迁移的桶
                finishing = advance = true;
                i = n; // recheck before commit
            }
        }
        else if ((f = tabAt(tab, i)) == null)
            //遇到数组上空的位置直接放置一个占位对象，以便查询操作的转发和标识当前处于扩容状态
            advance = casTabAt(tab, i, null, fwd);
        else if ((fh = f.hash) == MOVED)
            //数组上遇到hash值为MOVED，也就是 -1 的位置，说明该位置已经被其他线程迁移过了，将 advance 设置为 true ，以便继续往下一个桶检查并进行迁移操作
            advance = true; // already processed
        else {
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    Node<K,V> ln, hn;
                    //该节点为链表结构
                    if (fh >= 0) {
                        int runBit = fh & n;
                        Node<K,V> lastRun = f;
                        //遍历整条链表，找出 lastRun 节点
                        for (Node<K,V> p = f.next; p != null; p = p.next) {
                            int b = p.hash & n;
                            if (b != runBit) {
                                runBit = b;
                                lastRun = p;
                            }
                        }
                        //根据 lastRun 节点的高位标识(0 或 1)，首先将 lastRun设置为 ln 或者 hn 链的末尾部分节点，后续的节点使用头插法拼接
                        if (runBit == 0) {
                            ln = lastRun;
                            hn = null;
                        }
                        else {
                            hn = lastRun;
                            ln = null;
                        }
                        //使用高位和低位两条链表进行迁移，使用头插法拼接链表
                        for (Node<K,V> p = f; p != lastRun; p = p.next) {
                            int ph = p.hash; K pk = p.key; V pv = p.val;
                            if ((ph & n) == 0)
                                ln = new Node<K,V>(ph, pk, pv, ln);
                            else
                                hn = new Node<K,V>(ph, pk, pv, hn);
                        }
                        //setTabAt方法调用的是 Unsafe 类的 putObjectVolatile 方法
                        //使用 volatile 方式的 putObjectVolatile 方法，能够将数据直接更新回主内存，并使得其他线程工作内存的对应变量失效，达到各线程数据及时同步的效果
                        //使用 volatile 的方式将 ln 链设置到新数组下标为 i 的位置上
                        setTabAt(nextTab, i, ln);
                        //使用 volatile 的方式将 hn 链设置到新数组下标为 i + n(n为原数组长度) 的位置上
                        setTabAt(nextTab, i + n, hn);
                        //迁移完成后使用 volatile 的方式将占位对象设置到该 hash 桶上，该占位对象的用途是标识该hash桶已被处理过，以及查询请求的转发作用
                        setTabAt(tab, i, fwd);
                        //advance 设置为 true 表示当前 hash 桶已处理完，可以继续处理下一个 hash 桶
                        advance = true;
                    }
                    //该节点为红黑树结构
                    else if (f instanceof TreeBin) {
                        TreeBin<K,V> t = (TreeBin<K,V>)f;
                        //lo 为低位链表头结点，loTail 为低位链表尾结点，hi 和 hiTail 为高位链表头尾结点
                        TreeNode<K,V> lo = null, loTail = null;
                        TreeNode<K,V> hi = null, hiTail = null;
                        int lc = 0, hc = 0;
                        //同样也是使用高位和低位两条链表进行迁移
                        //使用for循环以链表方式遍历整棵红黑树，使用尾插法拼接 ln 和 hn 链表
                        for (Node<K,V> e = t.first; e != null; e = e.next) {
                            int h = e.hash;
                            //这里面形成的是以 TreeNode 为节点的链表
                            TreeNode<K,V> p = new TreeNode<K,V>
                                (h, e.key, e.val, null, null);
                            if ((h & n) == 0) {
                                if ((p.prev = loTail) == null)
                                    lo = p;
                                else
                                    loTail.next = p;
                                loTail = p;
                                ++lc;
                            }
                            else {
                                if ((p.prev = hiTail) == null)
                                    hi = p;
                                else
                                    hiTail.next = p;
                                hiTail = p;
                                ++hc;
                            }
                        }
                        //形成中间链表后会先判断是否需要转换为红黑树：
                        //1、如果符合条件则直接将 TreeNode 链表转为红黑树，再设置到新数组中去
                        //2、如果不符合条件则将 TreeNode 转换为普通的 Node 节点，再将该普通链表设置到新数组中去
                        //(hc != 0) ? new TreeBin<K,V>(lo) : t 这行代码的用意在于，如果原来的红黑树没有被拆分成两份，那么迁移后它依旧是红黑树，可以直接使用原来的 TreeBin 对象
                        ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) :
                        (hc != 0) ? new TreeBin<K,V>(lo) : t;
                        hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) :
                        (lc != 0) ? new TreeBin<K,V>(hi) : t;
                        //setTabAt方法调用的是 Unsafe 类的 putObjectVolatile 方法
                        //使用 volatile 方式的 putObjectVolatile 方法，能够将数据直接更新回主内存，并使得其他线程工作内存的对应变量失效，达到各线程数据及时同步的效果
                        //使用 volatile 的方式将 ln 链设置到新数组下标为 i 的位置上
                        setTabAt(nextTab, i, ln);
                        //使用 volatile 的方式将 hn 链设置到新数组下标为 i + n(n为原数组长度) 的位置上
                        setTabAt(nextTab, i + n, hn);
                        //迁移完成后使用 volatile 的方式将占位对象设置到该 hash 桶上，该占位对象的用途是标识该hash桶已被处理过，以及查询请求的转发作用
                        setTabAt(tab, i, fwd);
                        //advance 设置为 true 表示当前 hash 桶已处理完，可以继续处理下一个 hash 桶
                        advance = true;
                    }
                }
            }
        }
    }
}
```





### 3 ConcurrentHashmap的get()方法

get 方法操作就比较简单了，因为不涉及并发操作，直接查询就可以了，源码如下：

![image-20210827215735473](D:\1书本笔记\java实战项目\image-20210827215735473.png)

从源码中可以看出，步骤如下：

- 第1步，判断数组是否为空，通过key定位到数组下标是否为空；
- 第2步，判断node节点第一个元素是不是要找到，如果是直接返回；
- 第3步，如果是红黑树结构，就从红黑树里面查询；
- 第4步，如果是链表结构，循环遍历判断；

#### 1  为什么ConcurrentHashMap的get() 方法不需要加锁?

 先说结论：get操作可以无锁是由于Node的元素**val和指针next是用volatile修饰的**，在多线程环境下线程A修改结点的val或者新增节点的时候是对线程B可见的。

**volatile**登场

对于可见性，Java提供了volatile关键字来保证**可见性**、**有序性**。**但不保证原子性**。
普通的共享变量不能保证可见性，因为普通共享变量被修改之后，什么时候被写入主存是不确定的，当其他线程去读取时，此时内存中可能还是原来的旧值，因此无法保证可见性。

- volatile关键字对于基本类型的修改可以在随后对多个线程的读保持一致，但是对于引用类型如数组，实体bean，仅仅保证引用的可见性，但并不保证引用内容的可见性。。
- 禁止进行指令重排序。

背景：为了提高处理速度，处理器不直接和内存进行通信，而是先将系统内存的数据读到内部缓存（L1，L2或其他）后再进行操作，但操作完不知道何时会写到内存。

- **如果对声明了volatile的变量进行写操作**，JVM就会向处理器发送一条指令，将这个变量所在缓存行的数据写回到系统内存。但是，就算写回到内存，如果其他处理器缓存的值还是旧的，再执行计算操作就会有问题。
- 在多处理器下，为了保证各个处理器的缓存是一致的，就会**实现缓存一致性协议**，当某个CPU在写数据时，如果发现操作的变量是共享变量，则会通知其他CPU告知该变量的缓存行是无效的，因此其他CPU在读取该变量时，发现其无效会重新从主存中加载数据。
  [![img](http://axin-soochow.oss-cn-hangzhou.aliyuncs.com/18-9-11/36101323.jpg)](http://axin-soochow.oss-cn-hangzhou.aliyuncs.com/18-9-11/36101323.jpg)
  **总结下来**：
- 第一：使用volatile关键字会强制将修改的值立即写入主存；
- 第二：使用volatile关键字的话，当线程2进行修改时，会导致线程1的工作内存中缓存变量的缓存行无效（反映到硬件层的话，就是CPU的L1或者L2缓存中对应的缓存行无效）；
- 第三：由于线程1的工作内存中缓存变量的缓存行无效，所以线程1再次读取变量的值时会去主存读取。

![image-20210808103001597](D:\1书本笔记\java实战项目\image-20210808103001597.png)

#### 2 对volatile修饰的数组对get操作没有效果，那加在数组上的volatile的目的是什么？

其实就是为了使得Node数组在扩容的时候对其他线程具有可见性而加的volatile



### 4 ConcurrentHashMap的remove()方法

reomve 方法操作和 put 类似，只是方向是反的，源码如下：

![image-20210827220410436](D:\1书本笔记\java实战项目\image-20210827220410436.png)

replaceNode 方法，源码如下：

![img](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-collection-15/fa6f132af7b743f9b971526c70860c40.jpg)

从源码中可以看出，步骤如下：

- 第1步，循环遍历数组，接着校验参数；
- 第2步，判断是否有别的线程正在扩容，如果是一起扩容；
- 第3步，用 synchronized 同步锁，保证并发时元素移除安全；
- 第4步，因为 `check= -1`，所以不会进行扩容操作，利用CAS操作修改baseCount值；

### 5 ConcurrentHashMap 和 Hashtable 的区别

1. **底层数据结构**： JDK1.7 的 ConcurrentHashMap 底层采用 `分段数组+链表` 实现，而 JDK1.8 的 ConcurrentHashMap 实现跟 HashMap1.8 的数据结构一样，都是 `数组+链表/红黑二叉树`。Hashtable 和 JDK1.8 之前的 HashMap 的底层数据结构类似，都是采用 `数组+链表` 的形式。数组是 HashMap 的主体，链表则是为了解决哈希冲突而存在的；
2. **实现线程安全的方式**： ① 在 JDK1.7 的时候，ConcurrentHashMap（分段锁） 对整个桶数组进行了分割分段( Segment )，每一把锁只锁容器其中的一部分数据，这样多线程访问容器里不同数据段的数据，就不会存在锁竞争，提高了并发访问率。 到了 JDK1.8，摒弃了 Segment 的概念，而是直接用 Node 数组+链表+红黑树的数据结构来实现，并发控制使用 synchronized 和 CAS 来操作，（JDK1.6 以后对 synchronized 锁做了很多的优化） 整个看起来就像是优化过且线程安全的 HashMap，虽然在 JDK1.8 中还能看到 Segment 的数据结构，但是已经简化了属性，只是为了兼容旧版本；② Hashtable (同一把锁) :使用 synchronized 来保证线程安全，效率非常低下。一个线程访问同步方法时，当其他线程也访问同步方法，可能会进入阻塞或轮询状态，如使用 put 添加元素，另一个线程就不能使用 put 添加元素，也不能使用 get，竞争会越来越激烈，效率就越低。

![image-20210807224254704](D:\1书本笔记\java实战项目\image-20210807224254704.png)

![image-20210807224303863](D:\1书本笔记\java实战项目\image-20210807224303863.png)

![image-20210807224315118](D:\1书本笔记\java实战项目\image-20210807224315118.png)

### 6 ConcurrentHashMap 和HashMap的区别?

区别：

1. HashMap不支持并发操作，没有同步方法，ConcurrentHashMap支持并发操作，通过继承 ReentrantLock（JDK1.7重入锁）/CAS和synchronized(JDK1.8内置锁)来进行加锁（分段锁），每次需要加锁的操作锁住的是一个 segment，这样只要保证每个 Segment 是线程安全的，也就实现了全局的线程安全。
2. JDK1.8之前HashMap的结构为数组+链表，JDK1.8之后HashMap的结构为数组+链表+红黑树；JDK1.8之前ConcurrentHashMap的结构为segment数组+数组+链表，JDK1.8之后ConcurrentHashMap的结构为数组+链表+红黑树。

### 7  ConcurrentHashMap中变量使用final和volatile修饰的原因？

使用final**来实现不变模式**（immutable），他是**多线程安全里最简单的一种保障方式**。因为你拿他没有办法，想改变它也没有机会。不变模式主要通过final关键字来限定的。在JMM中final关键字还有特殊的语义。**Final域使得确保初始化安全性**（initialization safety）成为可能，**初始化安全性让不可变形对象不需要同步就能自由地被访问和共享**。

使用volatile来保证某个变量内存的改变对其他线程即时可见，在配合CAS可以实现不加锁对并发操作的支持

remove执行的开始就将table赋给一个局部变量tab，将tab依次复制出来，最后直到该删除位置，将指针指向下一个变量。



### 8  ConcurrentHashMap 不支持key和value为null的原因？

​       一句话解释的意思是：**如果key和value为null，那么结果可能会产生二义性**。

​		我们先来说value 为什么不能为 null。因为 ConcurrentHashMap 是用于多线程的 ，如果`ConcurrentHashMap.get(key)`得到了 null ，这就无法判断，是映射的value是 null ，还是没有找到对应的key而为 null ，就有了二义性。

而用于单线程状态的 HashMap 却可以用`containsKey(key)` 去判断到底是否包含了这个 null 。

我们用**反证法**来推理：

假设 ConcurrentHashMap 允许存放值为 null 的 value，这时有A、B两个线程，线程A调用`ConcurrentHashMap.get(key)`方法，返回为 null ，我们不知道这个 null 是没有映射的 null ，还是存的值就是 null 。

假设此时，返回为 null 的真实情况是没有找到对应的 key。那么，我们可以用 `ConcurrentHashMap.containsKey(key)`来验证我们的假设是否成立，我们期望的结果是返回 false 。

但是在我们调用 `ConcurrentHashMap.get(key)`方法之后，`containsKey`方法之前，线程B执行了`ConcurrentHashMap.put(key, null)`的操作。那么我们调用`containsKey`方法返回的就是 true 了，这就与我们的假设的真实情况不符合了，这就有了二义性。

### 7 ConcurrentHashMap 的并发度是什么？

并发度可以理解为程序运行时能够同时更新 ConccurentHashMap且不产生锁竞争的最大线程数。在JDK1.7中，实际上就是ConcurrentHashMap中的分段锁个数，即Segment[]的数组长度，默认是16，这个值可以在构造函数中设置。如果自己设置了并发度，ConcurrentHashMap 会使用大于等于该值的最小的2的幂指数作为实际并发度，也就是比如你设置的值是17，那么实际并发度是32。

### 3 ConcurrentHashMap 迭代器是强一致性还是弱一致性？

与 HashMap 迭代器是强一致性不同，ConcurrentHashMap 迭代器是弱一致性。

ConcurrentHashMap 的迭代器创建后，就会按照哈希表结构遍历每个元素，但在遍历过程中，内部元素可能会发生变化，如果变化发生在已遍历过的部分，迭代器就不会反映出来，而如果变化发生在未遍历过的部分，迭代器就会发现并反映出来，这就是弱一致性。

### 4 ConcurrentHashMap 的size()方法的流程？

- JDK1.7 和 JDK1.8 对 size 的计算是不一样的。 1.7 中是先不加锁计算三次，如果三次结果不一样在加锁。
- JDK1.8 size 是通过对 baseCount 和 counterCell 进行 CAS 计算，最终通过 **baseCount** 和 遍历 CounterCell 数组得出 size（其中baseCount也是被标记为volatile变量修饰的）。
- JDK 8 推荐使用mappingCount 方法，因为这个方法的返回值是 long 类型，不会因为 size 方法是 int 类型限制最大值。

其中CounterCell 这个类到底是什么？？

CounterCell 这个类到底是什么？我们会发现它使用了 @sun.misc.Contended 标记的类，内部包含一个 volatile 变量。**@sun.misc.Contended 这个注解标识**着这个类防止需要防止 "伪共享"。那么，什么又是伪共享呢？

> 缓存系统中是以缓存行（cache line）为单位存储的。缓存行是2的整数幂个连续字节，一般为32-256个字节。最常见的缓存行大小是64个字节。当多线程修改互相独立的变量时，如果这些变量共享同一个缓存行，就会无意中影响彼此的性能，这就是伪共享。

## 2 List

![image-20210809142110373](D:\1书本笔记\java实战项目\image-20210809142110373.png)

### 1 ArrayList

![image-20210809145901472](D:\1书本笔记\java实战项目\image-20210809145901472.png)

#### 1 ArrayList的几种构造方法

3种

ArrayList()：构造一个初始容量为10的空列表

ArrayList(Collection<?extend E> c)：构造一个包含指定元素的列表

ArrayList( int initialCapcity )：构造一个具有初始容量值得空列表

​		ArrayList就是数组列表，主要用来装载数据，当我们装载的是基本类型的数据int，long，boolean，short，byte…的时候我们只能存储他们对应的包装类，它的主要底层实现是数组Object[] elementData。

**小结**：ArrayList底层是用数组实现的存储。

**特点**：查询效率高，增删效率低，线程不安全。使用频率很高。

#### 2 ArrayList和Vetor的区别？

**首先我们给出标准答案：
1、Vector是线程安全的，ArrayList不是线程安全的。
2、ArrayList在底层数组不够用时在原来的基础上扩展0.5倍，Vector是扩展1倍。**

#### 3 ArrayList和LinkedList的区别?

ArrayList和LinkedList区别及使用场景
1. LinkedList和ArrayList的差别主要来自于Array和LinkedList数据结构的不同。ArrayList是基于**数组**实现的，LinkedList是基于**双链表实现**的。另外LinkedList类**不仅是List接口的实现类**，可以根据索引来随机访问集合中的元素，除此之外，**LinkedList还实现了Deque接口**，Deque接口是Queue接口的子接口，它代表一个双向队列，因此LinkedList可以作为双向队列 ，栈（可以参见Deque提供的接口方法）和List集合使用，功能强大。

2. 因为**Array是基于索引(index)的数据结构**，它使用索引在数组中搜索和读取数据是很快的，可以直接返回数组中index位置的元素，因此在随机访问集合元素上有较好的性能。Array获取数据的时间复杂度是O(1),但是要插入、删除数据却是开销很大的，因为这需要移动数组中插入位置之后的的所有元素。

3. 相对于ArrayList，LinkedList的随机访问集合元素时性能较差，**因为需要在双向列表中找到要index的位置**，再返回；但在插入，删除操作是更快的。因为LinkedList不像ArrayList一样，不需要改变数组的大小，也不需要在数组装满的时候要将所有的数据重新装入一个新的数组，这是ArrayList最坏的一种情况，时间复杂度是O(n)，而LinkedList中插入或删除的时间复杂度仅为O(1)。ArrayList在插入数据时还需要更新索引（除了插入数组的尾部）。

4. **LinkedList需要更多的内存**，因为ArrayList的每个索引的位置是实际的数据，**而LinkedList中的每个节点中存储的是实际的数据和前后节点的位置**。

使用场景：

（1）如果应用程序对数据有较多的随机访问，ArrayList对象要优于LinkedList对象；

  ( 2 ) 如果应用程序有更多的插入或者删除操作，较少的随机访问，LinkedList对象要优于ArrayList对象；

（3）不过ArrayList的插入，删除操作也不一定比LinkedList慢，如果在List靠近末尾的地方插入，那么ArrayList只需要移动较少的数据，而LinkedList则需要一直查找到列表尾部，反而耗费较多时间，这时ArrayList就比LinkedList要快。


#### 4 ArrayList的自动扩容

​			通过无参构造方法的方式ArrayList()初始化，则赋值底层数Object[] elementData为一个默认空数组Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {}所以数组容量为0，只有真正对数据进行添加add时，才分配默认DEFAULT_CAPACITY = 10的初始容量。（**默认的初始容量是10**）。

##### 1能具体说下1.7和1.8版本初始化的时候的区别么？

​		arrayList1.7开始变化有点大，一个是初始化的时候，1.7以前会调用this(10)才是真正的容量为10，1.7即本身以后是默认走了空数组，**只有第一次add的时候容量会变成10**

##### 2  通过grow()方法来进行扩容

​	 每次add的时候主要还要检测ArrayList的容量够不够？**不够的话将原来的容量扩大为之前的1.5倍。**

![image-20210809143027395](D:\1书本笔记\java实战项目\image-20210809143027395.png)

#### 5 ArrayList的add(int index,int val)方法和remove(int index)方法

**add(int index,int val)方法: 本质是数组的复制粘贴，所以比较耗时**

![image-20210809143510680](D:\1书本笔记\java实战项目\image-20210809143510680.png)

![image-20210809143527256](D:\1书本笔记\java实战项目\image-20210809143527256.png)

**remove(int index)方法：**

![image-20210809143835985](D:\1书本笔记\java实战项目\image-20210809143835985.png)

![image-20210809143900067](D:\1书本笔记\java实战项目\image-20210809143900067.png)

#### 6 对Arrsys.asList()的认识？

数组转List集合，这个方法，比较重要
    注意这里有两个容易犯的错误。
      1 **原始数据类型的数组，不能直接直接转化为List。应该把它变成包装类再进行转化**。
      2 **Arrays.asList() 方法得到的list是一个长度大小固定不变的list。如果需要该边它的大小，需要参考以下的方法**：

#### 7 为什么ArrayList访问的时候很快？

ArrayList 还实现了 **RandomAccess 接口**，这是一个标记接口，**LinkedList 没有实现 RandomAccess 接口**，这是因为 LinkedList 存储数据的内存地址是不连续的，所以不支持随机访问。

![image-20210809150320532](D:\1书本笔记\java实战项目\image-20210809150320532.png)

#### 8 ArrayList 还实现了 Serializable 接口

​		ArrayList 不想像数组这样活着，它想能屈能伸，所以它实现了动态扩容。一旦在添加元素的时候，发现容量用满了 `s == elementData.length`，就按照原来数组的 1.5 倍。

**问题：**

比如说，默认的数组大小是 10，当添加第 11 个元素的时候，数组的长度扩容了 1.5 倍，也就是 15，意味着还有 4 个内存空间是闲置的，对吧？

序列化的时候，如果把整个数组都序列化的话，是不是就多序列化了 4 个内存空间。当存储的元素数量非常非常多的时候，闲置的空间就非常非常大，序列化耗费的时间就会非常非常多。

ArrayList 做了一个愉快而又聪明的决定，内部提供了两个私有方法 writeObject （）

![image-20210809150517060](D:\1书本笔记\java实战项目\image-20210809150517060.png)

从 writeObject 方法的源码中可以看得出，它使用了 **ArrayList 的实际大小 size 而不是数组的长度**（`elementData.length`）来作为元素的上限进行序列化。

（值得一提的是ArrayList 中数组的大小 initialCapacity 和数组中的元素个数 size 是两个不同的东西。）

### 2 LinkedList

![image-20210809151307563](D:\1书本笔记\java实战项目\image-20210809151307563.png)

LinkedList 是一个继承自 AbstractSequentialList 的双向链表，因此它也可以被当作堆栈、队列或双端队列进行操作

### 3 CopyOnWriteArrayList





## 3 Queue

### 1 PriorityQueue

#### 1  PriorityQueue的使用？

PriorityQueue可以自动的创建大根堆或者小根堆（默认小根堆）。

**PriorityQueue创建大根堆的方法**：（通过lamada内置函数来进行写，会方便很多）

lamada后面的属性和前面的泛型很有关系，

1. 比如我们想通过字符串的长度来构建一个大根堆

```
PriorityQueue<String> queue = new PriorityQueue<>((o1,o2)->o2.length()-o1.length());
```

2. 比如我们想通过比较int[] 数组中元素的大小来构造一个大根堆。

```java
PriorityQueue<Intrger> queue = new PriorityQueue<>((o1,o2)->o2-o1);
```

**PriorityQueue创建小根堆的方法:**

```java
PriorityQueue<Intrger> queue = new PriorityQueue<>((o1,o2)->o1-o2);
//采用或者默认就行了
PriorityQueue<Intrger> queue = new PriorityQueue<>();
```

#### 2 PriorityQueue的方法？

这里需要注意的一点是poll()方法，返回的是根的root节点，也就是头节点。

![image-20210808135440876](D:\1书本笔记\java实战项目\image-20210808135440876.png)

### 2  ConcurrentLinkedQueue

