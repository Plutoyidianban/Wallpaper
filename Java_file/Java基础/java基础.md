# 1 JAVA基础

##  1 Object根类



### 1  对equals方法的理解？

equals()的作用是用来判断两个对象是否相等，在Object里面的定义是：

```
public boolean equals(Object obj) {
    return (this == obj);
}
```

**覆写equals时有哪些准则?**

这个我在Effective Java上看过，没记错的话应该是：

> **自反性**：对于任何非空引用值 x，x.equals(x) 都应返回 true。

> **对称性**：对于任何非空引用值 x 和 y，当且仅当 y.equals(x) 返回 true 时，x.equals(y) 才应返回 true。

> **传递性**：对于任何非空引用值 x、y 和 z，如果 x.equals(y) 返回 true， 并且 y.equals(z) 返回 true，那么 x.equals(z) 应返回 true。

> **一致性**：对于任何非空引用值 x 和 y，多次调用 x.equals(y) 始终返回 true 或始终返回 false， 前提是对象上 equals 比较中所用的信息没有被修改。

> **非空性**：对于任何非空引用值 x，x.equals(null) 都应返回 false。



上面的equals有以下几点诀窍：

- **使用==操作符检查“参数是否为这个对象的引用”**：如果是对象本身，则直接返回，拦截了对本身调用的情况，算是一种性能优化。
- **使用instanceof操作符检查“参数是否是正确的类型”**：如果不是，就返回false，正如对称性和传递性举例子中说得，不要想着兼容别的类型，很容易出错。在实践中检查的类型多半是equals所在类的类型，或者是该类实现的接口的类型，比如Set、List、Map这些集合接口。
- **把参数转化为正确的类型**： 经历了上一步的检测，基本会成功。
- **对于该类中的“关键域”，检查参数中的域是否与对象中的对应域相等**：基本类型的域就用`==`比较，float域用Float.compare方法，double域用Double.compare方法，至于别的引用域，我们一般递归调用它们的equals方法比较，加上判空检查和对自身引用的检查，一般会写成这样：`(field == o.field || (field != null && field.equals(o.field)))`,而上面的String里使用的是数组，所以只要把数组中的每一位拿出来比较就可以了。
- **编写完成后思考是否满足上面提到的对称性，传递性，一致性等等**



### 2   ==与equals方法的区别？

equals()的作用是用来判断两个对象是否相等，在Object里面的定义是：

```
public boolean equals(Object obj) {
    return (this == obj);
}
复制代码
```

这说明在我们实现自己的equals方法之前，equals等价于`==`,而`==`运算符是判断两个对象是不是同一个对象，即他们的**地址是否相等**。而覆写equals更多的是追求两个对象在**逻辑上的相等**，你可以说是**值相等**，也可说是**内容相等**。

在以下几种条件中，不覆写equals就能达到目的：

- **类的每个实例本质上是唯一的**：强调活动实体的而不关心值得，比如Thread，我们在乎的是哪一个线程，这时候用equals就可以比较了。
- **不关心类是否提供了逻辑相等的测试功能**：有的类的使用者不会用到它的比较值得功能，比如Random类，基本没人会去比较两个随机值吧
- **超类已经覆盖了equals，子类也只需要用到超类的行为**：比如AbstractMap里已经覆写了equals，那么继承的子类行为上也就需要这个功能，那也不需要再实现了。
- **类是私有的或者包级私有的，那也用不到equals方法**：这时候需要覆写equals方法来禁用它：`@Override public boolean equals(Object obj) { throw new AssertionError();}`






### 3  简述对hashcode()方法的理解？

​		Object 类中定义的 hashCode 方法为不同的对象返回不同的整形值。具有迷惑异议的地方就是`This is typically implemented by converting the internal  address of the object into an integer`这一句，意为通常情况下实现的方式是将**对象的内部地址转换为整形值**。

​		其实官方的代码里面有6种，这里提供了6种计算 hash 值的方案，有自增序列，随机数，关联内存地址等多种方式，其中官方默认的是最后一种，即随机数生成。可以看出 hashCode 也许和内存地址有关系，但不是直接代表内存地址的，具体需要看虚拟机版本和设置。



### 4  为什么重写equals方法必须要从写hashcode()方法？

因为如果不这样做的话，就会违反 hashCode 的通用约定，从而导致该类无法结合所有基于散列的集合一起正常工作，这类集合包括 HashMap 和 HashSet。

这里的**通用约定**，从 Object 类的 hashCode 方法的注释可以了解，主要包括以下几个方面，

- 在应用程序的执行期间，只要对象的 equals 方法的比较操作所用到的信息没有被修改，那么对同一个对象的多次调用，hashCode 方法都必须始终返回同一个值。
- 如果两个对象根据 equals **方法比较是相等的**，那么调用这两个对象中的 hashCode **方法都必须产生同样的整数结果**。
- 如果两个对象根据 equals 方法比较是不相等的，那么调用者两个对象中的 hashCode 方法，则不一定要求 hashCode 方法必须产生不同的结果。但是给不相等的对象产生不同的整数散列值，是有可能提高散列表（hash table）的性能。

从理论上来说如果重写了 equals 方法而没有重写 hashCode 方法则违背了上述约定的第二条，**相等的对象必须拥有相等的散列值**。



例子：

​		 如果你不将自定义的类定义为`HashMap`的key值的话，那么我们重写了`equals`方法而没有重写`hashCode`方法，编译器不会报任何错，在运行时也不会抛任何异常。

 		如果你想将自定义的类定义为`HashMap`的key值得话，那么如果重写了`equals`方法那么就必须也重写`hashCode`方法。
 	
 		接下来我们可以看一下我们使用自定义的类作为`HashMap`的key，并且自定义的类不重写`equals`和`hashCode`方法会发生什么。

自定义的类

```
1@Builder
2@NoArgsConstructor
3@AllArgsConstructor
4class CustomizedKey{
5    private Integer id;
6    private String name;
7}
复制代码
```

​		接下来我们看使用自定义的类作为key

```
 1    public static void main(String[] args) {
 2
 3        Map<CustomizedKey, Integer> data = getData();
 4
 5        CustomizedKey key = CustomizedKey.builder().id(1).name("key").build();
 6
 7        Integer integer = data.get(key);
 8
 9        System.out.printf(String.valueOf(integer));
10    }
11
12    private static Map<CustomizedKey,Integer> getData(){
13        Map<CustomizedKey,Integer> customizedKeyIntegerMap = new HashMap<>();
14        CustomizedKey key = CustomizedKey.builder().id(1).name("key").build();
15        customizedKeyIntegerMap.put(key,10);
16        return customizedKeyIntegerMap;
17    }
复制代码
```

我们可以看到程序最后打印的是一个`null`值。原因正如上面我们说的一样。

- `hashCode`：用来计算该对象放入数组中的哪个位置，因为是**两个都是new的对象**，所以**即使里面的值一样**，但是**对象所处的地址却不同**，所以使用默认的`hashCode`也就不同，**当然在`hashMap`中就不会认为两个是一个对象**。

### 5 简述深拷贝和浅拷贝？

**浅拷贝**

1. 基本数据类型的成员变量，进行值传递（将该属性值复制一份给新的对象）。
2. 引用数据类型的成员变量，比如说成员变量是某个数组、某个类的对象等进行引用传递（将该成员变量的引用值（内存地址）复制一份给新的对象）。

**深拷贝**

1. 基本数据类型的成员变量，进行值传递（将该属性值复制一份给新的对象）。
2. 引用数据类型的成员变量，比如说成员变量是某个数组、某个类的对象等，会重新分配内存并将成员变量拷贝一份赋值给新对象（将该成员变量的内容复制一份到新开辟的内存上，新的对象指向新的内存地址）。

### 5 简述对clone()方法的理解？

- 浅拷贝：只是拷贝了对象的实例，但是对象中的属性还是会指向原有的对象。
- 深拷贝：在浅拷贝的基础上，再将对象中的属性额外的复制一份，对象中的属性会指向新的对象。

如何不重写clone( )方法，那么默认是浅拷贝。

**如果只是实现基本类型的赋值，只需要实现`Cloneable`接口即可，但是如果是引用类型的话就需要将这些对象都实现`Cloneable`接口并重写`clone`方法**。

### 6 深拷贝的实现方式？

**1. BeanUtils**
spring提供了BeanUtils来实现对象的克隆，基本原理大体上就是使用反射机制来实现的。BeanUtils实现的并不是深度拷贝，不过BeanUtils可以拷贝两个不同的对象，这一点还是很不错的。

**2. 序列化拷贝**
如果想实现深度拷贝，最简单的方式我觉得是通过序列化的方式来实现。序列化有多种方式，我在这里使用JSON来进行序列化。大体思路是将对象转换成JSON，然后再将json转换成指定对象。

![image-20210809222742108](D:\1书本笔记\java实战项目\image-20210809222742108.png)

结果为:white 修改list的user不会影响到list2。
通过序列化进行拷贝的方式效率很低，在使用时需要三思而后行。如何可以通过浅拷贝实现就尽量不要深度拷贝，如果可以手动拷贝的就尽量不要用序列化进行拷贝。以免造成不必要的性能浪费。



### 7 一个数组的复制方法：Arrays.copyOf()方法和System.arraycopy()的区别？

Arrays.copyOf()底层其实还是调用了System.arraycopy()方法的。

1. 从两种拷贝方式的定义来看：
   System.arraycopy()使用时必须有原数组和目标数组，Arrays.copyOf()使用时只需要有原数组即可。
2. 从两种拷贝方式的底层实现来看：
   System.arraycopy()是用c或c++实现的，Arrays.copyOf()是在方法中重新创建了一个数组，并调用System.arraycopy()进行拷贝。
3. 两种拷贝方式的效率分析：
   由于Arrays.copyOf()不但创建了新的数组而且最终还是调用System.arraycopy()，所以System.arraycopy()的效率高于Arrays.copyOf()。

## 2 数据类型

### 1 基本类型以及其所占的字节数

![image-20210809224228064](D:\1书本笔记\java实战项目\image-20210809224228064.png)

### 2 为什么char是双字节？

要回答标题中的两个问题，先看下面的内容。

Unicode是一种字符集规范，而且还在不断发展之中。我们常说的UTF-8，UTF-16编码是其不同的两种实现。请注意，字符集和编码不是一回事。Unicode规范好比就是定义了每个字符对应一个数字，至于如何把这个数字存放在计算机中，那是另一回事。Unicode收录的每个字符对应一个数字，称作码点(code point)，通常用“U+”后面跟着一个十六进制数表示。

其实UTF-16比UTF-8更早出现，在UTF-16之前还存在UCS-2，该编码固定使用16位编码，后来发现16位根本不够使用，IEEE曾推荐过UCS-4，即固定使用4个字节编码，但因为太过浪费空间被Unicode拒绝采纳了。此时，UTF-16就出现了。Unicode将16位的字符集做了延申，并规定每216为一个**plane, 第一个plane**称作 Basic Multilingual Plane（简称 BMP）, 这里面的字符使用两个字节。对于超出了第一个plane的其他字符，将被收录在其他层级的plane，并使用第一层的代理（surrogate）将其表示，占用四个字节。至于怎么表示以及怎么编码解码的，请移步维基百科，有详细的解释。

JVM内部使用的是UTF-16编码。不管代码文件中char使用的是什么编码，都将被JVM转化为UTF-16而且只用两个字节，也就是说Java中的char占用两个字节，只能表示Unicode中第一层（BMP）中的字符，对于其他字符会报错：Invalid Character Constant, 而String中是可以的。

所以标题中的答案就是，之所以是两个字节，**是因为Java内部使用UTF-16编码，所以char为了表示BMP中的字符，所以占了两个字节。很明显，不能表示所有字符**。

### 3 简述Java中的拆箱和装箱？

**装箱过程是通过调用包装器的valueOf方法实现的，而拆箱过程是通过调用包装器的 xxxValue方法实现的**

**一.什么是装箱？什么是拆箱？**

　　在前面的文章中提到，Java为每种基本数据类型都提供了对应的包装器类型，至于为什么会为每种基本数据类型提供包装器类型在此不进行阐述，有兴趣的朋友可以查阅相关资料。在Java SE5之前，如果要生成一个数值为10的Integer对象，必须这样进行：

Integer i = new`    `Integer(``10``);

　　而在从Java SE5开始就提供了自动装箱的特性，如果要生成一个数值为10的Integer对象，只需要这样就可以了：

```
Integer i = 10;
```

　　这个过程中会自动根据数值创建对应的 Integer对象，这就是装箱。

　　那什么是拆箱呢？顾名思义，跟装箱对应，就是自动将包装器类型转换为基本数据类型：

```
Integer i = 10; //装箱int n = i;  //拆箱
```

　　简单一点说，装箱就是 自动将基本数据类型转换为包装器类型；拆箱就是 自动将包装器类型转换为基本数据类型。

　　下表是基本数据类型对应的包装器类型：

| int（4字节）    | Integer   |
| --------------- | --------- |
| byte（1字节）   | Byte      |
| short（2字节）  | Short     |
| long（8字节）   | Long      |
| float（4字节）  | Float     |
| double（8字节） | Double    |
| char（2字节）   | Character |
| boolean（未定） | Boolean   |

**二.装箱和拆箱是如何实现的**

　　上一小节了解装箱的基本概念之后，这一小节来了解一下装箱和拆箱是如何实现的。

　　我们就以Interger类为例，下面看一段代码：

![image-20210810084813862](D:\1书本笔记\java实战项目\image-20210810084813862.png)

　　反编译class文件之后得到如下内容：

　　![img](https://images0.cnblogs.com/i/288799/201406/101641567956500.jpg)

　　从反编译得到的字节码内容可以看出，在装箱的时候自动调用的是Integer的valueOf(int)方法。而在拆箱的时候自动调用的是Integer的intValue方法。

　　其他的也类似，比如Double、Character，不相信的朋友可以自己手动尝试一下。

　　因此可以用一句话总结装箱和拆箱的实现过程：

　　**装箱过程是通过调用包装器的valueOf方法实现的，而拆箱过程是通过调用包装器的 xxxValue方法实现的**。（xxx代表对应的基本数据类型）。

**三.面试中相关的问题**

　　虽然大多数人对装箱和拆箱的概念都清楚，但是在面试和笔试中遇到了与装箱和拆箱的问题却不一定会答得上来。下面列举一些常见的与装箱/拆箱有关的面试题。

1.下面这段代码的输出结果是什么？

![image-20210810084833389](D:\1书本笔记\java实战项目\image-20210810084833389.png)

　　也许有些朋友会说都会输出false，或者也有朋友会说都会输出true。但是事实上输出结果是：

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

```
true
false
```

 　为什么会出现这样的结果？输出结果表明i1和i2指向的是同一个对象，而i3和i4指向的是不同的对象。此时只需一看源码便知究竟，下面这段代码是Integer的valueOf方法的具体实现：

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

```
public static Integer valueOf(int i) {
        if(i >= -128 && i <= IntegerCache.high)
            return IntegerCache.cache[i + 128];
        else
            return new Integer(i);
    }
```

　　而其中IntegerCache类的实现为：

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 private static class IntegerCache {
        static final int high;
        static final Integer cache[];

        static {
            final int low = -128;

            // high value may be configured by property
            int h = 127;
            if (integerCacheHighPropValue != null) {
                // Use Long.decode here to avoid invoking methods that
                // require Integer's autoboxing cache to be initialized
                int i = Long.decode(integerCacheHighPropValue).intValue();
                i = Math.max(i, 127);
                // Maximum array size is Integer.MAX_VALUE
                h = Math.min(i, Integer.MAX_VALUE - -low);
            }
            high = h;

            cache = new Integer[(high - low) + 1];
            int j = low;
            for(int k = 0; k < cache.length; k++)
                cache[k] = new Integer(j++);
        }

        private IntegerCache() {}
    }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　从这2段代码可以看出，在通过valueOf方法创建Integer对象的时候，如果数值在[-128,127]之间，便返回指向IntegerCache.cache中已经存在的对象的引用；否则创建一个新的Integer对象。

　　**上面的代码中i1和i2的数值为100，因此会直接从cache中取已经存在的对象，所以i1和i2指向的是同一个对象，而i3和i4则是分别指向不同的对象**。

2.下面这段代码的输出结果是什么？

![image-20210810084926703](D:\1书本笔记\java实战项目\image-20210810084926703.png)

有的朋友会认为跟上面一道题目的输出结果相同，但是事实上却不是。实际输出结果为：

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

```
false
false
```

　　至于具体为什么，读者可以去查看Double类的valueOf的实现。

　　在这里只解释一下为什么Double类的valueOf方法会采用与Integer类的valueOf方法不同的实现。很简单：在某个范围内的整型数值的个数是有限的，而浮点数却不是。

　　注意，Integer、Short、Byte、Character、Long这几个类的valueOf方法的实现是类似的。

　　　　　Double、Float的valueOf方法的实现是类似的。

3.下面这段代码输出结果是什么：

![image-20210810084956430](D:\1书本笔记\java实战项目\image-20210810084956430.png)

　　输出结果是：

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

```
true
true
```

　　至于为什么是这个结果，同样地，看了Boolean类的源码也会一目了然。下面是Boolean的valueOf方法的具体实现：

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

```
public static Boolean valueOf(boolean b) {
        return (b ? TRUE : FALSE);
    }
```

　　而其中的 TRUE 和FALSE又是什么呢？在Boolean中定义了2个静态成员属性：

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 public static final Boolean TRUE = new Boolean(true);

    /** 
     * The <code>Boolean</code> object corresponding to the primitive 
     * value <code>false</code>. 
     */
    public static final Boolean FALSE = new Boolean(false);
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　至此，大家应该明白了为何上面输出的结果都是true了。

4.谈谈Integer i = new Integer(xxx)和Integer i =xxx;这两种方式的区别。

　　当然，这个题目属于比较宽泛类型的。但是要点一定要答上，我总结一下主要有以下这两点区别：

　　1）第一种方式不会触发自动装箱的过程；而第二种方式会触发；

　　2）在执行效率和资源占用上的区别。第二种方式的执行效率和资源占用在一般性情况下要优于第一种情况（注意这并不是绝对的）。

5.下面程序的输出结果是什么？

```java
public class Main {
    public static void main(String[] args) {
         
        Integer a = 1;
        Integer b = 2;
        Integer c = 3;
        Integer d = 3;
        Integer e = 321;
        Integer f = 321;
        Long g = 3L;
        Long h = 2L;
        // 1  ==实际上比较的是地址
        // 2  虽然是==，按道理上应该比较的地址但是如果操作符里面有+ - * / 等操作符，那么会自动触发拆箱，那么==此时比较是基本类型的值
        //3  包装类的equals方法不涉及类型的自动转型。
        
        System.out.println(c==d);//true
        System.out.println(e==f);//false 因为范围超过了[-128，127]
        System.out.println(c==(a+b));// true  操作符，那么会自动触发拆箱
        System.out.println(c.equals(a+b));//true  
        System.out.println(g==(a+b));//true
        System.out.println(g.equals(a+b));//false  包装类的equals方法不涉及类型的自动转型。
        System.out.println(g.equals(a+h));//true
    }
}
```

　　先别看输出结果，读者自己想一下这段代码的输出结果是什么。这里面需要注意的是：当 "=="运算符的两个操作数都是 包装器类型的引用，则是比较指向的是否是同一个对象，而如果其中有一个操作数是表达式（即包含算术运算）则比较的是数值（即会触发自动拆箱的过程）。另外，对于包装器类型，equals方法并不会进行类型转换。明白了这2点之后，上面的输出结果便一目了然：

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
true
false
true
true
true
false
true
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　第一个和第二个输出结果没有什么疑问。第三句由于 a+b包含了算术运算，因此会触发自动拆箱过程（会调用intValue方法），因此它们比较的是数值是否相等。而对于c.equals(a+b)会先触发自动拆箱过程，再触发自动装箱过程，也就是说a+b，会先各自调用intValue方法，得到了加法运算后的数值之后，便调用Integer.valueOf方法，再进行equals比较。同理对于后面的也是这样，不过要注意倒数第二个和最后一个输出的结果（如果数值是int类型的，装箱过程调用的是Integer.valueOf；如果是long类型的，装箱调用的Long.valueOf方法）。

### 4.谈谈Integer i = new Integer(xxx)和Integer i =xxx;这两种方式的区别。

　　当然，这个题目属于比较宽泛类型的。但是要点一定要答上，我总结一下主要有以下这两点区别：

　　1）第一种方式不会触发自动装箱的过程；而第二种方式会触发,执行valueOf( )方法；

　　2）在执行效率和资源占用上的区别。第二种方式的执行效率和资源占用在一般性情况下要优于第一种情况（注意这并不是绝对的）。

### 5 隐式转换和显示转换可能出现的问题？

**隐式转换**：底数据位向高数据位转换时。

**显示转换**：高数据位向底数据位转换时，发生的情况（小心使用，因为可能会出现数据丢失的情况）。

（需要提出一点，就是s+=1,这种操作会自动触发，数据类型的自动转换）

![image-20210810091351638](D:\1书本笔记\java实战项目\image-20210810091351638.png)

### 6 包装类中的缓存?

![image-20210810101745130](D:\1书本笔记\java实战项目\image-20210810101745130.png)

**char的缓存池是0-128。**

**两种浮点数类型的包装类`Float`,`Double`并没有实现常量池技术。**

**除了`Integer`以外，其他包装类的缓存范围都不能改变**

### 7 简述Integer中valueOf方法的实现

​		-128 到127 之间的数据会存在Integer的缓存池里面，一开始会先去常量的缓冲池里面找数据类型，如果存在的话，直接返回。如果没有的话，会直接在堆中创建一个新数据。

### 8 字符型常量和字符串常量的区别？

1、形式上: 字符常量是单引号引起的一个字符; 字符串常量是双引号引起的若干个字符
2、含义上: 字符常量相当于一个整型值( ASCII 值),可以参加表达式运算; 字符串常量代表一个地址值(该字符串在内存中存放位置)
3、占内存大小 字符常量只占 2 个字节; 字符串常量占若干个字节 (注意： char 在 Java 中占两个字节)

### 9 包装类存在的意义？

通俗解释就是由于Java是面对对象的语言，而基本类型不具有面对对象的概念，为了弥补不足，引入了包装类方便使用面对对象的变成思想操作基本类型。

## 3 java编码

### 1 简述对java字符、字节、编码的理解？

 编码问题的由来，相关概念的理解

#### 1.1 字符与编码的发展

从计算机对多国语言的支持角度看，大致可以分为三个阶段：

![img](https:////upload-images.jianshu.io/upload_images/2968570-8a73bc7ce9edc1e0.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

字符串在内存中的存放方法：

在 ASCII 阶段，单字节字符串使用一个字节存放一个字符（SBCS）。比如，"Bob123" 在内存中为：

```undefined
42  6F  62  31  32  33  00
                        
B   o   b   1   2   3   \0
```

在使用 ANSI 编码支持多种语言阶段，每个字符使用一个字节或多个字节来表示（MBCS），因此，这种方式存放的字符也被称作多字节字符。比如，"中文123" 在中文 Windows 95 内存中为7个字节，每个汉字占2个字节，每个英文和数字字符占1个字节：

```undefined
D6  D0  CE  C4  31  32  33  00
                
中   文   1   2   3   \0
```

在 UNICODE 被采用之后，计算机存放字符串时，改为存放每个字符在 UNICODE 字符集中的序号。目前计算机一般使用 2 个字节（16 位）来存放一个序号（DBCS），因此，这种方式存放的字符也被称作宽字节字符。比如，字符串 "中文123" 在 Windows 2000 下，内存中实际存放的是 5 个序号：

![img](https:////upload-images.jianshu.io/upload_images/2968570-118c62099372777c.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1115/format/webp)

```undefined
中   文   1   2   3   \0  　
```

一共占 10 个字节。

#### 1.2 字符，字节，字符串

理解编码的关键，是要把字符的概念和字节的概念理解准确。这两个概念容易混淆，我们在此做一下区分：

![img](https:////upload-images.jianshu.io/upload_images/2968570-fa146edffbfc9794.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

由于不同 ANSI 编码所规定的标准是不相同的，因此，对于一个给定的多字节字符串，我们必须知道它采用的是哪一种编码规则，才能够知道它包含了哪些“字符”。而对于 UNICODE 字符串来说，不管在什么环境下，它所代表的“字符”内容总是不变的。

#### 1.3 字符集与编码

各个国家和地区所制定的不同 ANSI 编码标准中，都只规定了各自语言所需的“字符”。比如：汉字标准（GB2312）中没有规定韩国语字符怎样存储。这些 ANSI 编码标准所规定的内容包含两层含义：

1. 使用哪些字符。也就是说哪些汉字，字母和符号会被收入标准中。所包含“字符”的集合就叫做“字符集”。
2. 规定每个“字符”分别用一个字节还是多个字节存储，用哪些字节来存储，这个规定就叫做“编码”。

各个国家和地区在制定编码标准的时候，“字符的集合”和“编码”一般都是同时制定的。因此，平常我们所说的“字符集”，比如：GB2312, GBK, JIS 等，除了有“字符的集合”这层含义外，同时也包含了“编码”的含义。

**“UNICODE 字符集”**包含了各种语言中使用到的所有“字符”。用来给 UNICODE 字符集编码的标准有很多种，比如：UTF-8, UTF-7, UTF-16, UnicodeLittle, UnicodeBig 等。

#### 1.4 常用的编码简介

简单介绍一下常用的编码规则，为后边的章节做一个准备。在这里，我们根据编码规则的特点，把所有的编码分成三类：

![img](https:////upload-images.jianshu.io/upload_images/2968570-379922e285420702.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

我们实际上没有必要去深究每一种编码具体把某一个字符编码成了哪几个字节，我们只需要知道“编码”的概念就是把“字符”转化成“字节”就可以了。对于“UNICODE 编码”，由于它们是可以通过计算得到的，因此，在特殊的场合，我们可以去了解某一种“UNICODE 编码”是怎样的规则。

### 2 为什么需要编码？

我们知道计算机处理的数据实际上都是二级制的数据,也就是计算机实际上只识别0和1两种状态。发明计算机的过程中人们需要解决的第一个问题就是文字的处理问题,也就是我们如何将文字符号转化为二级制数据，同时我们也需要能够将转化后的二进制数据重新转化为文字符号供我们阅读。前面的过程我们称之为编码，后面的这个过程我们称之为解码。这和电信领域更著名的一套编解码规则莫尔斯码是一个原理。

### 3 内码和外码的含义？

1、内码是指计算机汉字系统中使用的二进制字符编码，是沟通输入、输出与系统平台之间的交换码，通过内码可以达到通用和高效率传输文本的目的。如ASCII。

2、外码是相对于内码而言的辞汇。在计算机科学及相关领域中，外码指的是“外在的‘经过学习之后，可直接了解的编码形式（例如：文字或语音符号）’”。

### 4 简述GBK与GB2312的区别？

1、收录不同：GB2312标准共收录6763个汉字，其中一级汉字3755个，二级汉字3008个；GBK共收入21886个汉字和图形符号。

2、表示不同：GB2312对任意一个图形字符都采用两个字节表示，并对所收汉字进行了“分区”处理，每区含有94个汉字／符号，分别对应第一字节和第二字节。GBK采用双字节表示，总体编码范围为8140-FEFE之间，首字节在81-FE之间，尾字节在40-FE之间。

3、处理功能不同：对于人名、古汉语等方面出现的罕用字，GB2312不能处理，这导致了后来GBK 及GB18030 汉字字符集的出现。

### 5 UTF-8 UTF-16 UTF-32的区别？

结论:**UTF-8、UTF-16、UTF-32 都是 Unicode 的一种实现,其中UTF-32直接将Unicode的编号进行编码的,而UTF-8、UTF-16都是一种变长的编码样子**,

**Unicode 本身只规定了每个字符的数字编号是多少，并没有规定这个编号如何存储,UTF-8、UTF-16、UTF-32规定每个字符如何存储 **

链接:https://blog.csdn.net/hongsong673150343/article/details/88584753

**ASCII码**

我们都知道，在计算机的世界里，信息的表示方式**只有 0 和 1**,但是我们人类信息表示的方式却与之大不相同，很多时候是用语言文字、图像、声音等传递信息的。
那么我们怎样将其转化为二进制存储到计算机中，这个过程我们称之为**编码**。更广义地讲就是把信息从一种形式转化为另一种形式的过程。
我们知道一个二进制有两种状态：”0” 状态 和 “1”状态，那么它就可以代表两种不同的东西，我们想赋予它什么含义，就赋予什么含义，**比如说我规定，“0” 代表 “吃过了”, “1”代表 “还没吃”。**
这样，我们就相当于把现实生活中的信息编码成二进制数字了，并且这个例子中是一位二进制数字，那么 2 位二进制数可以代表多少种情况能？对，是四种，2^2,分别是 00、01、10、11，那7种呢？
答案是2^7=128。
我们知道，在计算机中每八个二进制位组成了一个字节（Byte），计算机存储的最小单位就是字节，字节如下图所示 ：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190315212459847.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hvbmdzb25nNjczMTUwMzQz,size_16,color_FFFFFF,t_70)
所以早期人们用 8 位二进制来编码英文字母(最前面的一位是 0)，也就是说，将英文字母和一些常用的字符和这 128 中二进制 0、1 串一一对应起来，比如说 大写字母“A”所对应的二进制位“01000001”，转换为十六进制为 41。
在美国，这 128 是够了，但是其他国家不答应啊，他们的字符和英文是有出入的，比如在法语中在字母上有注音符号，如 é ,这个怎么表示成二进制？
所以各个国家就决定把字节中最前面未使用的那一个位拿来使用，原来的 128 种状态就变成了 256 种状态，比如 é 就被编码成 130（二进制的 **1**0000010）。
为了保持与 ASCII 码的兼容性，一般最高位为 0 时和原来的 ASCII 码相同，最高位为 1 的时候，各个国家自己给后面的位 (1**xxx xxxx**) 赋予他们国家的字符意义。
但是这样一来又有问题出现了，**不同国家对新增的 128 个数字赋予了不同的含义**，比如说 130 在法语中代表了 é,但是在希伯来语中却代表了字母 Gimel（这不是希伯来字母，只是读音翻译成英文的形式）具体的希伯来字母 Gimel 看下图

#### Unicode的出现

Unicode **为世界上所有字符都分配了一个唯一的数字编号**，这个编号范围从 0x000000 到 0x10FFFF (十六进制)，有 110 多万，每个字符都有一个唯一的 Unicode 编号，这个编号一般写成 16 进制，在前面加上 U+。例如：“马”的 Unicode 是U+9A6C。
Unicode 就相当于一张表，建立了字符与编号之间的联系。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190315212927764.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hvbmdzb25nNjczMTUwMzQz,size_16,color_FFFFFF,t_70)
它是一种规定，**Unicode 本身只规定了每个字符的数字编号是多少，并没有规定这个编号如何存储。**
有的人会说了，那我可以直接把 Unicode 编号直接转换成二进制进行存储，是的，你可以，但是这个就需要人为的规定了，而 Unicode 并没有说这样弄，因为除了你这种直接转换成二进制的方案外，还有其他方案，接下来我们会逐一看到。
编号怎么对应到二进制表示呢？有多种方案：主要有 UTF-8，UTF-16，UTF-32。

#### 1、UTF-32

这个就是字符所对应编号的整数二进制形式，四个字节。这个就是直接转换。 比如马的 Unicode 为：U+9A6C，那么直接转化为二进制，它的表示就为：1001 1010 0110 1100。
这里需要说明的是，转换成二进制后计算机存储的问题，我们知道，**计算机在存储器中排列字节有两种方式：大端法和小端法，\**大端法就是将高位字节放到低地址处，比如 0x1234, 计算机用两个字节存储，一个是\**高位字节 0x12**,一个是**低位字节 0x34**，它的存储方式为下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190315213159937.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hvbmdzb25nNjczMTUwMzQz,size_16,color_FFFFFF,t_70)
UTF-32 用四个字节表示，处理单元为四个字节（一次拿到四个字节进行处理），如果不分大小端的话，那么就会出现解读错误，比如我们一次要处理四个字节 12 34 56 78，这四个字节是表示 0x12 34 56 78 还是表示 0x78 56 34 12？不同的解释最终表示的值不一样。
我们可以根据他们高低字节的存储位置来判断他们所代表的含义，所以在编码方式中有 UTF-32BE 和 UTF-32LE，分别对应大端和小端，来正确地解释多个字节（这里是四个字节）的含义。

#### 2、UTF-16

UTF-16 使用变长字节表示
① 对于编号在 U+0000 到 U+FFFF 的字符（常用字符集），直接用两个字节表示。
② 编号在 U+10000 到 U+10FFFF 之间的字符，需要用四个字节表示。
同样，UTF-16 也有字节的顺序问题（大小端），所以就有 UTF-16BE 表示大端，UTF-16LE 表示小端。

#### 3、UTF-8

UTF-8 就是使用变长字节表示,顾名思义，就是使用的字节数可变，这个变化是根据 Unicode 编号的大小有关，编号小的使用的字节就少，编号大的使用的字节就多。使用的字节个数从 1 到 4 个不等。
UTF-8 的编码规则是：

① 对于**单字节**的符号，字节的第一位设为 0，后面的7位为这个符号的 Unicode 码，因此对于英文字母，UTF-8 编码和 ASCII 码是相同的。

② 对于**n字节**的符号 **（n>1）**,第一个字节的前 n 位都设为 1，第 n+1 位设为 0，后面字节的前两位一律设为 10，剩下的没有提及的二进制位，全部为这个符号的 Unicode 码 。

举个例子：比如说一个字符的 Unicode 编码是 130，显然按照 UTF-8 的规则一个字节是表示不了它（因为如果是一个字节的话**前面的一位必须是 0**），所以需要两个字节(n = 2)。
根据规则，第一个字节的前 2 位都设为 1，第 3(2+1) 位设为 0，则第一个字节为：110X XXXX，后面字节的前两位一律设为 10，后面只剩下一个字节，所以后面的字节为：10XX XXXX。
所以它的格式为 110XXXXX 10XXXXXX 。

下面我们来具体看看具体的 **Unicode 编号范围**与**对应的 UTF-8 二进制格式**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190315213706969.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hvbmdzb25nNjczMTUwMzQz,size_16,color_FFFFFF,t_70)
那么对于一个具体的 Unicode 编号，具体怎么进行 UTF-8 的编码呢？
首先找到该 Unicode 编号所在的编号范围，进而可以找到与之对应的二进制格式，然后将该 Unicode 编号转化为二进制数（去掉高位的 0），最后将该二进制数从右向左依次填入二进制格式的 X 中，如果还有 X 未填，则设为 0 。
比如：“马”的 Unicode 编号是：0x9A6C，整数编号是 39532，对应第三个范围（2048 - 65535），其格式为：1110XXXX 10XXXXXX 10XXXXXX，39532 对应的二进制是 1001 1010 0110 1100，将二进制填入进去就为：
11101001 10101001 10101100 。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190315213903295.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hvbmdzb25nNjczMTUwMzQz,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190315213910329.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hvbmdzb25nNjczMTUwMzQz,size_16,color_FFFFFF,t_70)
由于 UTF-8 的处理单元为一个字节（也就是一次处理一个字节），所以处理器在处理的时候就不需要考虑这一个字节的存储是在高位还是在低位，直接拿到这个字节进行处理就行了，因为大小端是针对大于一个字节的数的存储问题而言的。

**综上所述，UTF-8、UTF-16、UTF-32 都是 Unicode 的一种实现。**

### 6 什么叫码点,单元代码?

在设计Java时，当时的Unicode才发布1.0版本，字符连65536代码值一半都不到，为了方便后面增加，Java使用了16位的Unicode字符集。但是没想到，随着计算机的普及，各国计算机的发展，16位也放不下人类的集体文化财富。

1. **码点是指一个编码表中的某个字符对应的代码值**。Unicode的码点分为17个代码级别，**第一个级别是基本的多语言级别，码点从U+0000——U+FFFF**，其余的**16个级别从U+10000——U+10FFFF**，其中包括一些辅助字符。
2. 基本的多语言级别，**每个字符用16位表示代码单元**，而辅助字符采用连续的一对连续代码单元进行编码





## 4 String类

### 1 从源码的层面讲讲String类的好处?

### 2 **String 类设计成不可变的原因及好处？**

其实好处就是原因，String 设计成不可变，主要是从性能和安全两方面考虑。

**1、常量池的需要**

这个方面很好理解，Java 中的字符串常量池的存在就是为了性能优化。

字符串常量池（String pool）是 Java 堆内存中一个特殊的存储区域，当创建一个 String 对象时，假如此字符串已经存在于常量池中，则不会创建新的对象，而是直接引用已经存在的对象。这样做能够减少 JVM 的内存开销，提高效率。

```javascript
String s1 = "abc";
String s2 = "abc";
```

比如引用 s1和 s2 都是指向常量池的同一个对象 "abc"，如果 String 是可变类，引用 s1 对 String 对象的修改，会直接导致引用 s2 获取错误的值。

![image-20210810221341994](D:\1书本笔记\java实战项目\image-20210810221341994.png)

所以，如果字符串是可变的，那么常量池就没有存在的意义了。

**2、hashcode 缓存的需要**

因为字符串不可变，所以在它创建的时候 hashcode 就被缓存了，不需要重新计算。这就使得字符串很适合作为 HashMap 中的 key，效率大大提高。

**3、多线程安全**

多线程中，可变对象的值很可能被其他线程改变，造成不可预期的结果。而不可变的 String 可以自由在多个线程之间共享，不需要同步处理。

### **3 String 类是如何实现不可变的？**

因为存储数据的char数组是使用final进行修饰的，所以不可变。

**1、私有成员变量**

String 的内部很简单，有两个私有成员变量

```javascript
/** The value is used for character storage. */
private final char value[];

/** Cache the hash code for the string */
private int hash; // Default to 0
```

而并没有对外提供可以修改这两个属性的方法。

**2、Public 的方法都是复制一份数据**

String 有很多 public 方法，每个方法都将创建新的 String 对象，比如 substring 方法：

```javascript
public String substring(int beginIndex) {
    if (beginIndex < 0) {
        throw new StringIndexOutOfBoundsException(beginIndex);
    }
    int subLen = value.length - beginIndex;
    if (subLen < 0) {
        throw new StringIndexOutOfBoundsException(subLen);
    }
    return (beginIndex == 0) ? this : new String(value, beginIndex, subLen);
}
```

**3、String 是 final 的**

String 被 final 修饰，因此我们不可以继承 String，因此就不能通过继承来重写一些方法。

```javascript
public final class String implements java.io.Serializable, Comparable<String>, CharSequence {
}
```

**4、构造函数深拷贝**

当传入可变数组 value[] 时，进行 copy 而不是直接将 value[] 复制给内部变量。

```javascript
public String(char value[]) {
    this.value = Arrays.copyOf(value, value.length);
}
```

从 String 类的设计方式，我们可以总结出实现不可变类的方法：

- 将 class 自身声明为 final，这样别人就不能通过扩展来绕过限制了。
- 将所有成员变量定义为 private 和 final，并且不要实现 setter 方法。
- 通过构造对象时，成员变量使用深拷贝来初始化，而不是直接赋值，这是一种防御措施，因为你无法确定输入对象不被其他人修改。
- 如果确实需要 getter 方法，或者其他可能返回内部状态的方法，使用 copy-on-write 原则，创建私有的 copy。

### 4 String，StringBuilder，StringBuffer的区别是啥？

- 从可变性来讲String的是不可变的，StringBuilder，StringBuffer的长度是可变的。
- 从运行速度上来讲StringBuilder > StringBuffer > String。
- 从线程安全上来StringBuilder是线程不安全的，而StringBuffer是线程安全的。

  所以 String：适用于少量的字符串操作的情况，StringBuilder：适用于单线程下在字符缓冲区进行大量操作的情况，StringBuffer：适用多线程下在字符缓冲区进行大量操作的情况。

### 5 String是不可变，但是下面的代码运行完，却发生变化了，这是为啥呢？

![image-20210810222948976](D:\1书本笔记\java实战项目\image-20210810222948976.png)

![image-20210810223017233](D:\1书本笔记\java实战项目\image-20210810223017233.png)

我们可以发现，在使用`+` 进行拼接的时候，实际上jvm是初始化了一个`StringBuilder`进行拼接的。相当于编译后的代码如下：

![image-20210810223040352](D:\1书本笔记\java实战项目\image-20210810223040352.png)

![image-20210810223154634](D:\1书本笔记\java实战项目\image-20210810223154634.png)

**面试官：为什么String Buffer是线程安全的？**

**小宅**：这是因为在`StringBuffer`类内，常用的方法都使用了`synchronized`  进行同步所以是线程安全的，然而`StringBuilder`并没有。这也就是运行速度`StringBuilder` > `StringBuffer`的原因了。

### 6 String的equals方法如何实现?

String重写了equals方法.

![image-20210810223853010](D:\1书本笔记\java实战项目\image-20210810223853010.png)

- 1.先判断两个对象的地址是否相等
- 2.再判断是否是String类型
- 3.如果都是String类型，就先比较长度是否相等，然后在比较值

### 7 简述对字符串常量池的认识?

字符串的分配和其他的对象分配一样，需要耗费高昂的时间和空间为代价，如果需要大量频繁的创建字符串，会极大程度地影响程序的性能，因此 JVM 为了提高性能和减少内存开销引入了字符串常量池（Constant Pool Table）的概念。

**字符串常量池相当于给字符串开辟一个常量池空间类似于缓存区**，对于直接赋值的字符串（String s="xxx"）来说，在每次创建字符串时优先使用已经存在字符串常量池的字符串，如果字符串常量池没有相关的字符串，会先在字符串常量池中创建该字符串，然后将引用地址返回变量，如下图所示：

![image-20210810225530038](D:\1书本笔记\java实战项目\image-20210810225530038.png)
以上说法可以通过如下代码进行证明：

```
public class StringExample {
    public static void main(String[] args) {
        String s1 = "Java";
        String s2 = "Java";
        System.out.println(s1 == s2);
    }
}
```

以上程序的执行结果为：`true`，说明变量 s1 和变量 s2 指向的是同一个地址。

**在这里我们顺便说一下字符串常量池的再不同 JDK 版本的变化**。

从**JDK 1.7 之后把永生代换成的元空间，把字符串常量池从方法区移到了 Java 堆上**。

JDK 1.7 内存布局如下图所示：

![JDK 1.7 内存布局.png](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/4/17/17185b71e294b6bb~tplv-t2oaga2asx-watermark.awebp)

JDK 1.8 内存布局如下图所示：

![JDK 1.8 内存布局.png](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/4/17/17185b71e2e144fc~tplv-t2oaga2asx-watermark.awebp)

**JDK 1.8 与 JDK 1.7 最大的区别是 JDK 1.8 将永久代取消，并设立了元空间**。官方给的说明是**由于永久代内存经常不够用或发生内存泄露**，会爆出 java.lang.OutOfMemoryError: PermGen 的异常，所以把将永久区废弃而改用元空间了，改为了使用本地内存空间，

### 8  new 字符串创建了几个对象的问题?

答案:   **创建 1 个或者 2 个对象**。

![image-20210810230202526](D:\1书本笔记\java实战项目\image-20210810230202526.png)

**在编译期 new 方式创建的字符串就会被放入到编译期的字符串常量池中**，也就是说 new String  的方式会首先去判断字符串常量池，如果没有就会新建字符串那么就会**创建 2 个对象**，如果已经存在就**只会在堆中创建一个对象**指向字符串常量池中的字符串。

![image-20210810232742505](D:\1书本笔记\java实战项目\image-20210810232742505.png)

![image-20210810232751811](D:\1书本笔记\java实战项目\image-20210810232751811.png)

从图中可以看出 s1 和 s2 的引用一定是相同的，而 s3 和 s4 的引用是不同的，对应的程序代码如下

![image-20210810232814046](D:\1书本笔记\java实战项目\image-20210810232814046.png)

### 9 业务中如何优化字符串的拼接功能?

总 节:

1. Java中字符串拼接不要直接使用`+`拼接。
2. 使用StringBuilder或者StringBuffer时，尽可能准确地估算capacity，并在构造时指定，避免内存浪费和频繁的扩容及复制。
3. 在没有线程安全问题时使用`StringBuilder`， 否则使用`StringBuffer`。
4. 两个字符串拼接直接调用`String.concat`性能最好。

**1 直接使用"+"号拼接?**

+号拼接的时候会调用底层的StringBuilder方法,进行append.

![image-20210810235421593](D:\1书本笔记\java实战项目\image-20210810235421593.png)

​		从反编译的结果来看，实际上对字符串使用`+`操作符进行拼接，**编译器会在编译阶段把代码优化成使用`StringBuilder`类，并调用`append`方法进行字符串拼接，最后调用`toString`方法**，`StringBuilder`在扩容时把容量增大到`当前容量的两倍+2`，这是很可怕的，如果在构造的时候没有指定容量，那么很有可能在扩容之后占用了浪费大量的内存空间。其次扩容后还调用了`Arrays.copyOf`方法，这个方法把扩容前的数据复制到扩容后的空间内，这样做的原因是：`StringBuilder`内部使用`char数组`存放数据，java的数组是不可扩容的，所以只能重新申请一片内存空间，并把已有的数据复制到新的空间去，这里它最终调用了`System.arraycopy`方法来复制，这是一个native方法，底层直接操作内存，所以比我们用循环来复制要块的多，即便如此，大量申请内存空间和复制数据带来的影响也不可忽视。

![image-20210811075800465](D:\1书本笔记\java实战项目\image-20210811075800465.png)

一眼就能看出`创建了太多的StringBuilder对象`，而且在每次循环过后str越来越大，导致每次申请的内存空间越来越大，并且当str长度大于16时，每次都要扩容两次！而实际上`toString`方法在创建`String`对象时，调用了`Arrays.copyOfRange`方法来复制数据，此时相当于每执行一次，扩容了两次，复制了3次数据，这样的代价是相当高的。



2 用StringBuilder或则StringBuilder进行拼接.

![image-20210811080635595](D:\1书本笔记\java实战项目\image-20210811080635595.png)



3 使用String.concat拼接

![image-20210811080527989](D:\1书本笔记\java实战项目\image-20210811080527989.png)

![image-20210811080536627](D:\1书本笔记\java实战项目\image-20210811080536627.png)

#  **关键字 修饰符 与特殊运算符**

### 1 简述对static关键字的理解?

在类中，用static声明的成员变量为静态成员变量，也成为类变量。**类变量的生命周期和类相同，在整个应用程序执行期间都有效**。

**这里要强调一下：**

- static修饰的成员变量和方法，从属于类
- 普通变量和方法从属于对象
- 静态方法不能调用非静态成员，编译会报错

static关键字的用途
一句话描述就是：方便在没有创建对象的情况下进行调用(方法/变量)。

显然，被static关键字修饰的方法或者变量不需要依赖于对象来进行访问，只要类被加载了，就可以通过类名去进行访问。

static可以用来修饰类的成员方法、类的成员变量，另外也可以编写static代码块来优化程序性能

#### 1 static方法

static方法也成为静态方法，由于静态方法不依赖于任何对象就可以直接访问，**因此对于静态方法来说，是没有this的，因为不依附于任何对象**，既然都没有对象，就谈不上this了，并且由于此特性，在静态方法中不能访问类的非静态成员变量和非静态方法，因为非静态成员变量和非静态方法都必须依赖于具体的对象才能被调用。
**虽然在静态方法中不能访问非静态成员方法和非静态成员变量，但是在非静态成员方法中是可以访问静态成员方法和静态成员变量。**

**static方法是属于类的，非实例对象，在JVM加载类时，就已经存在内存中，不会被虚拟机GC回收掉，这样内存负荷会很大，但是非static方法会在运行完毕后被虚拟机GC掉，减轻内存压力**

#### 2 static变量

static变量也称为静态变量，静态变量和非静态变量的区别：

- 静态变量被所有对象共享，在内存中只有一个副本，在类初次加载的时候才会初始化
- 非静态变量是对象所拥有的，在创建对象的时候被初始化，存在多个副本，各个对象拥有的副本互不影响

**static成员变量初始化顺序按照定义的顺序来进行初始化**

#### 3 static块

构造方法用于对象的初始化。静态初始化块，用于类的初始化操作。

在静态初始化块中不能直接访问非staic成员。

**静态初始化块可以置于类中的任何地方，类中可以有多个静态初始化块。**
**在类初次被加载时，会按照静态初始化块的顺序来执行每个块，并且只会执行一次。**

#### 4 static关键字会改变类中成员的访问权限吗？

Java中的static关键字不会影响到变量或者方法的作用域。在Java中能够影响到访问权限的只有private、public、protected（包括包访问权限）这几个关键字

#### 5  能通过this访问静态成员变量吗？

![image-20210811085245306](D:\1书本笔记\java实战项目\image-20210811085245306.png)

#### 6 static能作用于局部变量么？

static是不允许用来修饰局部变量。不要问为什么，这是Java语法的规定

#### 7 下面这段代码的输出结果是什么？

```java
public class Test {
    Person person = new Person("Test");
    static{
        System.out.println("test static");
    }

    public Test() {
        System.out.println("test constructor");
    }

    public static void main(String[] args) {
        new MyClass();
    }
}

class Person{
    static{
        System.out.println("person static");
    }
    public Person(String str) {
        System.out.println("person "+str);
    }
}


class MyClass extends Test {
    Person person = new Person("MyClass");
    static{
        System.out.println("myclass static");
    }

    public MyClass() {
        System.out.println("myclass constructor");
    }
}

output:
test static
myclass static
person static
person Test
test constructor
person MyClass
myclass constructor
```

为什么输出结果是这样的？我们来分析下这段代码的执行过程：

找到main方法入口，main方法是程序入口，但在执行main方法之前，要先加载Test类

加载Test类的时候，发现Test类有static块，而是先执行static块，输出test static结果

然后执行new MyClass(),执行此代码之前，先加载MyClass类，发现MyClass类继承Test类，而是要先加载Test类，Test类之前已加载

加载MyClass类，发现MyClass类有static块，而是先执行static块，输出myclass static结果

然后调用MyClass类的构造器生成对象，在生成对象前，需要先初始化父类Test的成员变量，而是执行Person person = new Person("Test")代码，发现Person类没有加载

加载Person类，发现Person类有static块，而是先执行static块，输出person static结果

接着执行Person构造器，输出person Test结果

然后调用父类Test构造器，输出test constructor结果，这样就完成了父类Test的初始化了

再初始化MyClass类成员变量，执行Person构造器，输出person MyClass结果

最后调用MyClass类构造器，输出myclass constructor结果，这样就完成了MyClass类的初始化了










### 2 简述父子类的初始化顺序(静态变量,静态语句块,实例变量,普通语句块,构造函数)

如果类还没有被加载： 
**1、先执行父类的静态代码块和静态变量初始化，并且静态代码块和静态变量的执行顺序只跟代码中出现的顺序有关。** 
**2、执行子类的静态代码块和静态变量初始化。** 
**3、执行父类的实例变量初始化** 
**4、执行父类的构造函数** 
**5、执行子类的实例变量初始化** 
**6、执行子类的构造函数** 

如果类已经被加载： 
**则静态代码块和静态变量就不用重复执行，再创建类对象时，只执行与实例相关的变量初始化和构造方法。**

### 3 java内部类详解

泛意义上的内部类一般来说包括这四种：成员内部类、局部内部类、匿名内部类和静态内部类。下面就先来了解一下这四种内部类的用法。

#### 1  成员内部类

**成员内部类是最普通的内部类，它的定义为位于另一个类的内部**，形如下面的形式：

![image-20210811104915091](D:\1书本笔记\java实战项目\image-20210811104915091.png)

　　这样看起来，类Draw像是类Circle的一个成员，Circle称为外部类。成员内部类可以无条件访问外部类的所有成员属性和成员方法（包括private成员和静态成员）。

![image-20210811104929916](D:\1书本笔记\java实战项目\image-20210811104929916.png)



　　不过要注意的是，当成员内部类拥有和外部类同名的成员变量或者方法时，会发生隐藏现象，即默认情况下访问的是成员内部类的成员。如果要访问外部类的同名成员，需要以下面的形式进行访问：

![image-20210811104943338](D:\1书本笔记\java实战项目\image-20210811104943338.png)

　　虽然成员内部类可以无条件地访问外部类的成员，而外部类想访问成员内部类的成员却不是这么随心所欲了。在外部类中如果要访问成员内部类的成员，必须先创建一个成员内部类的对象，再通过指向这个对象的引用来访问：

![image-20210811105003146](D:\1书本笔记\java实战项目\image-20210811105003146.png)

　　成员内部类是依附外部类而存在的，也就是说，如果要创建成员内部类的对象，前提是必须存在一个外部类的对象。创建成员内部类对象的一般方式如下：

![image-20210811105031948](D:\1书本笔记\java实战项目\image-20210811105031948.png)

　　内部类可以拥有private访问权限、protected访问权限、public访问权限及包访问权限。比如上面的例子，如果成员内部类Inner用private修饰，则只能在外部类的内部访问，如果用public修饰，则任何地方都能访问；如果用protected修饰，则只能在同一个包下或者继承外部类的情况下访问；如果是默认访问权限，则只能在同一个包下访问。这一点和外部类有一点不一样，外部类只能被public和包访问两种权限修饰。我个人是这么理解的，由于成员内部类看起来像是外部类的一个成员，所以可以像类的成员一样拥有多种权限修饰。

##### 1 为什么成员内部类可以无条件访问外部类的成员？

​			编译器会默认为成员内部类添加了一个指向外部类对象的引用，那么这个引用是如何赋初值的呢？我们在定义的内部类的构造器是无参构造器，编译器还是会默认添加一个参数，该参数的类型为指向外部类对象的一个引用，所以成员内部类中的Outter this&0 指针便指向了外部类对象，因此可以在成员内部类中随意访问外部类的成员。

#### 2 局部内部类

**局部内部类是定义在一个方法或者一个作用域里面的类，它和成员内部类的区别在于局部内部类的访问仅限于方法内或者该作用域内。**

![image-20210811105226931](D:\1书本笔记\java实战项目\image-20210811105226931.png)

　　**注意，局部内部类就像是方法里面的一个局部变量一样，是不能有public、protected、private以及static修饰符的。**

#### 3 匿名内部类

**匿名内部类也是不能有访问修饰符和static修饰符的**

​		匿名内部类可以使你的代码更加简洁，你可以在定义一个类的同时对其进行实例化。它与局部类很相似，不同的是它没有类名，如果某个局部类你只需要用一次，那么你就可以使用匿名内部类。

​		匿名内部类可以使你的代码更加简洁，你可以在定义一个类的同时对其进行实例化。它与局部类很相似，不同的是它没有类名，如果某个局部类你只需要用一次，那么你就可以使用匿名内部类。

##### 1 匿名内部类的语法

1. **实现接口的匿名类**

2. **匿名子类（继承父类）**

![image-20210811165927710](D:\1书本笔记\java实战项目\image-20210811165927710.png)

**案例二，匿名子类（继承父类）：**

public class AnimalTest {

    private final String ANIMAL = "动物";
    
    public void accessTest() {
        System.out.println("匿名内部类访问其外部类方法");
    }
    
    class Animal {
        private String name;
    
        public Animal(String name) {
            this.name = name;
        }
    
        public void printAnimalName() {
            System.out.println(bird.name);
        }
    }
    
    // 鸟类，匿名子类，继承自Animal类，可以覆写父类方法
    Animal bird = new Animal("布谷鸟") {
    
        @Override
        public void printAnimalName() {
            accessTest();   　　　　　　　　// 访问外部类成员
            System.out.println(ANIMAL);  // 访问外部类final修饰的变量
            super.printAnimalName();
        }
    };
    
    public void print() {
        bird.printAnimalName();
    }
    
    public static void main(String[] args) {
    
        AnimalTest animalTest = new AnimalTest();
        animalTest.print();
    }
}

运行结果：

```
运行结果：
匿名内部类访问其外部类方法
动物
布谷鸟
```

从以上两个实例中可知，匿名类表达式包含以下内部分：

1. 操作符：new；
2. 一个要实现的**接口或要继承的类**，案例一中的匿名类实现了HellowWorld接口，**案例二中的匿名内部类继承了Animal父类**；
3. 一对括号，如果是**匿名子类**，与实例化普通类的语法类似，**如果有构造参数，要带上构造参数**；**如果是实现一个接口，只需要一对空括号即可**；
4. 一段被"{}"括起来类声明主体；
5. 末尾的";"号（因为匿名类的声明是一个表达式，是语句的一部分，因此要以分号结尾）。

##### 2 访问作用域内的局部变量、定义和访问匿名内部类成员

匿名内部类与局部类对作用域内的变量拥有相同的的访问权限。

**(1)、匿名内部类可以访问外部内的所有成员；**

**(2)、匿名内部类不能访问外部类未加final修饰的变量（注意：JDK1.8即使没有用final修饰也可以访问）；**

**(3)、属性屏蔽，与内嵌类相同，匿名内部类定义的类型（如变量）会屏蔽其作用域范围内的其他同名类型（变量）**

 ***\*案例一，内嵌类的属性屏蔽：\****

public class ShadowTest {

    public int x = 0;
    
    class FirstLevel {
    
        public int x = 1;
    
        void methodInFirstLevel(int x) {
            System.out.println("x = " + x);
            System.out.println("this.x = " + this.x);
            System.out.println("ShadowTest.this.x = " + ShadowTest.this.x);
        }
    }
    
    public static void main(String... args) {
        ShadowTest st = new ShadowTest();
        ShadowTest.FirstLevel fl = st.new FirstLevel();
        fl.methodInFirstLevel(23);
    }
}

输出结果为：

```
x = 23
this.x = 1
ShadowTest.this.x = 0
```

这个实例中有三个变量x：1、ShadowTest类的成员变量；2、内部类FirstLevel的成员变量；3、内部类方法methodInFirstLevel的参数。

methodInFirstLevel的参数x屏蔽了内部类FirstLevel的成员变量，因此，在该方法内部使用x时实际上是使用的是参数x，可以使用this关键字来指定引用是成员变量x：

 1 System.out.println("this.x = " + this.x); 

利用类名来引用其成员变量拥有最高的优先级，不会被其他同名变量屏蔽，如：

 1 System.out.println("ShadowTest.this.x = " + ShadowTest.this.x); 

**(4)、 匿名内部类中不能定义静态属性、方法**

public class ShadowTest {
    public int x = 0;

    interface FirstLevel {
     void methodInFirstLevel(int x);
    }
    
    FirstLevel firstLevel =  new FirstLevel() {
    
        public int x = 1;
    
        public static String str = "Hello World";   // 编译报错
    
        public static void aa() {        // 编译报错
        }
    
        public static final String finalStr = "Hello World";  // 正常
    
        public void extraMethod() {  // 正常
            // do something
        }
    };
}

**(5)、匿名内部类可以有常量属性（final修饰的属性）；**

**(6)、匿名内部内中可以定义属性，如上面代码中的代码:private int x = 1;**

**(7)、匿名内部内中可以可以有额外的方法（父接口、类中没有的方法）;**

**(8)、匿名内部内中可以定义内部类；**

**(9)、匿名内部内中可以对其他类进行实例化**



##### 3  为什么局部内部类和匿名内部类只能访问局部final变量？

![image-20210811162945217](D:\1书本笔记\java实战项目\image-20210811162945217.png)

像这样。

　这段代码会被编译成两个class文件：Test.class和Test1.class。默认情况下，编译器会为匿名内部类和局部内部类起名为Outter1.class。默认情况下，编译器会为匿名内部类和局部内部类起名为Outterx.class（x为正整数）。

　　![img](https://images0.cnblogs.com/i/288799/201407/021900556994393.jpg)

　　根据上图可知，test方法中的匿名内部类的名字被起为 Test$1。

　　上段代码中，**如果把变量a和b前面的任一个final去掉**，这段代码都编译不过。我们先考虑这样一个问题：

　　**当test方法执行完毕之后，变量a的生命周期就结束了，而此时Thread对象的生命周期很可能还没有结束，**那么在Thread的run方法中继续访问变量a就变成不可能了，但是又要实现这样的效果，怎么办呢？Java采用了 **复制** 的手段来解决这个问题。将这段代码的字节码反编译可以得到下面的内容：

![img](https://images0.cnblogs.com/i/288799/201407/021939271846598.jpg)

　　我们看到在run方法中有一条指令：

```
bipush 10
```

　　这条指令表示将操作数10压栈，表示使用的是一个本地局部变量。这个过程是在编译期间由编译器默认进行，如果这个变量的值在编译期间可以确定，则编译器默认会在匿名内部类（局部内部类）的常量池中添加一个内容相等的字面量或直接将相应的字节码嵌入到执行字节码中。这样一来，匿名内部类使用的变量是另一个局部变量，只不过值和方法中局部变量的值相等，因此和方法中的局部变量完全独立开。

　　下面再看一个例子：

![image-20210811163052094](D:\1书本笔记\java实战项目\image-20210811163052094.png)

　　反编译得到：

![img](https://images0.cnblogs.com/i/288799/201407/021950384493440.jpg)

　　我们看到匿名内部类Test$1的构造器含有两个参数，一个是指向外部类对象的引用，一个是int型变量，很显然，这里是将变量test方法中的形参a以参数的形式传进来对匿名内部类中的拷贝（变量a的拷贝）进行赋值初始化。

　　也就说如果局部变量的值在编译期间就可以确定，则直接在匿名内部里面创建一个拷贝。如果局部变量的值无法在编译期间确定，则通过构造器传参的方式来对拷贝进行初始化赋值。

　　从上面可以看出，**在run方法中访问的变量a根本就不是test方法中的局部变量a**。这样一来就解决**了前面所说的 生命周期不一致的问题**。但是新的问题又来了，**既然在run方法中访问的变量a和test方法中的变量a不是同一个变量**，当在run方法中改变变量a的值的话，**会出现什么情况**？

　　对，**会造成数据不一致性**，这样就达不到原本的意图和要求。为了解决这个问题，**java编译器就限定必须将变量a限制为final变量**，不允许对变量a进行更改（对于引用类型的变量，是不允许指向新的对象），**这样数据不一致性的问题就得以解决了**。

　　到这里，想必大家应该清楚为何 方法中的局部变量和形参都必须用final进行限定了。

#### 4 静态内部类

​		静态内部类也是定义在另一个类里面的类，只不过在类的前面多了一个关键字static。静态内部类是不需要依赖于外部类的，这点和类的静态成员属性有点类似，并且它不能使用外部类的非static成员变量或者方法，这点很好理解，因为在没有外部类的对象的情况下，可以创建静态内部类的对象，如果允许访问外部类的非static成员就会产生矛盾，因为外部类的非static成员必须依附于具体的对象。

​	**从前面可以知道，静态内部类是不依赖于外部类的，也就说可以在不创建外部类对象的情况下创建内部类的对象。另外，静态内部类是不持有指向外部类对象的引用的**

#### 5 内部类的使用场景和好处？

　　为什么在Java中需要内部类？总结一下主要有以下四点：

　　1.每个内部类都能独立的继承一个接口的实现，所以无论外部类是否已经继承了某个(接口的)实现，对于内部类都没有影响。内部类使得多继承的解决方案变得完整，

　　2.方便将存在一定逻辑关系的类组织在一起，又可以对外界隐藏。

　　3.方便编写事件驱动程序

　　4.方便编写线程代码

　　个人觉得第一点是最重要的原因之一，内部类的存在使得Java的多继承机制变得更加完



### 4 final关键字的作用

1. final成员变量必须在声明的时候初始化或者在构造器中初始化，否则就会报编译错误。final变量一旦被初始化后不能再次赋值，**其中类常量必须在声明时初始化，final成员常量可以在构造函数初始化。**。

2. 本地变量必须在声明时赋值。 因为没有初始化的过程

3. 在匿名类中所有变量都必须是final变量。

4. final方法不能被重写, final类不能被继承

5. 接口中声明的所有变量本身是final的。类似于匿名类

6. final和abstract这两个关键字是反相关的，final类就不可能是abstract的。

7. final方法在编译阶段绑定，称为静态绑定(static binding)。

8. 将类、方法、变量声明为final能够提高性能，这样JVM就有机会进行估计，然后优化。




final的作用随着所修饰的类型而不同

       1、final修饰类中的属性或者变量
    
              无论属性是基本类型还是引用类型，final所起的作用都是变量里面存放的“值”不能变。
    
              这个值，对于基本类型来说，变量里面放的就是实实在在的值，如1，“abc”等。
    
              而引用类型变量里面放的是个地址，所以用final修饰引用类型变量指的是它里面的地址不能变，并不是说这个地址所指向的对象或数组的内容不可以变，这个一定要注意。
    
              例如：类中有一个属性是final Person p=new Person("name")； 那么你不能对p进行重新赋值，但是可以改变p里面属性的值，p.setName('newName');
    
              final修饰属性，声明变量时可以不赋值，而且一旦赋值就不能被修改了。对final属性可以在三个地方赋值：声明时、初始化块中、构造方法中。总之一定要赋值。      
    
      2、final修饰类中的方法
    
             作用：可以被继承，但继承后不能被重写。
    
      3、final修饰类
    
             作用：类不可以被继承。
**从java内存模型中理解final关键字**

java内存模型对final域遵守如下两个重拍序规则

1. 初次读一个包含final域的对象的引用和随后初次写这个final域，不能重拍序。
2. 在构造函数内对final域写入，随后将构造函数的引用赋值给一个引用变量，操作不能重排序。

**以上两个规则就限制了final域的初始化必须在构造函数内，不能重拍序到构造函数之外，普通变量可以。**

具体的操作是

1. java内存模型在final域写入和构造函数返回之前，插入一个StoreStore内存屏障，静止处理器将final域重拍序到构造函数之外。
2. java内存模型在初次读final域的对象和读对象内final域之间插入一个LoadLoad内存屏障。



### 5 简述使用final关键字的好处？

final方法的好处:

1. 提高了性能，JVM在常量池中会缓存final变量
2. final变量在多线程中并发安全，无需额外的同步开销
3. final方法是静态编译的，提高了调用速度
4. **final类创建的对象是只可读的，在多线程可以安全共享**

### 6 static和final所修饰的变量在JVM中的位置？

final所修饰的变量在JVM的运行时常量池里面

static所修饰的变量在方法区里面（JDK1.8又叫元空间）

我们写的每一个Java类被编译后，就会形成一份class文件；class文件中除了包含类的版本、字段、方法、接口等描述信息外，还有一项信息就是常量池(constant pool table)，用于存放编译器生成的各种 字面量 (Literal)和 符号引用 (Symbolic References)，每个class文件都有一个class常量池

1、从内存角度理解static与final关键字：

![image-20210811191750500](D:\1书本笔记\java实战项目\image-20210811191750500.png)

从该文章可以知道，被final修饰的变量存储在运行时常量池中。

2、java中静态变量在内存中的位置
![image-20210811191803580](D:\1书本笔记\java实战项目\image-20210811191803580.png)

方法区:
1.又叫静态区，跟堆一样，被所有的线程共享。方法区包含所有的class和static变量。
2.方法区中包含的都是在整个程序中永远唯一的元素，如class，static变量。

### 7 简述native关键字的含义？

java是跨平台的语言，既然是跨了平台，所付出的代价就是牺牲一些对底层的控制，而java要实现对底层的控制，就需要一些其他语言的帮助，这个就是native的作用了

------

Java不是完美的，Java的不足除了体现在运行速度上要比传统的C++慢许多之外，Java无法直接访问到操作系统底层（如系统硬件等)，为此Java使用native方法来扩展Java程序的功能。
　　可以将native方法比作Java程序同Ｃ程序的接口，其实现步骤：
　　１、在Java中声明native()方法，然后编译；
　　２、用javah产生一个.h文件；
　　３、写一个.cpp文件实现native导出方法，其中需要包含第二步产生的.h文件（注意其中又包含了JDK带的jni.h文件）；
　　４、将第三步的.cpp文件编译成动态链接库文件；
　　５、在Java中用System.loadLibrary()方法加载第四步产生的动态链接库文件，这个native()方法就可以在Java中被访问了。

### 8 简述运算符instanceof的含义以及应用场景?

判断 该运算符前面引用类型变量指向的对象是否是后面类，或者其子类、接口实现类创建的对象。如果是则返回true，否则返回false，



### 9 this和super关键字的用法？

#### 1 **this**

**this 关键字只能在方法内部使用，表示对`调用方法的那个对象`的引用。**

1. ### 调用成员变量

在一个类的方法内部，如果我们想调用其成员变量，不用 this，我们会怎么做？

```java
public class ThisTest {

    private String name = "xiaoming";

    public String getName() {
        return name;
    }

    public void setName(String name) {
        name = name;
    }
}
复制代码
```

看上面的代码，我们在 `ThisTest` 类中创建了一个 `name` 属性，然后创建了一个 `setName` 方法，注意这个方法的形参也是 `String name`，那么我们通过 `name = name` 这样赋值，会改变成员变量 `name` 的属性吗？

```java
public static void main(String[] args) {
    ThisTest thisTest = new ThisTest();
    thisTest.setName("xiaoma");
    System.out.println(thisTest.getName());
}
复制代码
```

打印结果是 `xiaoming`，而不是我们重新设置的 `xiaoma`，显然这种方式是不能在方法内部调用到成员变量的。因为形参的名字和成员变量的名字相同，`setName` 方法内部的 `name = name`，根据最近原则，编译器默认是将这两个 `name` 属性都解析为形参 `name`，从而导致我们设值操作和成员变量 `name` 完全没有关系，当然设置不了。

> 解决办法就是使用 this 关键字。我们将 setName 方法修改如下：

```java
public void setName(String name) {
    this.name = name;
}
复制代码
```

在调用上面的 main 方法进行赋值，打印的结果就是 `xiaoma`了。

this 表示当前对象，也就是调用该方法的对象，对象`.name` 肯定就是调用的成员变量。

2. **调用构造方法**

构造方法是与类同名的一个方法，构造方法没有返回值，但是也不能用 void 来修饰。在一个类中，必须存在一个构造方法，如果没有，编译器会在编译的时候自动为这个类添加一个无参构造方法。一个类能够存在多个构造方法，调用的时候根据参数来区分。

```java
public class Student {

    private int age;

    private String name;

    public Student() {
        this("小马",50);
    }

    public Student(String name, int age) {
        this.name = name;
        this.age = age;
        System.out.println(name + "今年" + age + "岁了");
    }


    public static void main(String[] args) {
        Student student01 = new Student();
        Student student02 = new Student("小军",45);
    }
}
复制代码
```

通过`this("小马",50)`来调用另外一个构造方法 `Student(String name, int age) `来给成员变量初始化赋值。

输出结果：

```
小马今年50岁了
小军今年45岁了

Process finished with exit code 0
复制代码
```

`注意：`通过 this 来调用构造方法，只能将这条代码放在构造函数的第一行，这是编译器的规定，如下所示：放在第二行会报错。

####  2 super

Java 中的 super 关键字则是表示 `父类对象的引用`。

**1、调用父类的构造方法**

Java中的继承大家都应该了解，子类继承父类，我们是能够用子类的对象调用父类的属性和方法的，我们知道属性和方法只能够通过对象调用，那么我们可以大胆假设一下：**在创建子类对象的同时，也创建了父类的对象，而创建对象是通过调用构造函数实现的，那么我们在创建子类对象的时候，应该会调用父类的构造方法。**

下面我们看这段代码：

```java
public class Teacher {

   public Teacher(){
       System.out.println("我是一名人民教师。");
   }
}

class Student extends Teacher {

    public Student(){
        System.out.println("我是一名学生。");
    }
}
复制代码
```

下面我们创建子类的对象：

```java
public static void main(String[] args) {
    Student s = new Student();
}
复制代码
```

输出结果：

```
我是一名人民教师。
我是一名学生。

Process finished with exit code 0
复制代码
```

通过打印结果看到我们在创建子类对象的时候，首先调用了父类的构造方法，接着调用子类的构造方法，也就是说在创建子类对象的时候，首先创建了父类对象，与前面我们猜想的一致。

> 那么问题又来了：是在什么时候调用的父类构造方法呢？

可以参考Java官方文档：[docs.oracle.com/javase/spec…](https://link.juejin.cn?target=https%3A%2F%2Fdocs.oracle.com%2Fjavase%2Fspecs%2Fjls%2Fse8%2Fhtml%2Fjls-8.html%23d5e14278)

![image-20210729185246748](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/324e36a3f23e4b84b6eb969afd6583ea~tplv-k3u1fbpfcp-watermark.awebp)

红色框内的英文翻译为：**如果声明的类是原始类Object，那么默认的构造函数有一个空的主体。否则，默认构造函数只是简单地调用没有参数的超类构造函数。**

也就是说：**除了顶级类 Object.class 构造函数没有调用父类的构造方法，其余的所有类都默认在构造函数中调用了父类的构造函数（没有显式声明父类的子类其父类是 Object）。**

> 那么是通过什么来调用的呢？我们接着看官方文档：

![image-20210729185503815](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8c66205756c7473daded9769aa5778f7~tplv-k3u1fbpfcp-watermark.awebp)

上面的意思大概就是：**超类构造函数通过 super 关键字调用，并且是以 super 关键字开头。**

所以上面的 `Student`类的构造方法实际上应该是这样的：

```java
class Student extends Teacher {

    public Student(){
        super();//子类通过super调用父类的构造方法
        System.out.println("我是一名学生。");
    }


    public static void main(String[] args) {
        Student s = new Student();
    }
}
复制代码
```

子类默认是通过 super() 调用父类的无参构造方法，如果父类显示声明了一个有参构造方法，而没有声明无参构造方法，实例化子类是会报错的。

![image-20210729185801603](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c9e07df6e0a245e69cac3e8707276cf5~tplv-k3u1fbpfcp-watermark.awebp)

　解决办法就是通过 super 关键字调用父类的有参构造方法：

```java
class Student extends Teacher {

    public Student(){
        super("小马");
        System.out.println("我是一名学生。");
    }


    public static void main(String[] args) {
        Student s = new Student();
    }
}
复制代码
```

**2、调用父类的成员属性**

```java
public class Teacher {

    public String name = "小马";

    public Teacher() {
        System.out.println("我是一名人民教师。");
    }
}

class Student extends Teacher {

    public Student() {
        System.out.println("我是一名学生。");
    }

    public void fatherName() {
        System.out.println("我的父类名字是：" + super.name);//调用父类的属性
    }

    public static void main(String[] args) {
        Student student = new Student();
        student.fatherName();
    }
}
复制代码
```

输出结果：

```
我是一名人民教师。
我是一名学生。
我的父类名字是：小马

Process finished with exit code 0
复制代码
```

**3、调用父类的方法**

```java
public class Teacher {

    public String name;

    public Teacher() {
        System.out.println("我是一名人民教师。");
    }

    public void setName(String name){
        this.name = name;
    }
}

class Student extends Teacher {

    public Student() {
        super();//调用父类的构造方法
        System.out.println("我是一名学生。");
    }

    public void fatherName() {
        super.setName("小军");//调用父类普通方法
        System.out.println("我的父类名字是：" + super.name);//调用父类的属性
    }

    public static void main(String[] args) {
        Student student = new Student();
        student.fatherName();
    }
}
复制代码
```

输出结果：

```
我是一名人民教师。
我是一名学生。
我的父类名字是：小军

Process finished with exit code 0
```

#### 3 this和super异同

- `super(参数)`：**调用基类中的某一个构造函数（应该为构造函数中的第一条语句）。**
- `this(参数)`：**调用本类中另一种形成的构造函数（应该为构造函数中的第一条语句）。**
- `super`:　它引用当前对象的直接父类中的成员（用来访问直接父类中被隐藏的父类中成员数据或函数，基类与派生类中有相同成员定义时如：`super.变量名`、 `super.成员函数据名（实参）` 。
- `this`：它代表当前对象名（在程序中易产生二义性之处，应使用 this 来指明当前对象；如果函数的形参与类中的成员数据同名，这时需用 this 来指明成员变量名）。
- 调用`super()`必须写在子类构造方法的第一行，否则编译不通过。每个子类构造方法的第一条语句，都是隐含地调用 `super()`，如果父类没有这种形式的构造函数，那么在编译的时候就会报错。
- `super()` 和 `this()` 类似,区别是，`super()` 从子类中调用父类的构造方法，`this()` 在同一类内调用其它方法。
- `super()` 和 `this()` 均需放在构造方法内第一行。
- **尽管可以用`this`调用一个构造器，但却不能调用两个。**
- `this` 和 `super` 不能同时出现在一个构造函数里面，因为`this`必然会调用其它的构造函数，其它的构造函数必然也会有 `super` 语句的存在，所以在同一个构造函数里面有相同的语句，就失去了语句的意义，编译器也不会通过。
- `this()` 和 `super()` 都指的是对象，所以，均不可以在 `static` 环境中使用。包括：`static` 变量,`static` 方法，`static 语句块`。
- 从本质上讲，`this` 是一个指向本对象的指针， 然而 `super` 是一个 Java 关键字。



### 10 private、protected、public、default的区别？

**（1）**对于**public**修饰符，它具有最大的访问权限，可以访问任何一个在CLASSPATH下的类、接口、异常等。它往往用于对外的情况，也就是对象或类对外的一种接口的形式。

**（2）**对于**protected**修饰符，它主要的作用就是用来保护子类的。它的含义在于子类可以用它修饰的成员，其他的不可以，它相当于传递给子类的一种继承的东西。

**（3）**对于**default**来说，有点的时候也成为friendly（友员），它是针对本包访问而设计的，任何处于本包下的类、接口、异常等，都可以相互访问，即使是父类没有用protected修饰的成员也可以。

**（4）**对于**private**来说，它的访问权限仅限于类的内部，是一种封装的体现，例如，大多数的成员变量都是修饰符为private的，它们不希望被其他任何外部的类访问。

![image-20210811194235022](D:\1书本笔记\java实战项目\image-20210811194235022.png)

**注意：Java的访问控制是停留在编译层的，也就是它不会在.class文件中留下任何的痕迹，只在编译的时候进行访问控制的检查。其实，通过反射的手段，是可以访问任何包下任何类中的成员，例如，访问类的私有成员也是可能的。**

**区别：**

> **（1）public：可以被所有其他类所访问。** **（2）private：只能被自己访问和修改。** **（3）protected：自身，子类及同一个包中类可以访问。** **（4）default（默认）：同一包中的类可以访问，声明时没有加修饰符，认为是friendly。**



# 面像对象

### 1 简述面向对象的三大特性？

封装
---->减少了大量的冗余代码
---->封装将复杂的功能封装起来，对外开放一个接口，简单调用即可。
将描述事物的数据和操作封装在一起，形成一个类；被封装的数据和操作只有通过提供的公共方法才能被外界访问（封装隐藏了对象的属性和实施细节），私有属性和方法是无法被访问的，表现了封装的隐藏性，增加数据的安全性。

继承–单根性，传递性
---->减少了类的冗余代码
---->让类与类之间产生关系，为多态打下基础
若一个新类继承了原有类的属性和方法，并增加了自己的新属性和新方法，称之为派生类，派生类就继承了原有类；**当子类继承父类的时候**，子类不继承父类的构造函数，但是子类生成对象时(new Student())**默认会先执行父类无参的构造函数**（实例化对象，让子类可以使用父类的成员），**当父类定义有参构造函数时**，无参构造函数就会被干掉，**这时子类会报错**，解决方案是1>**在父类重新定义无参构造函数** 2>**子类的构造函数**：base()，**调用父类的有参构造函数**，函数体中只需初始化特有属性；如果不想执行父类的构造函数，可以通过参数的不同调用父类一个空的构造函数。

多态
----->虚方法 virtual override
----->抽象类 abstract override
----->接口 interface
很重要的一个概念，一个接口，多个方法。通过继承实现的不同对象调用相同的方法，进而有不同的行为，实例如下：abstract-override

### 2 重写和重载的区别？

重写：**发生在子类和父类之间**，当子类继承父类中的方法时，子类中的方法与父类方法的名称，参数个数，参数类型完全一致时，称子类重写了父类的方法。 

重载：一个类中的多个方法的名称相同，参数个数或者参数类型不同，则称为重载方法 

覆盖：子类重新实现了父类的方法

### 3 构造器可以重写么？

**不能被重写，只能重载。**

### 4 简述抽象类、接口的区别？

#### 1 抽象类：

**抽象类**：在Java中被abstract关键字修饰的类称为抽象类，被abstract关键字修饰的方法称为抽象方法，抽象方法只有方法的声明，没有方法体。抽象类的特点：

a、抽象类不能被实例化只能被继承；

b、包含抽象方法的一定是抽象类，但是抽象类不一定含有抽象方法；

c、抽象类中的抽象方法的修饰符只能为public或者protected，默认为public；

d、一个子类继承一个抽象类，则子类必须实现父类抽象方法，否则子类也必须定义为抽象类；

e、抽象类可以包含属性、方法、构造方法，但是构造方法不能用于实例化，主要用途是被子类调用。

#### 2 接口：

Java中接口使用interface关键字修饰，特点为:

a、接口可以包含变量、方法；变量被隐士指定为public static final，方法被隐士指定为public abstract（JDK1.8之前）；

b、接口支持多继承，即一个接口可以extends多个接口，间接的解决了Java中类的单继承问题；

c、一个类可以实现多个接口；

d、JDK1.8中对接口增加了新的特性：（1）、默认方法（default method）：JDK 1.8允许给接口添加非抽象的方法实现，但必须使用default关键字修饰；定义了default的方法可以不被实现子类所实现，但只能被实现子类的对象调用；如果子类实现了多个接口，并且这些接口包含一样的默认方法，则子类必须重写默认方法；（2）、静态方法（static method）：JDK 1.8中允许使用static关键字修饰一个方法，并提供实现，称为接口静态方法。接口静态方法只能通过接口调用（接口名.静态方法名）

#### 3 接口与抽象类的区别

**相同点**

（1）都不能被实例化 （2）接口的实现类或抽象类的子类都只有实现了接口或抽象类中的方法后才能实例化。

**不同点**

（1）**接口只有定义，不能有方法的实现**，java 1.8中可以定义default方法体，而抽象类可以**有定义与实现，方法可在抽象类中实现**。

（2）实现接口的关键字为implements，继承抽象类的关键字为extends。**一个类可以实现多个接口，但一个类只能继承一个抽象类**。所以，使用接口可以间接地实现多重继承。

（3）接口强调特定功能的实现，而抽象类强调所属关系。

（4）**接口成员变量默认为public static final，必须赋初值，不能被修改**；其所有的成员方法都是public、abstract的。**抽象类中成员变量默认default**，可在子类中被重新定义，也可被重新赋值；抽象方法被abstract修饰，不能被private、static、synchronized和native等修饰，必须以分号结尾，不带花括号。



### 5 接口和抽象类在不同的JDK版本的变换？

从jdk1.8开始，接口中的方法不再是只能有抽象方法（普通方法会被隐式地指定为public abstract方法），他还可以有静态方法和default方法。并且静态方法与default方法可以有方法体！

### 6 通过实例对象.方法名的过程中都做了什么？



### 7 为什么父类一定要含无参构造函数？

​		如果父类没有提供默认的构造方法，而只是提供了有参构造方法，子类在继承时候，就会出错。
子类直接出现编译错误，错误提示是： 在父亲类那里没有找到默认的构造器。 说明：如果父类没有提供默认的构造方法，而只是提供了有参构造方法，子类在继承时候，就会出错。



### 8 简述静态类和单例的区别？

1. 什么是单例模式
   单例模式指的是在应用***整个生命周期内只能存在一个实例。***单例模式是一种被广泛使用的设计模式。他有很多好处，能够避免实例对象的重复创建，减少创建实例的系统开销，节省内存。
2. 单例模式和静态类的区别
   首先理解一下什么是静态类，静态类就是一个类里面都是静态方法和静态field，构造器被private修饰，因此不能被实例化。Math类就是一个静态类。



1）首先单例模式会提供给你一个全局唯一的对象，静态类只是提供给你很多静态方法，这些方法不用创建对象，通过类就可以直接调用；

2）***单例模式的灵活性更高，方法可以被override，因为静态类都是静态方法，所以不能被override；***

3）如果是一个非常重的对象，单例模式可以懒加载，静态类就无法做到；

那么时候时候应该用静态类，什么时候应该用单例模式呢？首先如果你只是想使用一些工具方法，那么最好用静态类，***静态类比单例类更快***，因为静态的绑定是在编译期进行的。***如果你要维护状态信息，或者访问资源时，应该选用单例模式。***还可以这样说，当你需要面向对象的能力时（比如继承、多态）时，选用单例类，当你仅仅是提供一些方法时选用静态类。



### 9 成员变量和局部变量的区别？

**1、在类中的位置不同**

成员变量：在类中方法外面

局部变量：在方法或者代码块中，或者方法的声明上（即在参数列表中）

**2、在内存中的位置不同，可以看看[Java程序内存的简单分析](http://www.cnblogs.com/huangminwen/p/5928315.html)**

成员变量：**在堆中**（方法区中的静态区）

局部变量：**在栈中**

**3、生命周期不同**

成员变量：随着对象的创建而存在，随着对象的消失而消失

局部变量：**随着方法的调用或者代码块的执行而存在，随着方法的调用完毕或者代码块的执行完毕而消失**

**4、初始值**

成员变量：有默认初始值

局部变量：没有默认初始值，使用之前需要赋值，否则编译器会报错（The local variable xxx may not have been initialized）

### 10 静态方法和实例方法的区别？

1、静态方法属于整个类所有，因此调用它不需要实例化，可以直接调用（类.静态方法（））。实例方法必须先实例化，创建一个对象，才能进行调用（对象.实例方法（））。
2、静态方法只能访问静态成员，不能访问实例成员；而实例方法可以访问静态成员和实例成员。
3、在程序运行期间，静态方法是一直存放在内存中，因此调用速度快，但是却占用内存。实例方法是使用完成后由回收机制自动进行回收，下次再使用必须再实例化。
4、一般来说，公共的函数、经常调用的可以写成静态方法，比如数据连接等（SqlHelper)。



### 11 为什么java中只有值传递？

回答自己的理解。

**无论是值传递还是引用传递，其实都是一种求值策略(Evaluation strategy)。**在求值策略中，还有一种叫做按**共享传递**(call by sharing)。其实Java中的参数传递严格意义上说应该是按共享传递。

按共享传递，是指在调用函数时，传递给函数的是实参的地址的拷贝（如果实参在栈中，则直接拷贝该值）。在函数内部对参数进行操作时，需要先拷贝的地址寻找到具体的值，再进行操作。如果该值在栈中，那么因为是直接拷贝的值，所以函数内部对参数进行操作不会对外部变量产生影响。如果原来拷贝的是原值在堆中的地址，那么需要先根据该地址找到堆中对应的位置，再进行操作。因为传递的是地址的拷贝所以函数内对值的操作对外部变量是可见的。

简单点说，Java中的传递，是值传递，而这个值，实际上是对象的引用。

**而按共享传递其实只是按值传递的一个特例罢了。**所以我们可以说Java的传递是按共享传递，或者说Java中的传递是值传递



# 常量池

### 1 简述对java字符串常量池的认识？

链接：https://zhuanlan.zhihu.com/p/52710835

java中有**几种不同的常量池**，以下的内容是对java中几种常量池的介绍以及重点研究一下字符串常量池。

**class常量池**
我们写的每一个Java类被编译后，就会形成一份class文件；**class文件中除了包含类的版本、字段、方法、接口等描述信息外，还有一项信息就是常量池(constant pool table)，用于存放编译器生成的各种 字面量 (Literal)和 符号引用 (Symbolic References)，每个class文件都有一个class常量池**。

其中 字面量 包括：1.文本字符串 2.八种基本类型的值 3.被声明为final的常量等; 符号引用 包括：1.类和方法的全限定名 2.字段的名称和描述符 3.方法的名称和描述符。

**运行时常量池**
运行时常量池存在于内存中，也就是**class常量池被加载到内存之后的版本**，是**方法区的一部分**。不同之处是：它的字面量可以动态的添加(String类的intern()),符号引用可以被解析为直接引用。

JVM在执行某个类的时候，必须经过加载、连接、初始化，而连接又包括验证、准备、解析三个阶段。而当类加载到内存中后，jvm就会将class常量池中的内容存放到运行时常量池中，由此可知，运行时常量池也是每个类都有一个。在解析阶段，会把符号引用替换为直接引用，解析的过程会去查询字符串常量池，也就是我们下面要说的StringTable，以保证运行时常量池所引用的字符串与字符串常量池中是一致的。

**字符串常量池**
在JDK6.0及之前版本，字符串常量池存放在方法区中在JDK7.0版本以后，字符串常量池被移到了堆中了。至于为什么移到堆内，大概是由于方法区的内存空间太小了。



在HotSpot VM里实现的string pool功能的是一个StringTable类，它是一个Hash表，默认值大小长度是1009；这个StringTable在每个HotSpot VM的实例只有一份，被所有的类共享。字符串常量由一个一个字符组成，放在了StringTable上。

在JDK6.0中，StringTable的长度是固定的，长度就是1009，因此如果放入String Pool中的String非常多，就会造成hash冲突，导致链表过长，当调用String#intern()时会需要到链表上一个一个找，从而导致性能大幅度下降；在JDK7.0中，StringTable的长度可以通过参数指定。

下面看一下实例：

String s = new String("abc")
这条语句创建了几个对象?

答案：共2个。第一个对象是”abc”字符串存储在常量池中，第二个对象在JAVA Heap中的 String 对象。这里不要混淆了s是放在栈里面的指向了Heap堆中的String对象。

比较下列两种创建字符串的方法：

String str1 = new String("abc");
String str2 = "abc";
答案：第一种是用new()来新建对象的，它会在存放于堆中。每调用一次就会创建一个新的对象。 运行时期创建 。

第二种是先在栈中创建一个对String类的对象引用变量str2，然后通过符号引用去字符串常量池里找有没有”abc”,如果没有，则将”abc”存放进字符串常量池，并令str2指向”abc”，如果已经有”abc” 则直接令str2指向“abc”。“abc”存于常量池在 编译期间完成 。

String s1 = new String("s1") ;
String s1 = new String("s1") ;
上面一共创建了几个对象？

答案：答案:3个 ,编译期Constant Pool中创建1个,运行期heap中创建2个.（用new创建的每new一次就在堆上创建一个对象，用引号创建的如果在常量池中已有就直接指向，不用创建）

比较字符串的‘==’和‘equals()’区别？

答案：

‘==’ 比较的是变量(栈)内存中存放的对象的(堆)内存地址，用来判断两个对象的地址是否相同，即是否是指相同一个对象。比较的是真正意义上的指针操作。注意：

1、比较的是操作符两端的操作数是否是同一个对象。

2、两边的操作数必须是同一类型的（可以是父子类之间）才能编译通过。

3、引用类型比较的是地址（即是否指向同一个对象），基本数据类型比较的是值，值相等则为true，如：int a=10 与 long b=10L 与 double c=10.0都是相同的（为true），因为他们都指向地址为10的堆。

‘equals()’用来比较的是两个对象是否相等，由于所有的类都是继承自java.lang.Object类的，在Object中的基类中定义了一个equals的方法，这个方法的初始行为是比较对象的内存地址，但String类中重写了equals方法， 比较的是字符串的内容 ，而不再是比较类在堆内存中的存放地址了。

总结：在没有重写equals方法的情况下，他们之间的比较还是基于他们在内存中的存放位置的地址值的，因为Object的equals方法也是用双等号（==）进行比较的，所以比较后的结果跟双等号（==）的结果相同。String类中重写了equals方法，变成了字符串内容的比较。

```js
String s1 = "sss111";
　　String s2 = "sss111";
　　System.out.println(s1 == s2); //结果为true
　　String s1 = new String("sss111");
　　String s2 = "sss111";
　　System.out.println(s1 == s2); //结果为false
String s0 = "111";              //pool
String s1 = new String("111");  //heap
final String s2 = "111";        //pool
String s3 = "sss111";           //pool
String s4 = "sss" + "111";      //pool
String s5 = "sss" + s0;         //heap 
String s6 = "sss" + s1;         //heap
String s7 = "sss" + s2;         //pool
String s8 = "sss" + s0;         //heap
 
System.out.println(s3 == s4);   //true
System.out.println(s3 == s5);   //false
System.out.println(s3 == s6);   //false
System.out.println(s3 == s7);   //true
System.out.println(s5 == s6);   //false
System.out.println(s5 == s8);   //false
```

结合上面分析,总结如下:向大家推荐一个架构学习交流裙。交流学习裙号：687810532，里面会分享一些资深架构师录制的视频录像

1.单独使用””引号创建的字符串都是常量,编译期就已经确定存储到Constant Pool中；

2.使用new String(“”)创建的对象会存储到heap中,是运行期新创建的；

3.使用只包含常量的字符串连接符如”aa” + “aa”创建的也是常量,编译期就能确定,已经确定存储到String Pool中,String pool中存有“aaaa”；但不会存有“aa”。

4.使用**包含变量的字符串连接符如”aa” + s1创建的对象是运行期才创建的,存储在heap中**；只要s1是变量，不论s1指向池中的字符串对象还是堆中的字符串对象，运行期s1 + “aa”操作实际上是编译器创建了StringBuilder对象进行了append操作后通过toString()返回了一个字符串对象存在heap上。

5.String s2 = “aa” + s1; String s3 = “aa” + s1; 这种情况，虽然s2,s3都是指向了使用包含变量的字符串连接符如”aa” + s1创建的存在堆上的对象，并且都是s1 + “aa”。但是却指向两个不同的对象，**两行代码实际上在堆上new出了两个StringBuilder对象来进行append操作**。在Thinking in java一书中285页的例子也可以说明。

6.对于final String s2 = “111”。**s2是一个用final修饰的变量，在编译期已知，在运行s2+”aa”时直接用常量**“111”来代替s2。所以s2+”aa”等效于“111”+ “aa”。在编译期就已经生成的字符串对象“111aa”存放在常量池中。

### 2 字符串常量池随着JDK的版本的变化而变化？

在JDK6.0及之前版本，字符串常量池存放在方法区中在JDK7.0版本以后，字符串常量池被移到了堆中了。至于为什么移到堆内，大概是由于方法区的内存空间太小了。

在HotSpot VM里实现的string pool功能的是一个StringTable类，它是一个Hash表，默认值大小长度是1009；这个StringTable在每个HotSpot VM的实例只有一份，被所有的类共享。字符串常量由一个一个字符组成，放在了StringTable上。

在JDK6.0中，StringTable的长度是固定的，长度就是1009，因此如果放入String Pool中的String非常多，就会造成hash冲突，导致链表过长，当调用String#intern()时会需要到链表上一个一个找，从而导致性能大幅度下降；在JDK7.0中，StringTable的长度可以通过参数指定。

### 3 简述java的Class常量池？



#### 1 **直接引用和符号引用简述**

​		举个例子：现在我要在A类中引用到B类，符号引用就是我只要知道B类的全类名是什么就可以了，而不用知道B类在内存中的那个具体位置（有可能B类还没有被加载进内存呢）。直接引用就相当于是一个指针，能够直接或者间接的定位到内存中的B类的具体位置。将符号引用转换为直接引用简单来说就是：在A类中可以通过使用B类的全类名转换得到B类在内存中的具体位置。

**Class常量池**
常量池中主要存放两类数据，**一是字面量、二是符号引用**。

![image-20210812195222172](D:\1书本笔记\java实战项目\image-20210812195222172.png)

1. 字面量：比如String类型的字符串值或者定义为final类型的常量的值。

2. 符号引用：

类或接口的全限定名（包括他的父类和所实现的接口）
变量或方法的名称
变量或方法的描述信息
this

   1. 方法的描述：参数个数、参数类型、方法返回类型等等
   2. 变量的描述信息：变量的返回值

局部变量的讨论
tip：网上有文章说不会将局部变量的变量名放入到常量池中？关于是否会将局部变量放入到常量池中，测试结果如下：

        ①.当在一个方法中有给局部变量赋值的语句时，会将局部变量名放入到常量池
    
        ②.当在方法中只是声明了一个局部变量，并没有为该局部变量的赋值操作时，不会将该局部变量名放入到常量池中
### 4 简述java的运行时常量池？

方法区的一部分，所有线程共享。虚拟机加载Class后把常量池中的数据放入到运行时常量池。

**运行时常量池**
运行时常量池存在于内存中，也就是**class常量池被加载到内存之后的版本**，是**方法区的一部分**。不同之处是：它的字面量可以动态的添加(String类的intern()),符号引用可以被解析为直接引用。

JVM在执行某个类的时候，必须经过加载、连接、初始化，而连接又包括验证、准备、解析三个阶段。而当类加载到内存中后，jvm就会将class常量池中的内容存放到运行时常量池中，由此可知，运行时常量池也是每个类都有一个。在解析阶段，会把符号引用替换为直接引用，解析的过程会去查询字符串常量池，也就是我们下面要说的StringTable，以保证运行时常量池所引用的字符串与字符串常量池中是一致的。

# 异常

### 1 简述对java异常的了解？

异常本质上是程序上的错误，包括程序逻辑错误和系统错误。比如使用空的引用、数组下标越界、内存溢出错误等，这些都是意外的情况，背离我们程序本身的意图。错误在我们编写程序的过程中会经常发生，包括编译期间和运行期间的错误，在编译期间出现的错误有编译器帮助我们一起修正，然而运行期间的错误便不是编译器力所能及了，并且运行期间的错误往往是难以预料的。假若程序在运行期间出现了错误，如果置之不理，程序便会终止或直接导致系统崩溃，显然这不是我们希望看到的结果


作者：武培轩
链接：https://juejin.cn/post/6844903981299433479
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

### 2 异常的继承体系结构

Java 把异常当作对象来处理，并定义一个基类 java.lang.Throwable 作为所有异常的超类。

Java 包括三种类型的异常: 检查性异常(checked exceptions)、非检查性异常(unchecked Exceptions) 和错误(errors)。

- 检查性异常(checked exceptions) 是必须在在方法的 throws 子句中声明的异常。它们扩展了异常，旨在成为一种“在你面前”的异常类型。JAVA希望你能够处理它们，因为它们以某种方式依赖于程序之外的外部因素。检查的异常表示在正常系统操作期间可能发生的预期问题。 当你尝试通过网络或文件系统使用外部系统时，通常会发生这些异常。 大多数情况下，对检查性异常的正确响应应该是稍后重试，或者提示用户修改其输入。
- 非检查性异常(unchecked Exceptions) 是不需要在throws子句中声明的异常。 由于程序错误，JVM并不会强制你处理它们，因为它们大多数是在运行时生成的。 它们扩展了 RuntimeException。 最常见的例子是 NullPointerException， 未经检查的异常可能不应该重试，正确的操作通常应该是什么都不做，并让它从你的方法和执行堆栈中出来。
- 错误(errors) 是严重的运行时环境问题，肯定无法恢复。 例如 OutOfMemoryError，LinkageError 和 StackOverflowError，通常会让程序崩溃。



![Java中的异常层次结构](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/10/26/16e088773b877fae~tplv-t2oaga2asx-watermark.awebp)

所有不是 Runtime Exception 的异常，统称为 Checked Exception，又被称为**检查性异常**。Java 语言将派生于 RuntimeException 类或 Error 类的所有异常称为非检查性异常。

**Exception又分为RuntimeException异常和非RuntimeException异常**

### 3 Exception和Error的区别？

**Error与Exception的区别**
	(1)Error类和Exception类都是继承Throwable类
	(2)Error（错误）是系统中的错误，程序员是不能改变的和处理的，是在程序编译时出现的错误，只能通过修改程序才能修正。一般是指与虚拟机相关的问题，如系统崩溃，虚拟机错误，内存空间不足，方法调用栈溢等。对于这类错误的导致的应用程序中断，仅靠程序本身无法恢复和和预防，遇到这样的错误，建议让程序终止。
	(3)Exception（异常）表示程序可以处理的异常，可以捕获且可能恢复。遇到这类异常，应该尽可能处理异常，使程序恢复运行，而不应该随意终止异常。
Exception又分为两类:
	CheckedException：（编译时异常） 需要用try——catch显示的捕获，对于可恢复的异常使用CheckedException。
	UnCheckedException（RuntimeException）：（运行时异常）不需要捕获，对于程序错误（不可恢复）的异常使用RuntimeException。
**常见的RuntimeException异常:**
	illegalArgumentException：此异常表明向方法传递了一个不合法或不正确的参数。
	illegalStateException：在不合理或不正确时间内唤醒一方法时出现的异常信息。换句话说，即 Java 环境或 Java 应用不满足请求操作。
	NullpointerException：空指针异常（我目前遇见的最多的）
	IndexOutOfBoundsException：索引超出边界异常
**常见的CheckedException异常:**
	我们在编写程序过程中try——catch捕获到的一场都是CheckedException。
	io包中的IOExecption及其子类，都是CheckedException。

### 4 简述throw和throws的区别？

throw：

1. 表示**方法内抛出某种异常对象**
2. 如果异常对象是非 RuntimeException 则需要在方法申明时加上该异常的抛出 即需要加上 throws 语句 或者 在方法体内 try catch 处理该异常，否则编译报错
3. 执行到 throw 语句则后面的语句块不再执行



throws：

1. **方法的定义上使用 throws 表示这个方法可能抛出某种异常**
2. **需要由方法的调用者进行异常处理**



# 泛型

作者：JayDroid
链接：https://www.jianshu.com/p/986f732ed2f1
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



> 泛型存在的意义？
>  泛型类，泛型接口，泛型方法如何定义？
>  如何限定类型变量？
>  泛型中使用的约束和局限性有哪些？
>  泛型类型的继承规则是什么？
>  泛型中的通配符类型是什么？
>  如何获取泛型的参数类型？
>  虚拟机是如何实现泛型的？
>  在日常开发中是如何运用泛型的？

![img](https:////upload-images.jianshu.io/upload_images/2516326-7bbe8045e54e21c5.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

Java泛型详解.png

## 一，晓之以理动之以码

## 1，泛型的定义以及存在意义

泛型，即“参数化类型”。就是将类型由原来的具体的类型参数化，类似于方法中的变量参数，此时类型也定义成参数形式（可以称之为类型形参），然后在使用/调用时传入具体的类型（类型实参）。
 例如：GenericClass**<T>**{}

> 一些常用的泛型类型变量：
>  E：元素（Element），多用于java集合框架
>  K：关键字（Key）
>  N：数字（Number）
>  T：类型（Type）
>  V：值（Value）

如果要实现不同类型的加法，每种类型都需要重载一个add方法



```csharp
package com.jay.java.泛型.needGeneric;

/**
 * Author：Jay On 2019/5/9 16:06
 * <p>
 * Description: 为什么使用泛型
 */
public class NeedGeneric1 {

    private static int add(int a, int b) {
        System.out.println(a + "+" + b + "=" + (a + b));
        return a + b;
    }

    private static float add(float a, float b) {
        System.out.println(a + "+" + b + "=" + (a + b));
        return a + b;
    }

    private static double add(double a, double b) {
        System.out.println(a + "+" + b + "=" + (a + b));
        return a + b;
    }

    private static <T extends Number> double add(T a, T b) {
        System.out.println(a + "+" + b + "=" + (a.doubleValue() + b.doubleValue()));
        return a.doubleValue() + b.doubleValue();
    }

    public static void main(String[] args) {
        NeedGeneric1.add(1, 2);
        NeedGeneric1.add(1f, 2f);
        NeedGeneric1.add(1d, 2d);
        NeedGeneric1.add(Integer.valueOf(1), Integer.valueOf(2));
        NeedGeneric1.add(Float.valueOf(1), Float.valueOf(2));
        NeedGeneric1.add(Double.valueOf(1), Double.valueOf(2));
    }
}
```

取出集合元素时需要人为的强制类型转化到具体的目标类型，且很容易现“java.lang. ClassCast Exception”异常。



```cpp
package com.jay.java.泛型.needGeneric;

import java.util.ArrayList;
import java.util.List;

/**
 * Author：Jay On 2019/5/9 16:23
 * <p>
 * Description: 为什么要使用泛型
 */
public class NeedGeneric2 {
    static class C{

    }
    public static void main(String[] args) {
        List list=new ArrayList();
        list.add("A");
        list.add("B");
        list.add(new C());
        list.add(100);
        //1.当我们将一个对象放入集合中，集合不会记住此对象的类型，当再次从集合中取出此对象时，改对象的编译类型变成了Object类型，但其运行时类型任然为其本身类型。
        //2.因此，//1处取出集合元素时需要人为的强制类型转化到具体的目标类型，且很容易出现“java.lang.ClassCastException”异常。
        for (int i = 0; i < list.size(); i++) {
//            System.out.println(list.get(i));
            String value= (String) list.get(i);
            System.out.println(value);
        }
    }
}
```

所以使用泛型的意义在于
 **1,适用于多种数据类型执行相同的代码（代码复用）
 2, 泛型中的类型在使用时指定，不需要强制类型转换（类型安全，编译器会检查类型）**

## 2，泛型类的使用

定义一个泛型类：public class `GenericClass<T>`{}



```cpp
package com.jay.java.泛型.DefineGeneric;

/**
 * Author：Jay On 2019/5/9 16:49
 * <p>
 * Description: 泛型类
 */
public class GenericClass<T> {
    private T data;

    public T getData() {
        return data;
    }

    public void setData(T data) {
        this.data = data;
    }

    public static void main(String[] args) {
        GenericClass<String> genericClass=new GenericClass<>();
        genericClass.setData("Generic Class");
        System.out.println(genericClass.getData());
    }
}
```

## 3，泛型接口的使用

定义一个泛型接口：public interface `GenericIntercace<T>`{}



```dart
/**
 * Author：Jay On 2019/5/9 16:57
 * <p>
 * Description: 泛型接口
 */
public interface GenericIntercace<T> {
     T getData();
}
```

实现泛型接口方式一：public class `ImplGenericInterface1<T>` implements `GenericIntercace<T>`



```cpp
/**
 * Author：Jay On 2019/5/9 16:59
 * <p>
 * Description: 泛型接口实现类-泛型类实现方式
 */
public class ImplGenericInterface1<T> implements GenericIntercace<T> {
    private T data;

    private void setData(T data) {
        this.data = data;
    }

    @Override
    public T getData() {
        return data;
    }

    public static void main(String[] args) {
        ImplGenericInterface1<String> implGenericInterface1 = new ImplGenericInterface1<>();
        implGenericInterface1.setData("Generic Interface1");
        System.out.println(implGenericInterface1.getData());
    }
}
```

实现泛型接口方式二：public class ImplGenericInterface2 implements `GenericIntercace<String>` {}



```dart
/**
 * Author：Jay On 2019/5/9 17:01
 * <p>
 * Description: 泛型接口实现类-指定具体类型实现方式
 */
public class ImplGenericInterface2 implements GenericIntercace<String> {
    @Override
    public String getData() {
        return "Generic Interface2";
    }

    public static void main(String[] args) {
        ImplGenericInterface2 implGenericInterface2 = new ImplGenericInterface2();
        System.out.println(implGenericInterface2.getData());
    }
}
```

## 4，泛型方法的使用

定义一个泛型方法： private static`<T> T`genericAdd(T a, T b) {}



```csharp
/**
 * Author：Jay On 2019/5/10 10:46
 * <p>
 * Description: 泛型方法
 */
public class GenericMethod1 {
    private static int add(int a, int b) {
        System.out.println(a + "+" + b + "=" + (a + b));
        return a + b;
    }

    private static <T> T genericAdd(T a, T b) {
        System.out.println(a + "+" + b + "="+a+b);
        return a;
    }

    public static void main(String[] args) {
        GenericMethod1.add(1, 2);
        GenericMethod1.<String>genericAdd("a", "b");
    }
}
```



```java
/**
 * Author：Jay On 2019/5/10 16:22
 * <p>
 * Description: 泛型方法
 */
public class GenericMethod3 {

    static class Animal {
        @Override
        public String toString() {
            return "Animal";
        }
    }

    static class Dog extends Animal {
        @Override
        public String toString() {
            return "Dog";
        }
    }

    static class Fruit {
        @Override
        public String toString() {
            return "Fruit";
        }
    }

    static class GenericClass<T> {

        public void show01(T t) {
            System.out.println(t.toString());
        }

        public <T> void show02(T t) {
            System.out.println(t.toString());
        }

        public <K> void show03(K k) {
            System.out.println(k.toString());
        }
    }

    public static void main(String[] args) {
        Animal animal = new Animal();
        Dog dog = new Dog();
        Fruit fruit = new Fruit();
        GenericClass<Animal> genericClass = new GenericClass<>();
        //泛型类在初始化时限制了参数类型
        genericClass.show01(dog);
//        genericClass.show01(fruit);

        //泛型方法的参数类型在使用时指定
        genericClass.show02(dog);
        genericClass.show02(fruit);

        genericClass.<Animal>show03(animal);
        genericClass.<Animal>show03(dog);
        genericClass.show03(fruit);
//        genericClass.<Dog>show03(animal);
    }
}
```

## 5，限定泛型类型变量

1,对类的限定：public class `TypeLimitForClass<T extends List & Serializable>{}`
 2,对方法的限定：public static`<T extends Comparable<T>>`T getMin(T a, T b) {}



```dart
/**
 * Author：Jay On 2019/5/10 16:38
 * <p>
 * Description: 类型变量的限定-方法
 */
public class TypeLimitForMethod {

    /**
     * 计算最小值
     * 如果要实现这样的功能就需要对泛型方法的类型做出限定
     */
//    private static <T> T getMin(T a, T b) {
//        return (a.compareTo(b) > 0) ? a : b;
//    }

    /**
     * 限定类型使用extends关键字指定
     * 可以使类，接口，类放在前面接口放在后面用&符号分割
     * 例如：<T extends ArrayList & Comparable<T> & Serializable>
     */
    public static <T extends Comparable<T>> T getMin(T a, T b) {
        return (a.compareTo(b) < 0) ? a : b;
    }

    public static void main(String[] args) {
        System.out.println(TypeLimitForMethod.getMin(2, 4));
        System.out.println(TypeLimitForMethod.getMin("a", "r"));
    }
}
```



```cpp
/**
 * Author：Jay On 2019/5/10 17:02
 * <p>
 * Description: 类型变量的限定-类
 */
public class TypeLimitForClass<T extends List & Serializable> {
    private T data;

    public T getData() {
        return data;
    }

    public void setData(T data) {
        this.data = data;
    }

    public static void main(String[] args) {
        ArrayList<String> stringArrayList = new ArrayList<>();
        stringArrayList.add("A");
        stringArrayList.add("B");
        ArrayList<Integer> integerArrayList = new ArrayList<>();
        integerArrayList.add(1);
        integerArrayList.add(2);
        integerArrayList.add(3);
        TypeLimitForClass<ArrayList> typeLimitForClass01 = new TypeLimitForClass<>();
        typeLimitForClass01.setData(stringArrayList);
        TypeLimitForClass<ArrayList> typeLimitForClass02 = new TypeLimitForClass<>();
        typeLimitForClass02.setData(integerArrayList);

        System.out.println(getMinListSize(typeLimitForClass01.getData().size(), typeLimitForClass02.getData().size()));

    }

    public static <T extends Comparable<T>> T getMinListSize(T a, T b) {
        return (a.compareTo(b) < 0) ? a : b;
    }
```

## 6，泛型中的约束和局限性

1,不能实例化泛型类
 2,静态变量或方法不能引用泛型类型变量，但是静态泛型方法是可以的
 3,基本类型无法作为泛型类型
 4,无法使用instanceof关键字或==判断泛型类的类型
 5,泛型类的原生类型与所传递的泛型无关，无论传递什么类型，原生类是一样的
 6,泛型数组可以声明但无法实例化
 7,泛型类不能继承Exception或者Throwable
 8,不能捕获泛型类型限定的异常但可以将泛型限定的异常抛出



```dart
/**
 * Author：Jay On 2019/5/10 17:41
 * <p>
 * Description: 泛型的约束和局限性
 */
public class GenericRestrict1<T> {
    static class NormalClass {

    }

    private T data;

    /**
     * 不能实例化泛型类
     * Type parameter 'T' cannot be instantiated directly
     */
    public void setData() {
        //this.data = new T();
    }

    /**
     * 静态变量或方法不能引用泛型类型变量
     * 'com.jay.java.泛型.restrict.GenericRestrict1.this' cannot be referenced from a static context
     */
//    private static T result;

//    private static T getResult() {
//        return result;
//    }

    /**
     * 静态泛型方法是可以的
     */
    private static <K> K getKey(K k) {
        return k;
    }

    public static void main(String[] args) {
        NormalClass normalClassA = new NormalClass();
        NormalClass normalClassB = new NormalClass();
        /**
         * 基本类型无法作为泛型类型
         */
//        GenericRestrict1<int> genericRestrictInt = new GenericRestrict1<>();
        GenericRestrict1<Integer> genericRestrictInteger = new GenericRestrict1<>();
        GenericRestrict1<String> genericRestrictString = new GenericRestrict1<>();
        /**
         * 无法使用instanceof关键字判断泛型类的类型
         * Illegal generic type for instanceof
         */
//        if(genericRestrictInteger instanceof GenericRestrict1<Integer>){
//            return;
//        }

        /**
         * 无法使用“==”判断两个泛型类的实例
         * Operator '==' cannot be applied to this two instance
         */
//        if (genericRestrictInteger == genericRestrictString) {
//            return;
//        }

        /**
         * 泛型类的原生类型与所传递的泛型无关，无论传递什么类型，原生类是一样的
         */
        System.out.println(normalClassA == normalClassB);//false
        System.out.println(genericRestrictInteger == genericRestrictInteger);//
        System.out.println(genericRestrictInteger.getClass() == genericRestrictString.getClass()); //true
        System.out.println(genericRestrictInteger.getClass());//com.jay.java.泛型.restrict.GenericRestrict1
        System.out.println(genericRestrictString.getClass());//com.jay.java.泛型.restrict.GenericRestrict1

        /**
         * 泛型数组可以声明但无法实例化
         * Generic array creation
         */
        GenericRestrict1<String>[] genericRestrict1s;
//        genericRestrict1s = new GenericRestrict1<String>[10];
        genericRestrict1s = new GenericRestrict1[10];
        genericRestrict1s[0]=genericRestrictString;
    }

}
```



```dart
/**
 * Author：Jay On 2019/5/10 18:45
 * <p>
 * Description: 泛型和异常
 */
public class GenericRestrict2 {

    private class MyException extends Exception {
    }

    /**
     * 泛型类不能继承Exception或者Throwable
     * Generic class may not extend 'java.lang.Throwable'
     */
//    private class MyGenericException<T> extends Exception {
//    }
//
//    private class MyGenericThrowable<T> extends Throwable {
//    }

    /**
     * 不能捕获泛型类型限定的异常
     * Cannot catch type parameters
     */
    public <T extends Exception> void getException(T t) {
//        try {
//
//        } catch (T e) {
//
//        }
    }

    /**
     *可以将泛型限定的异常抛出
     */
    public <T extends Throwable> void getException(T t) throws T {
        try {

        } catch (Exception e) {
            throw t;
        }
    }
}
```

## 7，泛型类型继承规则

1,对于泛型参数是继承关系的泛型类之间是没有继承关系的
 2,泛型类可以继承其它泛型类，例如: public class ArrayList<E> extends AbstractList<E>
 3,泛型类的继承关系在使用中同样会受到泛型类型的影响



```cpp
/**
 * Author：Jay On 2019/5/10 19:13
 * <p>
 * Description: 泛型继承规则测试类
 */
public class GenericInherit<T> {
    private T data1;
    private T data2;

    public T getData1() {
        return data1;
    }

    public void setData1(T data1) {
        this.data1 = data1;
    }

    public T getData2() {
        return data2;
    }

    public void setData2(T data2) {
        this.data2 = data2;
    }

    public static <V> void setData2(GenericInherit<Father> data2) {

    }

    public static void main(String[] args) {
//        Son 继承自 Father
        Father father = new Father();
        Son son = new Son();
        GenericInherit<Father> fatherGenericInherit = new GenericInherit<>();
        GenericInherit<Son> sonGenericInherit = new GenericInherit<>();
        SubGenericInherit<Father> fatherSubGenericInherit = new SubGenericInherit<>();
        SubGenericInherit<Son> sonSubGenericInherit = new SubGenericInherit<>();

        /**
         * 对于传递的泛型类型是继承关系的泛型类之间是没有继承关系的
         * GenericInherit<Father> 与GenericInherit<Son> 没有继承关系
         * Incompatible types.
         */
        father = new Son();
//        fatherGenericInherit=new GenericInherit<Son>();

        /**
         * 泛型类可以继承其它泛型类，例如: public class ArrayList<E> extends AbstractList<E>
         */
        fatherGenericInherit=new SubGenericInherit<Father>();

        /**
         *泛型类的继承关系在使用中同样会受到泛型类型的影响
         */
        setData2(fatherGenericInherit);
//        setData2(sonGenericInherit);
        setData2(fatherSubGenericInherit);
//        setData2(sonSubGenericInherit);

    }

    private static class SubGenericInherit<T> extends GenericInherit<T> {

    }
```

## 8，通配符类型

1,`<? extends Parent>` 指定了泛型类型的上届
 2,`<? super Child>` 指定了泛型类型的下届
 3, `<?>` 指定了没有限制的泛型类型

![img](https:////upload-images.jianshu.io/upload_images/2516326-c36715daf91572cf.png?imageMogr2/auto-orient/strip|imageView2/2/w/487/format/webp)

通配符测试类结构.png



```csharp
/**
 * Author：Jay On 2019/5/10 19:51
 * <p>
 * Description: 泛型通配符测试类
 */
public class GenericByWildcard {
    private static void print(GenericClass<Fruit> fruitGenericClass) {
        System.out.println(fruitGenericClass.getData().getColor());
    }

    private static void use() {
        GenericClass<Fruit> fruitGenericClass = new GenericClass<>();
        print(fruitGenericClass);
        GenericClass<Orange> orangeGenericClass = new GenericClass<>();
        //类型不匹配,可以使用<? extends Parent> 来解决
//        print(orangeGenericClass);
    }

    /**
     * <? extends Parent> 指定了泛型类型的上届
     */
    private static void printExtends(GenericClass<? extends Fruit> genericClass) {
        System.out.println(genericClass.getData().getColor());
    }

    public static void useExtend() {
        GenericClass<Fruit> fruitGenericClass = new GenericClass<>();
        printExtends(fruitGenericClass);
        GenericClass<Orange> orangeGenericClass = new GenericClass<>();
        printExtends(orangeGenericClass);

        GenericClass<Food> foodGenericClass = new GenericClass<>();
        //Food是Fruit的父类，超过了泛型上届范围，类型不匹配
//        printExtends(foodGenericClass);

        //表示GenericClass的类型参数的上届是Fruit
        GenericClass<? extends Fruit> extendFruitGenericClass = new GenericClass<>();
        Apple apple = new Apple();
        Fruit fruit = new Fruit();
        /*
         * 道理很简单，？ extends X  表示类型的上界，类型参数是X的子类，那么可以肯定的说，
         * get方法返回的一定是个X（不管是X或者X的子类）编译器是可以确定知道的。
         * 但是set方法只知道传入的是个X，至于具体是X的那个子类，不知道。
         * 总结：主要用于安全地访问数据，可以访问X及其子类型，并且不能写入非null的数据。
         */
//        extendFruitGenericClass.setData(apple);
//        extendFruitGenericClass.setData(fruit);

        fruit = extendFruitGenericClass.getData();

    }

    /**
     * <? super Child> 指定了泛型类型的下届
     */
    public static void printSuper(GenericClass<? super Apple> genericClass) {
        System.out.println(genericClass.getData());
    }

    public static void useSuper() {
        GenericClass<Food> foodGenericClass = new GenericClass<>();
        printSuper(foodGenericClass);

        GenericClass<Fruit> fruitGenericClass = new GenericClass<>();
        printSuper(fruitGenericClass);

        GenericClass<Apple> appleGenericClass = new GenericClass<>();
        printSuper(appleGenericClass);

        GenericClass<HongFuShiApple> hongFuShiAppleGenericClass = new GenericClass<>();
        // HongFuShiApple 是Apple的子类，达不到泛型下届，类型不匹配
//        printSuper(hongFuShiAppleGenericClass);

        GenericClass<Orange> orangeGenericClass = new GenericClass<>();
        // Orange和Apple是兄弟关系，没有继承关系，类型不匹配
//        printSuper(orangeGenericClass);

        //表示GenericClass的类型参数的下界是Apple
        GenericClass<? super Apple> supperAppleGenericClass = new GenericClass<>();
        supperAppleGenericClass.setData(new Apple());
        supperAppleGenericClass.setData(new HongFuShiApple());
        /*
         * ？ super  X  表示类型的下界，类型参数是X的超类（包括X本身），
         * 那么可以肯定的说，get方法返回的一定是个X的超类，那么到底是哪个超类？不知道，
         * 但是可以肯定的说，Object一定是它的超类，所以get方法返回Object。
         * 编译器是可以确定知道的。对于set方法来说，编译器不知道它需要的确切类型，但是X和X的子类可以安全的转型为X。
         * 总结：主要用于安全地写入数据，可以写入X及其子类型。
         */
//        supperAppleGenericClass.setData(new Fruit());

        //get方法只会返回一个Object类型的值。
        Object data = supperAppleGenericClass.getData();
    }

    /**
     * <?> 指定了没有限定的通配符
     */
    public static void printNonLimit(GenericClass<?> genericClass) {
        System.out.println(genericClass.getData());
    }

    public static void useNonLimit() {
        GenericClass<Food> foodGenericClass = new GenericClass<>();
        printNonLimit(foodGenericClass);
        GenericClass<Fruit> fruitGenericClass = new GenericClass<>();
        printNonLimit(fruitGenericClass);
        GenericClass<Apple> appleGenericClass = new GenericClass<>();
        printNonLimit(appleGenericClass);

        GenericClass<?> genericClass = new GenericClass<>();
        //setData 方法不能被调用， 甚至不能用 Object 调用；
//        genericClass.setData(foodGenericClass);
//        genericClass.setData(new Object());
        //返回值只能赋给 Object
        Object object = genericClass.getData();

    }

}
```

## 9，获取泛型的参数类型

[Type是什么](https://www.jianshu.com/p/c820e55d9f27)
 这里的Type指java.lang.reflect.Type, 是Java中所有类型的公共高级接口, 代表了Java中的所有类型. Type体系中类型的包括：数组类型(GenericArrayType)、参数化类型(ParameterizedType)、类型变量(TypeVariable)、通配符类型(WildcardType)、原始类型(Class)、基本类型(Class), 以上这些类型都实现Type接口.

> 参数化类型,就是我们平常所用到的泛型List、Map；
>  数组类型,并不是我们工作中所使用的数组String[] 、byte[]，而是带有泛型的数组，即T[] ；
>  通配符类型, 指的是<?>, <? extends T>等等
>  原始类型, 不仅仅包含我们平常所指的类，还包括枚举、数组、注解等；
>  基本类型, 也就是我们所说的java的基本类型，即int,float,double等



```dart
public interface ParameterizedType extends Type {
    // 返回确切的泛型参数, 如Map<String, Integer>返回[String, Integer]
    Type[] getActualTypeArguments();
    
    //返回当前class或interface声明的类型, 如List<?>返回List
    Type getRawType();
    
    //返回所属类型. 如,当前类型为O<T>.I<S>, 则返回O<T>. 顶级类型将返回null 
    Type getOwnerType();
}
```



```cpp
/**
 * Author：Jay On 2019/5/11 22:41
 * <p>
 * Description: 获取泛型类型测试类
 */
public class GenericType<T> {
    private T data;

    public T getData() {
        return data;
    }

    public void setData(T data) {
        this.data = data;
    }

    public static void main(String[] args) {
        GenericType<String> genericType = new GenericType<String>() {};
        Type superclass = genericType.getClass().getGenericSuperclass();
        //getActualTypeArguments 返回确切的泛型参数, 如Map<String, Integer>返回[String, Integer]
        Type type = ((ParameterizedType) superclass).getActualTypeArguments()[0]; 
        System.out.println(type);//class java.lang.String
    }
}
```

## 10，虚拟机是如何实现泛型的

Java泛型是Java1.5之后才引入的，为了向下兼容。Java采用了C++完全不同的实现思想。Java中的泛型更多的看起来像是编译期用的
 Java中泛型在运行期是不可见的，会被擦除为它的上级类型。如果是没有限定的泛型参数类型，就会被替换为Object.



```xml
GenericClass<String> stringGenericClass=new GenericClass<>();
GenericClass<Integer> integerGenericClass=new GenericClass<>();
```

C++中GenericClass<String>和GenericClass<Integer>是两个不同的类型
 Java进行了类型擦除之后统一改为GenericClass<Object>



```cpp
/**
 * Author：Jay On 2019/5/11 16:11
 * <p>
 * Description:泛型原理测试类
 */
public class GenericTheory {
    public static void main(String[] args) {
        Map<String, String> map = new HashMap<>();
        map.put("Key", "Value");
        System.out.println(map.get("Key"));
        GenericClass<String, String> genericClass = new GenericClass<>();
        genericClass.put("Key", "Value");
        System.out.println(genericClass.get("Key"));
    }

    public static class GenericClass<K, V> {
        private K key;
        private V value;

        public void put(K key, V value) {
            this.key = key;
            this.value = value;
        }

        public V get(V key) {
            return value;
        }
    }

    /**
     * 类型擦除后GenericClass2<Object>
     * @param <T>
     */
    private class GenericClass2<T> {

    }

    /**
     * 类型擦除后GenericClass3<ArrayList>
     * 当使用到Serializable时会将相应代码强制转换为Serializable
     * @param <T>
     */
    private class GenericClass3<T extends ArrayList & Serializable> {

    }
}
```

对应的字节码文件



```dart
 public static void main(String[] args) {
        Map<String, String> map = new HashMap();
        map.put("Key", "Value");
        System.out.println((String)map.get("Key"));
        GenericTheory.GenericClass<String, String> genericClass = new GenericTheory.GenericClass();
        genericClass.put("Key", "Value");
        System.out.println((String)genericClass.get("Key"));
    }
```

## 二，学以致用

## 1，泛型解析JSON数据封装

api返回的json数据



```json
{
    "code":200,
    "msg":"成功",
    "data":{
        "name":"Jay",
        "email":"10086"
    }
}
BaseResponse .java
```



```cpp
/**
 * Author：Jay On 2019/5/11 20:48
 * <p>
 * Description: 接口数据接收基类
 */
public class BaseResponse {

    private int code;
    private String msg;

    public int getCode() {
        return code;
    }

    public void setCode(int code) {
        this.code = code;
    }

    public String getMsg() {
        return msg;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }
}
UserResponse.java
```



```kotlin
/**
 * Author：Jay On 2019/5/11 20:49
 * <p>
 * Description: 用户信息接口实体类
 */
public class UserResponse<T> extends BaseResponse {
    private T data;

    public T getData() {
        return data;
    }

    public void setData(T data) {
        this.data = data;
    }
}
```

## 2，泛型+反射实现巧复用工具类



```java
/**
 * Author：Jay On 2019/5/11 21:05
 * <p>
 * Description: 泛型相关的工具类
 */
public class GenericUtils {

    public static class Movie {
        private String name;
        private Date time;

        public String getName() {
            return name;
        }

        public Date getTime() {
            return time;
        }

        public Movie(String name, Date time) {
            this.name = name;
            this.time = time;
        }

        @Override
        public String toString() {
            return "Movie{" + "name='" + name + '\'' + ", time=" + time + '}';
        }
    }

    public static void main(String[] args) {
        List<Movie> movieList = new ArrayList<>();
        for (int i = 0; i < 5; i++) {
            movieList.add(new Movie("movie" + i, new Date()));
        }
        System.out.println("排序前:" + movieList.toString());

        GenericUtils.sortAnyList(movieList, "name", true);
        System.out.println("按name正序排：" + movieList.toString());

        GenericUtils.sortAnyList(movieList, "name", false);
        System.out.println("按name逆序排：" + movieList.toString());
    }

    /**
     * 对任意集合的排序方法
     * @param targetList 要排序的实体类List集合
     * @param sortField  排序字段
     * @param sortMode   true正序，false逆序
     */
    public static <T> void sortAnyList(List<T> targetList, final String sortField, final boolean sortMode) {
        if (targetList == null || targetList.size() < 2 || sortField == null || sortField.length() == 0) {
            return;
        }
        Collections.sort(targetList, new Comparator<Object>() {
            @Override
            public int compare(Object obj1, Object obj2) {
                int retVal = 0;
                try {
                    // 获取getXxx()方法名称
                    String methodStr = "get" + sortField.substring(0, 1).toUpperCase() + sortField.substring(1);
                    Method method1 = ((T) obj1).getClass().getMethod(methodStr, null);
                    Method method2 = ((T) obj2).getClass().getMethod(methodStr, null);
                    if (sortMode) {
                        retVal = method1.invoke(((T) obj1), null).toString().compareTo(method2.invoke(((T) obj2), null).toString());
                    } else {
                        retVal = method2.invoke(((T) obj2), null).toString().compareTo(method1.invoke(((T) obj1), null).toString());
                    }
                } catch (Exception e) {
                    System.out.println("List<" + ((T) obj1).getClass().getName() + ">排序异常！");
                    e.printStackTrace();
                }
                return retVal;
            }
        });
    }
}
```

## 3，Gson库中的泛型的使用-TypeToken



```csharp
/**
 * Author：Jay On 2019/5/11 22:11
 * <p>
 * Description: Gson库中的泛型使用
 */
public class GsonGeneric {
    public static class Person {
        private String name;
        private int age;

        public Person(String name, int age) {
            this.name = name;
            this.age = age;
        }

        @Override
        public String toString() {
            return "Person{" +
                    "name='" + name + '\'' +
                    ", age=" + age +
                    '}';
        }
    }

    public static void main(String[] args) {
        Gson gson = new Gson();
        List<Person> personList = new ArrayList<>();
        for (int i = 0; i < 5; i++) {
            personList.add(new Person("name" + i, 18 + i));
        }
        // Serialization
        String json = gson.toJson(personList);
        System.out.println(json);
        // Deserialization
        Type personType = new TypeToken<List<Person>>() {}.getType();
        List<Person> personList2 = gson.fromJson(json, personType);
        System.out.println(personList2);
    }
}
```



## 三、泛型的自动擦除？

​		Java的泛型是伪泛型，这是因为Java在编译期间，所有的泛型信息都会被擦掉，正确理解泛型概念的首要前提是理解类型擦除。Java的泛型基本上都是在编译器这个层次上实现的，在生成的字节码中是不包含泛型中的类型信息的，使用泛型的时候加上类型参数，在编译器编译的时候会去掉，这个过程成为**类型擦除**。如在代码中定义`List<Object>`和`List<String>`等类型，在编译后都会变成`List`，JVM看到的只是`List`，而由泛型附加的类型信息对JVM是看不到的。Java编译器会在编译时尽可能的发现可能出错的地方，但是仍然无法在运行时刻出现的类型转换异常的情况，类型擦除也是 Java 的泛型与 C++ 模板机制实现方式之间的重要区别。

#### 1、原始类型相等

```java
public class Test {

    public static void main(String[] args) {

        ArrayList<String> list1 = new ArrayList<String>();
        list1.add("abc");

        ArrayList<Integer> list2 = new ArrayList<Integer>();
        list2.add(123);

        System.out.println(list1.getClass() == list2.getClass());
    }

}
```

​	在这个例子中，我们定义了两个`ArrayList`数组，不过一个是`ArrayList<String>`泛型类型的，只能存储字符串；一个是`ArrayList<Integer>`泛型类型的，只能存储整数，最后，我们通过`list1`对象和`list2`对象的`getClass()`方法获取他们的类的信息，最后发现结果为`true`。说明泛型类型`String`和`Integer`都被擦除掉了，只剩下原始类型。

#### 2、通过反射添加其它类型元素

```java
public class Test {

    public static void main(String[] args) throws Exception {

        ArrayList<Integer> list = new ArrayList<Integer>();

        list.add(1);  //这样调用 add 方法只能存储整形，因为泛型类型的实例为 Integer

        list.getClass().getMethod("add", Object.class).invoke(list, "asd");

        for (int i = 0; i < list.size(); i++) {
            System.out.println(list.get(i));
        }
    }

}
```

在程序中定义了一个`ArrayList`泛型类型实例化为`Integer`对象，如果直接调用`add()`方法，那么只能存储整数数据，不过当我们利用反射调用`add()`方法的时候，却可以存储字符串，这说明了`Integer`泛型实例在编译之后被擦除掉了，只保留了原始类型。

## 四、类型擦除后保留的原始类型

在上面，两次提到了原始类型，什么是原始类型？

**原始类型** 就是擦除去了泛型信息，最后在字节码中的类型变量的真正类型，无论何时定义一个泛型，相应的原始类型都会被自动提供，类型变量擦除，并使用其**限定类型**（无限定的变量用Object）替换。

#### 1、原始类型Object

```java
public class Pair<T> {  
    private T value;  
    public T getValue() {  
        return value;  
    }  
    public void setValue(T  value) {  
        this.value = value;  
    }  
}  
```

Pair的原始类型为:

```java
public class Pair {  
    private Object value;  
    public Object getValue() {  
        return value;  
    }  
    public void setValue(Object  value) {  
        this.value = value;  
    }  
}
```

因为在`Pair<T>`中，T 是一个无限定的类型变量，所以用`Object`替换，其结果就是一个普通的类，如同泛型加入Java语言之前的已经实现的样子。在程序中可以包含不同类型的`Pair`，如`Pair<String>`或`Pair<Integer>`，但是擦除类型后他们的就成为原始的`Pair`类型了，原始类型都是`Object`。

从上面的"一-2"中，我们也可以明白`ArrayList<Integer>`被擦除类型后，原始类型也变为`Object`，所以通过反射我们就可以存储字符串了。

如果类型变量有限定，那么原始类型就用第一个边界的类型变量类替换。

比如: Pair这样声明的话

```java
public class Pair<T extends Comparable> {}
```

那么原始类型就是`Comparable`。

要区分**原始类型**和**泛型变量的类型**。

在调用泛型方法时，可以指定泛型，也可以不指定泛型。

- 在不指定泛型的情况下，泛型变量的类型为该方法中的几种类型的同一父类的最小级，直到 Object。
- 在指定泛型的情况下，该方法的几种类型必须是该泛型的实例的类型或者其子类。

```java
public class Test {  
    public static void main(String[] args) {  

        /**不指定泛型的时候*/  
        int i = Test.add(1, 2); //这两个参数都是Integer，所以T为Integer类型  
        Number f = Test.add(1, 1.2); //这两个参数一个是Integer，以风格是Float，所以取同一父类的最小级，为Number  
        Object o = Test.add(1, "asd"); //这两个参数一个是Integer，以风格是Float，所以取同一父类的最小级，为Object  

        /**指定泛型的时候*/  
        int a = Test.<Integer>add(1, 2); //指定了Integer，所以只能为Integer类型或者其子类  
        int b = Test.<Integer>add(1, 2.2); //编译错误，指定了Integer，不能为Float  
        Number c = Test.<Number>add(1, 2.2); //指定为Number，所以可以为Integer和Float  
    }  

    //这是一个简单的泛型方法  
    public static <T> T add(T x,T y){  
        return y;  
    }  
}
```

其实在泛型类中，不指定泛型的时候，也差不多，只不过这个时候的泛型为`Object`，就比如`ArrayList`中，如果不指定泛型，那么这个`ArrayList`可以存储任意的对象。

#### 2、Object泛型

```java
public static void main(String[] args) {  
    ArrayList list = new ArrayList();  
    list.add(1);  
    list.add("121");  
    list.add(new Date());  
}  
```

# 五、类型擦除引起的问题及解决方法

因为种种原因，Java不能实现真正的泛型，只能使用类型擦除来实现伪泛型，这样虽然不会有类型膨胀问题，但是也引起来许多新问题，所以，SUN对这些问题做出了种种限制，避免我们发生各种错误。

#### 1、先检查再编译以及编译的对象和引用传递问题

**Q**: 既然说类型变量会在编译的时候擦除掉，那为什么我们往 ArrayList 创建的对象中添加整数会报错呢？不是说泛型变量String会在编译的时候变为Object类型吗？为什么不能存别的类型呢？既然类型擦除了，如何保证我们只能使用泛型变量限定的类型呢？

**A**: Java编译器是通过先检查代码中泛型的类型，然后在进行类型擦除，再进行编译。

例如：

```java
public static  void main(String[] args) {  

    ArrayList<String> list = new ArrayList<String>();  
    list.add("123");  
    list.add(123);//编译错误  
}
```

在上面的程序中，使用`add`方法添加一个整型，在IDE中，直接会报错，说明这就是在编译之前的检查，因为如果是在编译之后检查，类型擦除后，原始类型为`Object`，是应该允许任意引用类型添加的。可实际上却不是这样的，这恰恰说明了关于泛型变量的使用，是会在编译之前检查的。

那么，这个类型检查是针对谁的呢？我们先看看参数化类型和原始类型的兼容。

以 ArrayList举例子，以前的写法:

```java
ArrayList list = new ArrayList();  
```

现在的写法:

```java
ArrayList<String> list = new ArrayList<String>();
```

如果是与以前的代码兼容，各种引用传值之间，必然会出现如下的情况：

```java
ArrayList<String> list1 = new ArrayList(); //第一种 情况
ArrayList list2 = new ArrayList<String>(); //第二种 情况
```

这样是没有错误的，不过会有个编译时警告。

不过在第一种情况，可以实现与完全使用泛型参数一样的效果，第二种则没有效果。

因为类型检查就是编译时完成的，`new ArrayList()`只是在内存中开辟了一个存储空间，可以存储任何类型对象，而**真正设计类型检查的是它的引用**，因为我们是使用它引用`list1`来调用它的方法，比如说调用`add`方法，所以`list1`引用能完成泛型类型的检查。而引用`list2`没有使用泛型，所以不行。

举例子：

```java
public class Test {  

    public static void main(String[] args) {  

        ArrayList<String> list1 = new ArrayList();  
        list1.add("1"); //编译通过  
        list1.add(1); //编译错误  
        String str1 = list1.get(0); //返回类型就是String  

        ArrayList list2 = new ArrayList<String>();  
        list2.add("1"); //编译通过  
        list2.add(1); //编译通过  
        Object object = list2.get(0); //返回类型就是Object  

        new ArrayList<String>().add("11"); //编译通过  
        new ArrayList<String>().add(22); //编译错误  

        String str2 = new ArrayList<String>().get(0); //返回类型就是String  
    }  

}  
```

通过上面的例子，我们可以明白，**类型检查就是针对引用的**，谁是一个引用，用这个引用调用泛型方法，就会对这个引用调用的方法进行类型检测，而无关它真正引用的对象。

泛型中参数话类型为什么不考虑继承关系？

在Java中，像下面形式的引用传递是不允许的:

```java
ArrayList<String> list1 = new ArrayList<Object>(); //编译错误  
ArrayList<Object> list2 = new ArrayList<String>(); //编译错误
```

我们先看第一种情况，将第一种情况拓展成下面的形式：

```java
ArrayList<Object> list1 = new ArrayList<Object>();  
list1.add(new Object());  
list1.add(new Object());  
ArrayList<String> list2 = list1; //编译错误
```

实际上，在第4行代码的时候，就会有编译错误。那么，我们先假设它编译没错。那么当我们使用`list2`引用用`get()`方法取值的时候，返回的都是`String`类型的对象（上面提到了，类型检测是根据引用来决定的），可是它里面实际上已经被我们存放了`Object`类型的对象，这样就会有`ClassCastException`了。所以为了避免这种极易出现的错误，Java不允许进行这样的引用传递。（这也是泛型出现的原因，就是为了解决类型转换的问题，我们不能违背它的初衷）。

再看第二种情况，将第二种情况拓展成下面的形式：

```java
ArrayList<String> list1 = new ArrayList<String>();  
list1.add(new String());  
list1.add(new String());

ArrayList<Object> list2 = list1; //编译错误
```

没错，这样的情况比第一种情况好的多，最起码，在我们用`list2`取值的时候不会出现`ClassCastException`，因为是从`String`转换为`Object`。可是，这样做有什么意义呢，泛型出现的原因，就是为了解决类型转换的问题。我们使用了泛型，到头来，还是要自己强转，违背了泛型设计的初衷。所以java不允许这么干。再说，你如果又用`list2`往里面`add()`新的对象，那么到时候取得时候，我怎么知道我取出来的到底是`String`类型的，还是`Object`类型的呢？

**所以，要格外注意，泛型中的引用传递的问题。**

#### 2、自动类型转换

因为类型擦除的问题，所以所有的泛型类型变量最后都会被替换为原始类型。

既然都被替换为原始类型，那么为什么我们在获取的时候，不需要进行强制类型转换呢？

看下`ArrayList.get()`方法：

```java
public E get(int index) {  

    RangeCheck(index);  

    return (E) elementData[index];  

}
```

可以看到，在`return`之前，会根据泛型变量进行强转。假设泛型类型变量为`Date`，虽然泛型信息会被擦除掉，但是会将`(E) elementData[index]`，编译为`(Date) elementData[index]`。所以我们不用自己进行强转。当存取一个泛型域时也会自动插入强制类型转换。假设`Pair`类的`value`域是`public`的，那么表达式：

```java
Date date = pair.value;
```

也会自动地在结果字节码中插入强制类型转换。

#### 3、类型擦除与多态的冲突和解决方法

现在有这样一个泛型类：

```java
class Pair<T> {  

    private T value;  

    public T getValue() {  
        return value;  
    }  

    public void setValue(T value) {  
        this.value = value;  
    }  
}
```

然后我们想要一个子类继承它。

```java
class DateInter extends Pair<Date> {  

    @Override
    public void setValue(Date value) {  
        super.setValue(value);  
    }  

    @Override
    public Date getValue() {  
        return super.getValue();  
    }  
}
```

在这个子类中，我们设定父类的泛型类型为`Pair<Date>`，在子类中，我们覆盖了父类的两个方法，我们的原意是这样的：将父类的泛型类型限定为`Date`，那么父类里面的两个方法的参数都为`Date`类型。

```java
public Date getValue() {  
    return value;  
}  

public void setValue(Date value) {  
    this.value = value;  
}
```

所以，我们在子类中重写这两个方法一点问题也没有，实际上，从他们的`@Override`标签中也可以看到，一点问题也没有，实际上是这样的吗？

分析：实际上，类型擦除后，父类的的泛型类型全部变为了原始类型`Object`，所以父类编译之后会变成下面的样子：

```java
class Pair {  
    private Object value;  

    public Object getValue() {  
        return value;  
    }  

    public void setValue(Object  value) {  
        this.value = value;  
    }  
}  
```

再看子类的两个重写的方法的类型：

```java
@Override  
public void setValue(Date value) {  
    super.setValue(value);  
}  
@Override  
public Date getValue() {  
    return super.getValue();  
}
```

先来分析`setValue`方法，父类的类型是`Object`，而子类的类型是`Date`，参数类型不一样，这如果实在普通的继承关系中，根本就不会是重写，而是重载。

我们在一个main方法测试一下：

```java
public static void main(String[] args) throws ClassNotFoundException {  
        DateInter dateInter = new DateInter();  
        dateInter.setValue(new Date());                  
        dateInter.setValue(new Object()); //编译错误  
}
```

如果是重载，那么子类中两个`setValue`方法，一个是参数`Object`类型，一个是`Date`类型，可是我们发现，根本就没有这样的一个子类继承自父类的Object类型参数的方法。所以说，却是是重写了，而不是重载了。

为什么会这样呢？

原因是这样的，我们传入父类的泛型类型是`Date，Pair<Date>`，我们的本意是将泛型类变为如下：

```java
class Pair {  
    private Date value;  
    public Date getValue() {  
        return value;  
    }  
    public void setValue(Date value) {  
        this.value = value;  
    }  
}
```

然后再子类中重写参数类型为Date的那两个方法，实现继承中的多态。

可是由于种种原因，虚拟机并不能将泛型类型变为`Date`，只能将类型擦除掉，变为原始类型`Object`。这样，我们的本意是进行重写，实现多态。可是类型擦除后，只能变为了重载。这样，类型擦除就和多态有了冲突。JVM知道你的本意吗？知道！！！可是它能直接实现吗，不能！！！如果真的不能的话，那我们怎么去重写我们想要的`Date`类型参数的方法啊。

于是JVM采用了一个特殊的方法，来完成这项功能，那就是**桥方法**。

首先，我们用`javap -c className`的方式反编译下`DateInter`子类的字节码，结果如下：

```class
class com.tao.test.DateInter extends com.tao.test.Pair<java.util.Date> {  
  com.tao.test.DateInter();  
    Code:  
       0: aload_0  
       1: invokespecial #8                  // Method com/tao/test/Pair."<init>":()V  
       4: return  

  public void setValue(java.util.Date);  //我们重写的setValue方法  
    Code:  
       0: aload_0  
       1: aload_1  
       2: invokespecial #16                 // Method com/tao/test/Pair.setValue:(Ljava/lang/Object;)V  
       5: return  

  public java.util.Date getValue();    //我们重写的getValue方法  
    Code:  
       0: aload_0  
       1: invokespecial #23                 // Method com/tao/test/Pair.getValue:()Ljava/lang/Object;  
       4: checkcast     #26                 // class java/util/Date  
       7: areturn  

  public java.lang.Object getValue();     //编译时由编译器生成的桥方法  
    Code:  
       0: aload_0  
       1: invokevirtual #28                 // Method getValue:()Ljava/util/Date 去调用我们重写的getValue方法;  
       4: areturn  

  public void setValue(java.lang.Object);   //编译时由编译器生成的桥方法  
    Code:  
       0: aload_0  
       1: aload_1  
       2: checkcast     #26                 // class java/util/Date  
       5: invokevirtual #30                 // Method setValue:(Ljava/util/Date; 去调用我们重写的setValue方法)V  
       8: return  
}
```

从编译的结果来看，我们本意重写`setValue`和`getValue`方法的子类，竟然有4个方法，其实不用惊奇，最后的两个方法，就是编译器自己生成的桥方法。可以看到桥方法的参数类型都是Object，也就是说，子类中真正覆盖父类两个方法的就是这两个我们看不到的桥方法。而在我们自己定义的`setvalue`和`getValue`方法上面的`@Oveerride`只不过是假象。而桥方法的内部实现，就只是去调用我们自己重写的那两个方法。

所以，**虚拟机巧妙的使用了桥方法，来解决了类型擦除和多态的冲突**。

不过，要提到一点，这里面的`setValue`和`getValue`这两个桥方法的意义又有不同。

`setValue`方法是为了解决类型擦除与多态之间的冲突。

而`getValue`却有普遍的意义，怎么说呢，如果这是一个普通的继承关系：

那么父类的`getValue`方法如下：

```java
public Object getValue() {  
    return value;  
}
```

而子类重写的方法是：

```java
public Date getValue() {  
    return super.getValue();  
}
```

其实这在普通的类继承中也是普遍存在的重写，这就是协变。

关于协变：。。。。。。

并且，还有一点也许会有疑问，子类中的桥方法`Object getValue()`和`Date getValue()`是同时存在的，可是如果是常规的两个方法，他们的方法签名是一样的，也就是说虚拟机根本不能分别这两个方法。如果是我们自己编写Java代码，这样的代码是无法通过编译器的检查的，但是虚拟机却是允许这样做的，因为虚拟机通过参数类型和返回类型来确定一个方法，所以编译器为了实现泛型的多态允许自己做这个看起来“不合法”的事情，然后交给虚拟器去区别。

#### 4、泛型类型变量不能是基本数据类型

不能用类型参数替换基本类型。就比如，没有`ArrayList<double>`，只有`ArrayList<Double>`。因为当类型擦除后，`ArrayList`的原始类型变为`Object`，但是`Object`类型不能存储`double`值，只能引用`Double`的值。

#### 5、编译时集合的instanceof

```java
ArrayList<String> arrayList = new ArrayList<String>();
```

因为类型擦除之后，`ArrayList<String>`只剩下原始类型，泛型信息`String`不存在了。

那么，编译时进行类型查询的时候使用下面的方法是错误的

```java
if( arrayList instanceof ArrayList<String>)
```

#### 6、泛型在静态方法和静态类中的问题

泛型类中的静态方法和静态变量不可以使用泛型类所声明的泛型类型参数

举例说明：

```java
public class Test2<T> {    
    public static T one;   //编译错误    
    public static  T show(T one){ //编译错误    
        return null;    
    }    
}
```

因为泛型类中的泛型参数的实例化是在定义对象的时候指定的，而静态变量和静态方法不需要使用对象来调用。对象都没有创建，如何确定这个泛型参数是何种类型，所以当然是错误的。

但是要注意区分下面的一种情况：

```java
public class Test2<T> {    

    public static <T >T show(T one){ //这是正确的    
        return null;    
    }    
}
```

因为这是一个泛型方法，在泛型方法中使用的T是自己在方法中定义的 T，而不是泛型类中的T。





# 代理

### 1 静态代理

- 定义：通过在代码中显示定义了一个业务实现类的代理，在代理类中实现了同名的被代理类的方法，通过调用代理类的方法，实现对被代理类方法的增强。代理和被代理对象在代理在代理之前是确定的，他们都是实现的相同的接口或者继承相同的抽象类

![preview](https://segmentfault.com/img/bVby06d?w=1063&h=464/view)

```java
/**
 * 被代理的接口类
 * @author zhiyuan.shen
 */
public interface Subject {

    /**
     * 具体方法
     */
    void doAction();

}

/**
 * @author zhiyuan.shen
 */
public class RealSubject implements Subject {
    @Override
    public void doAction() {
        System.out.println("service impl class.");
    }
}

/**
 * 代理类
 * @author zhiyuan.shen
 */
public class Proxy implements Subject {

    private Subject subject;

    public Proxy(Subject subject) {
        this.subject = subject;
    }

    @Override
    public void doAction () {
        System.out.println("before");
        subject.doAction();
        System.out.println("after");
    }

}
```

- 测试与应用类

```java
/**
 * @author zhiyuan.shen
 */
public class Test {

    public static void main(String[] args) {
        //创建服务类
        RealSubject realSubject = new RealSubject();
        //自己执行方法
        realSubject.doAction();

        System.out.println("----------");

        //创建代理类
        Proxy proxy = new Proxy(realSubject);
        //代理执行
        proxy.doAction();

    }
}
```

- 输出结果

```asciidoc
service impl class.
----------
before
service impl class.
after
```

- 静态代理角色介绍
  - 共同接口：真实的对象和代理类共同实现的接口，规范方法定义。
  - 真实对象：实现共同接口，可以独立运行，具备完整功能的对象。
  - 代理类：对真实对象的增强，组合了真实对象。

### 2 动态代理

- 定义：通过接口中的方法名在动态生成的代理中动态调用实现类中的同名方法，一定是接口。JDK动态代理利用了JDK API，动态地在内存中构建代理对象，从而实现对目标对象的代理功能。

JDK中生成代理对象主要涉及的类和方法：

java.lang.reflect Proxy类，使用的方法：

```java
public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h)
```

这三个参数的含义：

- ClassLoader loader：目标对象的类加载器
- Class<?>[] interfaces：目标对象实现的接口
- InvocationHandler h：事件处理器，代理对象的具体代理操作

java.lang.reflect InvocationHandler接口，使用的方法：

```java
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable;
```

在invoke实现代理类的具体代理操作。

所谓Dynamic Proxy是这样一种class:

- 他是在运行时生成的class
- 该class需要实现一组interface
- 使用动态代理类时，必须实现InvocationHandler接口
- 实现JDK动态代理的步骤
  1. 创建一个实现接口InvocationHandler的类，它必须实现invoke方法
  2. 创建被代理的类以及接口
  3. 调用Proxy的静态方法，创建一个代理类 newProxyInstance(ClassLoader loader, Class[] interfaces, InvocationHandler h)
  4. 通过代理调用方法
- 示例

```java
/**
 * 移动的车接口
 * @author zhiyuan.shen 
 */
public interface MoveAble {
    void move();
}

/**
 * 具体的实现类
 * @author zhiyuan.shen
 */
public class Car implements MoveAble {
    /**
     * 实现开车
     */
    @Override
    public void move() {
        try {
            System.out.println("car moving.");
            Thread.sleep(new Random().nextInt(1000));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

/**
 * 时间处理器
 * @author zhiyuan.shen
 */
public class TimeHandler implements InvocationHandler{

    private Object target;

    public TimeHandler(Object target) {
        this.target = target;
    }

    /**
     *
     * @param proxy 被代理的对象
     * @param method 被代理对象的方法
     * @param args 被代理方法的参数
     * @return 方法的返回值
     * @throws Throwable
     */
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        long startTime = System.currentTimeMillis();
        System.out.println("car start...");
        method.invoke(target);
        System.out.println("car end...");
        long endTime = System.currentTimeMillis();
        System.out.println("used " + (endTime - startTime) + " ms！");
        return null;
    }

}
```

- 测试与应用类

```java
public class Main {

    public static void main(String[] args) {
        Car car = new Car();

        InvocationHandler handler = new TimeHandler(car);
        Class<?> clazz = car.getClass();
        /**
         * loader 类加载器
         * interfaces 实现接口
         * InvocationHandler
         *
         * ---动态代理实现思路---
         * 实现功能：通过Proxy的newProxyInstance返回代理对象
         * 1.声明一段源码（动态产生代理）
         * 2.编译源码（JDK Complider API）, 产生一个新的类（代理类）
         * 3.将这个类load到内存当中，产生一个新的对象（代理对象）
         * 4.return 代理对象
         */
        MoveAble moveAble = (MoveAble)Proxy.newProxyInstance(clazz.getClassLoader(), clazz.getInterfaces(), handler);
        moveAble.move();
    }
}
```

- 输出结果

```gams
car start...
car end...
used 120 ms?
```

### 3 CGlib代理

- 定义：通过继承，生成的代理类是业务类的子类，重写父类的方法实现代理。CGLIB（CODE GENERLIZE LIBRARY）代理是针对类实现代理，主要是对指定的类生成一个子类，覆盖其中的所有方法，所以该类或方法不能声明称final的。
- cglib特点
  - JDK的动态代理有一个限制，就是**使用动态代理的对象必须实现一个或多个接口**。如果**想代理没有实现接口的类，就可以使用CGLIB实现**。
  - CGLIB是一个强大的高性能的代码生成包，它可以在运行期扩展Java类与实现Java接口。
  - 它广泛的被许多AOP的框架使用，例如Spring AOP和dynaop，为他们提供方法的interception（拦截）。
  - CGLIB包的底层是通过使用一个小而快的字节码处理框架ASM，来转换字节码并生成新的类。
- 示例：cglib动态代理需要引入cglib-full-2.0.2.jar包

### 4.JDK动态代理和CGLIB动态代理区别

- JDK动态代理：
  1. 只能代理实现了接口的类
  2. 没有实现接口的类不能实现JDK动态代理
- CGLIB动态代理：
  1. 针对类来实现代理
  2. 对指定目标类产生一个子类，通过方法拦截技术拦截所有父类方法的调用
  3. 不能对final的类代理

### 5 动态代理用多了之后对内存方面有什么影响？



# 序列化

### 1 简述对java序列化的理解？

![image-20210813231944898](D:\1书本笔记\java实战项目\image-20210813231944898.png)

Java平台允许我们在内存中创建可复用的Java对象，但一般情况下，只有当JVM处于运行时，这些对象才可能存在，即，这些对象的生命周期不会比JVM的生命周期更长。但在现实应用中，就可能要求在JVM停止运行之后能够保存指定的对象，并在将来重新读取被保存的对象。Java对象序列化就能够帮助我们实现该功能。
 使用Java对象序列化，在保存对象时，会把其状态保存为一组字节，在未来，再将这些字节组装成对象。必须注意地是，对象序列化保存的是对象的"状态"，即它的成员变量。由此可知，对象序列化不会关注类中的静态变量。
 除了在持久化对象时会用到对象序列化之外，当使用RMI(远程方法调用)，或在网络中传递对象时，都会用到对象序列化



### 2 简述 java序列化的相关特性？





### 3 简述java序列化的几种方式？

#### 1 [Serializable](https://www.cnblogs.com/9dragon/p/10901448.html#h1serializable)

##### 1.1 普通序列化

Serializable接口是一个标记接口，不用实现任何方法。一旦实现了此接口，该类的对象就是可序列化的。

1. **序列化步骤：**

- **步骤一：创建一个ObjectOutputStream输出流；**

- **步骤二：调用ObjectOutputStream对象的writeObject输出可序列化对象。**

  ```java
  public class Person implements Serializable {
    private String name;
    private int age;
    //我不提供无参构造器
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
  
    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
  }
  
  public class WriteObject {
    public static void main(String[] args) {
        try (//创建一个ObjectOutputStream输出流
             ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("object.txt"))) {
            //将对象序列化到文件s
            Person person = new Person("9龙", 23);
            oos.writeObject(person);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
  }
  ```

1. **反序列化步骤：**

- **步骤一：创建一个ObjectInputStream输入流；**

- **步骤二：调用ObjectInputStream对象的readObject()得到序列化的对象。**

  我们将上面序列化到person.txt的person对象反序列化回来

  ```java
  public class Person implements Serializable {
    private String name;
    private int age;
    //我不提供无参构造器
    public Person(String name, int age) {
        System.out.println("反序列化，你调用我了吗？");
        this.name = name;
        this.age = age;
    }
  
    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
  }
  
  public class ReadObject {
    public static void main(String[] args) {
        try (//创建一个ObjectInputStream输入流
             ObjectInputStream ois = new ObjectInputStream(new FileInputStream("person.txt"))) {
            Person brady = (Person) ois.readObject();
            System.out.println(brady);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
  }
  //输出结果
  //Person{name='9龙', age=23}
  ```

  **waht???? 输出告诉我们，反序列化并不会调用构造方法。反序列的对象是由JVM自己生成的对象，不通过构造方法生成。**

##### 1.2 成员是引用的序列化

**如果一个可序列化的类的成员不是基本类型，也不是String类型，那这个引用类型也必须是可序列化的；否则，会导致此类不能序列化。**

看例子，我们新增一个Teacher类。将Person去掉实现Serializable接口代码。

```java
public class Person{
    //省略相关属性与方法
}
public class Teacher implements Serializable {

    private String name;
    private Person person;

    public Teacher(String name, Person person) {
        this.name = name;
        this.person = person;
    }

     public static void main(String[] args) throws Exception {
        try (ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("teacher.txt"))) {
            Person person = new Person("路飞", 20);
            Teacher teacher = new Teacher("雷利", person);
            oos.writeObject(teacher);
        }
    }
}
```

![img](https://img2018.cnblogs.com/blog/1603499/201905/1603499-20190521180304399-894547036.jpg)

我们看到程序直接报错，因为Person类的对象是不可序列化的，这导致了Teacher的对象不可序列化

##### 1.3 同一对象序列化多次的机制

**同一对象序列化多次，会将这个对象序列化多次吗？**答案是**否定**的。

```Java
public class WriteTeacher {
    public static void main(String[] args) throws Exception {
        try (ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("teacher.txt"))) {
            Person person = new Person("路飞", 20);
            Teacher t1 = new Teacher("雷利", person);
            Teacher t2 = new Teacher("红发香克斯", person);
            //依次将4个对象写入输入流
            oos.writeObject(t1);
            oos.writeObject(t2);
            oos.writeObject(person);
            oos.writeObject(t2);
        }
    }
}
```

依次将t1、t2、person、t2对象序列化到文件teacher.txt文件中。

**注意：反序列化的顺序与序列化时的顺序一致**。

```java
public class ReadTeacher {
    public static void main(String[] args) {
        try (ObjectInputStream ois = new ObjectInputStream(new FileInputStream("teacher.txt"))) {
            Teacher t1 = (Teacher) ois.readObject();
            Teacher t2 = (Teacher) ois.readObject();
            Person p = (Person) ois.readObject();
            Teacher t3 = (Teacher) ois.readObject();
            System.out.println(t1 == t2);
            System.out.println(t1.getPerson() == p);
            System.out.println(t2.getPerson() == p);
            System.out.println(t2 == t3);
            System.out.println(t1.getPerson() == t2.getPerson());
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
//输出结果
//false
//true
//true
//true
//true
```

从输出结果可以看出，**Java序列化同一对象，并不会将此对象序列化多次得到多个对象。**

- **Java序列化算法**

1. **所有保存到磁盘的对象都有一个序列化编码号**

2. **当程序试图序列化一个对象时，会先检查此对象是否已经序列化过，只有此对象从未（在此虚拟机）被序列化过，才会将此对象序列化为字节序列输出。**

3. **如果此对象已经序列化过，则直接输出编号即可。**

   图示上述序列化过程。

![img](https://img2018.cnblogs.com/blog/1603499/201905/1603499-20190521180352659-740977206.jpg)



##### 1.4 java序列化算法潜在的问题

由于java序利化算法不会重复序列化同一个对象，只会记录已序列化对象的编号。**如果序列化一个可变对象（对象内的内容可更改）后，更改了对象内容，再次序列化，并不会再次将此对象转换为字节序列，而只是保存序列化编号。**

```java
public class WriteObject {
    public static void main(String[] args) {
        try (ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("person.txt"));
             ObjectInputStream ios = new ObjectInputStream(new FileInputStream("person.txt"))) {
            //第一次序列化person
            Person person = new Person("9龙", 23);
            oos.writeObject(person);
            System.out.println(person);

            //修改name
            person.setName("海贼王");
            System.out.println(person);
            //第二次序列化person
            oos.writeObject(person);

            //依次反序列化出p1、p2
            Person p1 = (Person) ios.readObject();
            Person p2 = (Person) ios.readObject();
            System.out.println(p1 == p2);
            System.out.println(p1.getName().equals(p2.getName()));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
//输出结果
//Person{name='9龙', age=23}
//Person{name='海贼王', age=23}
//true
//true
```

##### 1.5 可选的自定义序列化

1. 有些时候，我们有这样的需求，某些属性不需要序列化。**使用transient关键字选择不需要序列化的字段。**

   ```java
   public class Person implements Serializable {
      //不需要序列化名字与年龄
      private transient String name;
      private transient int age;
      private int height;
      private transient boolean singlehood;
      public Person(String name, int age) {
          this.name = name;
          this.age = age;
      }
      //省略get,set方法
   }
   
   public class TransientTest {
      public static void main(String[] args) throws Exception {
          try (ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("person.txt"));
               ObjectInputStream ios = new ObjectInputStream(new FileInputStream("person.txt"))) {
              Person person = new Person("9龙", 23);
              person.setHeight(185);
              System.out.println(person);
              oos.writeObject(person);
              Person p1 = (Person)ios.readObject();
              System.out.println(p1);
          }
      }
   }
   //输出结果
   //Person{name='9龙', age=23', singlehood=true', height=185cm}
   //Person{name='null', age=0', singlehood=false', height=185cm}
   ```

   从输出我们看到，**使用transient修饰的属性，java序列化时，会忽略掉此字段，所以反序列化出的对象，被transient修饰的属性是默认值。对于引用类型，值是null；基本类型，值是0；boolean类型，值是false。**

2. 使用transient虽然简单，但将此属性完全隔离在了序列化之外。java提供了**可选的自定义序列化。**可以进行控制序列化的方式，或者对序列化数据进行编码加密等。

   ```java
   private void writeObject(java.io.ObjectOutputStream out) throws IOException；
   private void readObject(java.io.ObjectIutputStream in) throws IOException,ClassNotFoundException;
   private void readObjectNoData() throws ObjectStreamException;
   ```

   通过重写writeObject与readObject方法，可以自己选择哪些属性需要序列化， 哪些属性不需要。如果writeObject使用某种规则序列化，则相应的readObject需要相反的规则反序列化，以便能正确反序列化出对象。这里展示对名字进行反转加密。

   ```java
   public class Person implements Serializable {
      private String name;
      private int age;
      //省略构造方法，get及set方法
   
      private void writeObject(ObjectOutputStream out) throws IOException {
          //将名字反转写入二进制流
          out.writeObject(new StringBuffer(this.name).reverse());
          out.writeInt(age);
      }
   
      private void readObject(ObjectInputStream ins) throws IOException,ClassNotFoundException{
          //将读出的字符串反转恢复回来
          this.name = ((StringBuffer)ins.readObject()).reverse().toString();
          this.age = ins.readInt();
      }
   }
   ```

   当序列化流不完整时，readObjectNoData()方法可以用来正确地初始化反序列化的对象。例如，使用不同类接收反序列化对象，或者序列化流被篡改时，系统都会调用readObjectNoData()方法来初始化反序

#### 2 [Externalizable：强制自定义序列化](https://www.cnblogs.com/9dragon/p/10901448.html#h2externalizable)

通过实现Externalizable接口，必须实现writeExternal、readExternal方法。

```java
public interface Externalizable extends java.io.Serializable {
     void writeExternal(ObjectOutput out) throws IOException;
     void readExternal(ObjectInput in) throws IOException, ClassNotFoundException;
}
public class ExPerson implements Externalizable {

    private String name;
    private int age;
    //注意，必须加上pulic 无参构造器
    public ExPerson() {
    }

    public ExPerson(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public void writeExternal(ObjectOutput out) throws IOException {
        //将name反转后写入二进制流
        StringBuffer reverse = new StringBuffer(name).reverse();
        System.out.println(reverse.toString());
        out.writeObject(reverse);
        out.writeInt(age);
    }

    @Override
    public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {
        //将读取的字符串反转后赋值给name实例变量
        this.name = ((StringBuffer) in.readObject()).reverse().toString();
        System.out.println(name);
        this.age = in.readInt();
    }

    public static void main(String[] args) throws IOException, ClassNotFoundException {
        try (ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("ExPerson.txt"));
             ObjectInputStream ois = new ObjectInputStream(new FileInputStream("ExPerson.txt"))) {
            oos.writeObject(new ExPerson("brady", 23));
            ExPerson ep = (ExPerson) ois.readObject();
            System.out.println(ep);
        }
    }
}
//输出结果
//ydarb
//brady
//ExPerson{name='brady', age=23}
```

**注意：Externalizable接口不同于Serializable接口，实现此接口必须实现接口中的两个方法实现自定义序列化，这是强制性的；特别之处是必须提供pulic的无参构造器，因为在反序列化的时候需要反射创建对象。**

#### 3 两种序列化对比

![image-20210813231844168](D:\1书本笔记\java实战项目\image-20210813231844168.png)



### 4 java序列化后serialVersionUid的作用

serialVersionUID适用于JAVA的序列化机制。简单来说，Java的序列化机制是通过判断类的serialVersionUID来验证版本一致性的。
 在进行反序列化时，JVM会把传来的字节流中的serialVersionUID与本地相应实体类的serialVersionUID进行比较，如果相同就认为是一致的，可以进行反序列化，否则就会出现序列化版本不一致的异常，即是InvalidCastException。·

![image-20210813231302630](D:\1书本笔记\java实战项目\image-20210813231302630.png)

### 5 Java序列化背后的源码？

简单的对象，对于不想序列化的字段，只要声明为`transient`就好。而有时候，我想对部分字段处理后序列化。比如ArrayList中存储数据的`transient Object[] elementData;`。我们知道ArrayList是可以序列化的，根源就在于自定义这里了。下面跟踪`ObjectOutputStream`源码，知道自定义的执行部分就可以验证了。

入口： java.io.ObjectOutputStream#writeObject



```
public final void writeObject(Object obj) throws IOException {
    if (enableOverride) {
        writeObjectOverride(obj);
        return;
    }
    try {
        writeObject0(obj, false);
    } catch (IOException ex) {
        if (depth == 0) {
            writeFatalException(ex);
        }
        throw ex;
    }
}
```

然后，核心方法

```
private void writeObject0(Object obj, boolean unshared)
        throws IOException{
    boolean oldMode = bout.setBlockDataMode(false);depth++;
    try {
        //省略若干行
        for (;;) {
            // 省略若干行
            desc = ObjectStreamClass.lookup(cl, true);
            //省略若干行
        }
        //省略若干行
        if (obj instanceof String) {
            writeString((String) obj, unshared);
        } else if (cl.isArray()) {
            writeArray(obj, desc, unshared);
        } else if (obj instanceof Enum) {
            writeEnum((Enum<?>) obj, desc, unshared);
        } else if (obj instanceof Serializable) {
            writeOrdinaryObject(obj, desc, unshared);
        } else {
            //....
        }
    } finally {
        depth--;
        bout.setBlockDataMode(oldMode);
    }
}
```

这里，显然可以看到真正的执行序列化代码是`writeOrdinaryObject(obj, desc, unshared);`。 但直接追踪进去发现里面有许多初始化的字段是在之前做的处理。因此，先卖个关子，看前面初始化的部分，只找到我们想要初始化的字段即可。

进入`desc = ObjectStreamClass.lookup(cl, true);`



```
static ObjectStreamClass lookup(Class<?> cl, boolean all) {
    //省略若干行
    if (entry == null) {
        try {
            entry = new ObjectStreamClass(cl);
        } catch (Throwable th) {
            entry = th;
        }
        //.....
    }
    //省略若干行
}
```

进入`entry = new ObjectStreamClass(cl);`这里就是真正的初始化地方，前面省略的代码是缓存处理，当然缓存使用的ConcurrentHashMap。



```
private ObjectStreamClass(final Class<?> cl) {
    //省略无数行以及括号
    writeObjectMethod = getPrivateMethod(cl, "writeObject",
                            new Class<?>[] { ObjectOutputStream.class },
                            Void.TYPE);
    readObjectMethod = getPrivateMethod(cl, "readObject",
                            new Class<?>[] { ObjectInputStream.class },
                            Void.TYPE);
    //省略无数行
```

没错，费了这么大劲就是为了找到这两个method。通过反射，获取到目标class的两个私有方法`writeObject`, `readObject`。这两个就是自定义方法所在。

初始化完毕之后，我们再来继续序列化的代码. 回到刚才的核心方法，找到`writeOrdinaryObject(obj, desc, unshared);`， 进入，然后，继续找到`writeSerialData(obj, desc);`, 到这里就是真正执行序列化的代码了。



```
private void writeSerialData(Object obj, ObjectStreamClass desc)
        throws IOException
{
    ObjectStreamClass.ClassDataSlot[] slots = desc.getClassDataLayout();
    for (int i = 0; i < slots.length; i++) {
        ObjectStreamClass slotDesc = slots[i].desc;
        if (slotDesc.hasWriteObjectMethod()) {
            //....
            try {
                curContext = new SerialCallbackContext(obj, slotDesc);
                bout.setBlockDataMode(true);
                slotDesc.invokeWriteObject(obj, this);
                bout.setBlockDataMode(false);
                bout.writeByte(TC_ENDBLOCKDATA);
            } finally {
                //...
            }

            curPut = oldPut;
        } else {
            defaultWriteFields(obj, slotDesc);
        }
    }
}
```

显然，判断`writeObject`这个method是否初始化了，如果有，则直接调用这个方法，没有则默认处理。到此，跟踪完毕，我想要自定义序列化只要重写`writeObject`, `readObject`这两个方法即可。



### 6 ArrayList中的序列化的认识?

**1、序列化**

从上面序列化的工作流程可以看出，要想序列化对象，使用ObjectOutputStream对象输出流的writeObject()方法写入对象状态信息，即可使用readObject()方法读取信息。

那是不是可以在ArrayList中调用ObjectOutputStream对象的writeObject()方法将elementData的值写入输出流呢？

见源码：

```
private void writeObject(java.io.ObjectOutputStream s) throws java.io.IOException
{
    // Write out element count, and any hidden stuff
    int expectedModCount = modCount;
    s.defaultWriteObject();
    // Write out size as capacity for behavioural compatibility with clone()
    s.writeInt(size);
    // Write out all elements in the proper order.
    for (int i = 0; i < size; i++)
    {
        s.writeObject(elementData[i]);
    }
    if (modCount != expectedModCount)
    {
        throw new ConcurrentModificationException();
    }
}
```

虽然**elementData被transient修饰**，不能被序列化，但是我们可以将它的值取出来，然后将该值写入输出流。

```
// 片段1 它的功能等价于片段2
s.writeObject(elementData[i]);  // 传值时，是将实参elementData[i]赋给s.writeObject()的形参
//  片段2
Object temp = new Object();     // temp并没有被transient修饰
temp = elementData[i];
s.writeObject(temp);
```

### 7 Java序列化中的父类相关问题？

父类实现了Serializable，子类不需要实现Serializable

 相关注意事项:

  a）序列化时，只对对象的状态进行保存，而不管对象的方法；

  b）当一个父类实现序列化，子类自动实现序列化，不需要显式实现Serializable接口；

  c）当一个对象的实例变量引用其他对象，序列化该对象时也把引用对象进行序列化；

  d）并非所有的对象都可以序列化，至于为什么不可以，有很多原因了,比如：

​     1.安全方面的原因，比如一个对象拥有private，public等field，对于一个要传输的对象，比如写到文件，或者进行rmi传输等等，在序列化进行传输的过程中，这个对象的private等域是不受保护的。

    2.  资源分配方面的原因，比如socket，thread类，如果可以序列化，进行传输或者保存，也无法对他们进行重新的资源分配，而且，也是没有必要这样实现。



# 枚举

### 1 枚举的理解？

​			枚举类型是Java 5中新增特性的一部分，它是一种特殊的数据类型，之所以特殊是因为**它既是一种类(class)类型却又比类类型多了些特殊的约束**，但是这些约束的存在也造就**了枚举类型的简洁性、安全性以及便捷性**。

### 2 枚举和常量类的区别？

![image-20210814103525789](D:\1书本笔记\java实战项目\image-20210814103525789.png)

![image-20210814103555679](D:\1书本笔记\java实战项目\image-20210814103555679.png)

![image-20210814103613095](D:\1书本笔记\java实战项目\image-20210814103613095.png)

![image-20210814103621143](D:\1书本笔记\java实战项目\image-20210814103621143.png)

###  3 枚举的本质

你是否被问到过以下的问题：

> 1.枚举允许继承类吗？
> 2.枚举允许实现接口吗？
> 3.枚举可以用等号比较吗？
> 4.可以继承枚举吗？
> 5.枚举可以实现单例模式吗？
> \6. 当使用compareTo()比较枚举时，比较的是什么？
> \7. 当使用equals()比较枚举的时候，比较的是什么？

面试官的问题五花八门，但归根结底都是在考察同一个问题：枚举的本质。我们先来写一个简单的枚举类：

```java
public enum Fruit{
    APPLE(1),ORANGE(2),BANANA(3);
    int code;

    Fruit(int code){
        this.code=code;
    }
}
```



使用Jad命令反编译Fruit.class文件之后，可以得到如下内容：

```java
public final class Fruit extends Enum
{

    public static Fruit[] values()
    {
        return (Fruit[])$VALUES.clone();
    }

    public static Fruit valueOf(String s)
    {
        return (Fruit)Enum.valueOf(Fruit, s);
    }

    private Fruit(String s, int i, int j)
    {
        super(s, i);
        code = j;
    }

    public static final Fruit APPLE;
    public static final Fruit ORANGE;
    public static final Fruit BANANA;
    int code;
    private static final Fruit $VALUES[];

    static
    {
        APPLE = new Fruit("APPLE", 0, 1);
        ORANGE = new Fruit("ORANGE", 1, 2);
        BANANA = new Fruit("BANANA", 2, 3);
        $VALUES = (new Fruit[] {
            APPLE, ORANGE, BANANA
        });
    }
}
```



可见，Jvm编译器背地里是使用上面的方式来处理枚举的。它做了几件事：

- *定义一个继承自Enum类的Fruit类，Fruit类是用final修饰的*
- *为每个枚举实例对应创建一个类对象，这些类对象是用public static final修饰的。同时生成一个数组，用于保存全部的类对象*
- *生成一个静态代码块，用于初始化类对象和类对象数组*
- *生成一个构造函数，构造函数包含自定义参数和两个默认参数（下文会讲解这两个默认参数）*
- *生成一个静态的values()方法，用于返回所有的类对象*
- *生成一个静态的valueOf()方法，根据name参数返回对应的类实例（下文会讲解name参数）*



关于最基本的Enum类，[Jdk文档](https://link.zhihu.com/?target=http%3A//tool.oschina.net/apidocs/apidoc%3Fapi%3Djdk_7u4)是这样描述的：

> This is the common base class of all Java language enumeration types. More information about enums, including descriptions of the implicitly declared methods synthesized by the compiler, can be found in section 8.9 ofThe Java™ Language Specification.
> 这是Java语言中所有枚举类型的基础类，更多的关于枚举的信息，包括一些编译器隐式声明的方法的描述，可以在[The Java™ Language Specification文档的8.9小节](https://link.zhihu.com/?target=https%3A//docs.oracle.com/javase/specs/jls/se7/html/jls-8.html%23jls-8.9)找到。

话不多说，来看源码：

```java
public abstract class Enum<E extends java.lang.Enum<E>>
        implements Comparable<E>, Serializable {
    private final String name;
    private final int ordinal;
    
    public final String name() {
        return name;
    }
    
    public final int ordinal() {
        return ordinal;
    }
    
    protected Enum(String name, int ordinal) {
        this.name = name;
        this.ordinal = ordinal;
    }
    public String toString() {
        return name;
    }
    public final boolean equals(Object other) {
        return this == other;
    }
    public final int hashCode() {
        return super.hashCode();
    }
    protected final Object clone() throws CloneNotSupportedException {
        throw new CloneNotSupportedException();
    }

    public final int compareTo(E o) {
        java.lang.Enum other = (java.lang.Enum) o;
        java.lang.Enum self = this;
        if (self.getClass() != other.getClass() && // optimization
                self.getDeclaringClass() != other.getDeclaringClass())
            throw new ClassCastException();
        return self.ordinal - other.ordinal;
    }

    public final Class<E> getDeclaringClass() {
        Class clazz = getClass();
        Class zuper = clazz.getSuperclass();
        return (zuper == java.lang.Enum.class) ? clazz : zuper;
    }

    public static <T extends java.lang.Enum<T>> T valueOf(Class<T> enumType,
                                                          String name) {
        T result = enumType.enumConstantDirectory().get(name);
        if (result != null)
            return result;
        if (name == null)
            throw new NullPointerException("Name is null");
        throw new IllegalArgumentException(
                "No enum constant " + enumType.getCanonicalName() + "." + name);
    }

    protected final void finalize() {
    }

    private void readObject(ObjectInputStream in) throws IOException,
            ClassNotFoundException {
        throw new InvalidObjectException("can't deserialize enum");
    }

    private void readObjectNoData() throws ObjectStreamException {
        throw new InvalidObjectException("can't deserialize enum");
    }
}
```



在Enum源代码中，有以下几个值得关注的点：

- *Enum类有两个成员变量：name和ordinal。其中，name用于记录枚举常量的名字。比如APPLE、ORANGE和BANANA。ordinal用于记录枚举常量在声明时的顺序(从0开始)。比如APPLE是0、ORANGE是1、BANANA是2。*
- *Enum类有一个构造函数，它有两个入参，分别为name和ordianl赋值。*
- *Enum类重写了toString()方法，返回枚举常量的name值。*
- *Enum类重写了equals()方法，直接用等号比较。*
- *Enum类不允许克隆，clone()方法直接抛出异常。（保证枚举永远是单例的）*
- *Enum类实现了Comparable接口，直接比较枚举常量的ordinal的值。*
- *Enum类有一个静态的valueOf()方法，可以根据枚举类型以及name返回对应的枚举常量。*
- *Enum类不允许反序列化，为了保证枚举永远是单例的。*



读到这里，来看之前几题小题的答案。

> \1. 枚举不允许继承类。Jvm在生成枚举时已经继承了Enum类，由于Java语言是单继承，不支持再继承额外的类（唯一的继承名额被Jvm用了）。
> \2. 枚举允许实现接口。因为枚举本身就是一个类，类是可以实现多个接口的。
> \3. 枚举可以用等号比较。Jvm会为每个枚举实例对应生成一个类对象，这个类对象是用public static final修饰的，在static代码块中初始化，是一个单例。
> \4. 不可以继承枚举。因为Jvm在生成枚举类时，将它声明为final。
> \5. 枚举本身就是一种对单例设计模式友好的形式，它是实现单例模式的一种很好的方式。
> \6. 枚举类型的compareTo()方法比较的是枚举类对象的ordinal的值。
> \7. 枚举类型的equals()方法比较的是枚举类对象的内存地址，作用与等号等价。

### 4 枚举和序列化的关系？

**大概意思是枚举类只可以序列化，不可以反序列化。**

1. 为了保证枚举类型像Java规范中所说的那样，每一个枚举类型极其定义的枚举变量在JVM中都是唯一的，在枚举类型的序列化和反序列化上，Java做了特殊的规定。英文原文我就不贴了。

2. 大概意思就是说，在序列化的时候Java仅仅是将枚举对象的name属性输出到结果中，反序列化的时候则是通过java.lang.Enum的valueOf方法来根据名字查找枚举对象。同时，编译器是不允许任何对这种序列化机制的定制的，因此禁用了writeObject、readObject、readObjectNoData、writeReplace和readResolve等方法。 



# I/O

### 1 简述I/O是什么？I/O的大致分类

![image-20210817160355367](D:\1书本笔记\java实战项目\image-20210817160355367.png)

![image-20210817160408029](D:\1书本笔记\java实战项目\image-20210817160408029.png)

### 2 Unix中的5种I/O模型？

**Blocking IO - 阻塞IO**

**NoneBlocking IO - 非阻塞IO**

**IO multiplexing - IO多路复用**

**signal driven IO - 信号驱动IO**

**asynchronous IO - 异步IO**

在讨论之前先说明一下IO发生时涉及到的对象和步骤，对于一个network IO，它会涉及到两个系统对象：

- **application** 调用这个IO的进程
- **kernel** 系统内核

那他们经历的两个交互过程是：

- **阶段1 wait for data** 等待数据准备
- **阶段2 copy data from kernel to user** 将数据从内核拷贝到用户进程中

之所以会有同步、异步、阻塞和非阻塞这几种说法就是根据程序在这两个阶段的处理方式不同而产生的。了解了这些背景之后，我们就分别针对四种IO模型进行讲解



#### 1  Blocking IO - 阻塞IO



![img](https://upload-images.jianshu.io/upload_images/11224747-02876ed5afe356ff.gif?imageMogr2/auto-orient/strip|imageView2/2/w/552/format/webp)

​			当用户进程调用了recvfrom这个系统调用，kernel就开始了IO的第一个阶段：准备数据。对于network IO来说，很多时候数据在一开始还没有到达（比如，还没有收到一个完整的UDP包），这个时候kernel就要等待足够的数据到来。而在用户进程这边，整个进程会被阻塞。当kernel一直等到数据准备好了，它就会将数据从kernel中拷贝到用户内存，然后kernel返回结果，用户进程才解除block的状态，重新运行起来。

所以，blocking IO的特点就是在IO执行的两个阶段都被block了。

#### 2  NoneBlockingIO - 非阻塞IO

linux下，可以通过设置socket使其变为non-blocking。当对一个non-blocking socket执行读操作时，流程是这个样子：

![img](https://upload-images.jianshu.io/upload_images/11224747-e4c17ed342162afc.gif?imageMogr2/auto-orient/strip|imageView2/2/w/603/format/webp)



从图中可以看出，当用户进程发出recvfrom这个系统调用后，如果kernel中的数据还没有准备好，那么它并不会block用户进程，而是立刻返回一个结果（no datagram ready）。从用户进程角度讲 ，它发起一个操作后，并没有等待，而是马上就得到了一个结果。用户进程得知数据还没有准备好后，它可以每隔一段时间再次发送recvfrom操作。一旦kernel中的数据准备好了，并且又再次收到了用户进程的system call，那么它马上就将数据拷贝到了用户内存，然后返回。

所以，用户进程其实是需要不断的主动询问kernel数据好了没有。



#### 3  IO multiplexing - IO多路复用

I/O多路复用(multiplexing)是网络编程中最常用的模型，像我们最常用的select、epoll都属于这种模型。以select为例：

![img](https://upload-images.jianshu.io/upload_images/11224747-dcc024f7fa2e8460.gif?imageMogr2/auto-orient/strip|imageView2/2/w/609/format/webp)

看起来它与blocking I/O很相似，两个阶段都阻塞。但它与blocking I/O的一个重要区别就是它可以等待多个数据报就绪（datagram ready），即可以处理多个连接。这里的select相当于一个“代理”，调用select以后进程会被select阻塞，这时候在内核空间内select会监听指定的多个datagram (如socket连接)，如果其中任意一个数据就绪了就返回。此时程序再进行数据读取操作，将数据拷贝至当前进程内。由于select可以监听多个socket，我们可以用它来处理多个连接。

在select模型中每个socket一般都设置成non-blocking，虽然等待数据阶段仍然是阻塞状态，但是它是被select调用阻塞的，而不是直接被I/O阻塞的。select底层通过轮询机制来判断每个socket读写是否就绪。

当然select也有一些缺点，比如底层轮询机制会增加开销、支持的文件描述符数量过少等。为此，Linux引入了epoll作为select的改进版本。

#### 4  asynchronous IO - 异步IO

异步I/O在网络编程中几乎用不到，在File I/O中可能会用到：

![img](https://upload-images.jianshu.io/upload_images/11224747-05e3a70e98d2331e.gif?imageMogr2/auto-orient/strip|imageView2/2/w/572/format/webp)

​		这里面的读取操作的语义与上面的几种模型都不同。这里的读取操作(aio_read)会通知内核进行读取操作并将数据拷贝至进程中，完事后通知进程整个操作全部完成（绑定一个回调函数处理数据）。读取操作会立刻返回，程序可以进行其它的操作，所有的读取、拷贝工作都由内核去做，做完以后通知进程，进程调用绑定的回调函数来处理数据

#### 先来说阻塞和非阻塞：

- 阻塞调用会一直等待远程数据就绪再返回，即上面的**阶段1**会阻塞调用者，直到读取结束。
- 而非阻塞无论在什么情况下都会立即返回，虽然非阻塞大部分时间不会被block，但是它仍要求进程不断地去主动询问kernel是否准备好数据，也需要进程主动地再次调用recvfrom来将数据拷贝到用户内存。

#### 再说一说同步和异步：

- 同步方法会一直阻塞进程，直到I/O操作结束，注意这里相当于上面的**阶段1，阶段2**都会阻塞调用者。其中 Blocking IO - 阻塞IO，Nonblocking IO - 非阻塞IO，IO multiplexing - IO多路复用，signal driven IO - 信号驱动IO 这四种IO都可以归类为同步IO。
- 而异步方法不会阻塞调用者进程，即使是从内核空间的缓冲区将数据拷贝到进程中这一操作也不会阻塞进程，拷贝完毕后内核会通知进程数据拷贝结束。

### 3 Linux内核中的select\poll\epoll的工作原理？

**IO多路复用的三种机制Select，Poll，Epoll**

与多进程和多线程技术相比，**I/O多路复用技术的最大优势是系统开销小，系统不必创建进程/线程，也不必维护这些进程/线程，从而大大减小了系统的开销**。

在介绍select、poll、epoll之前，首先介绍一下Linux操作系统中**基础的概念**：

- **用户空间 / 内核空间**
   现在操作系统都是采用虚拟存储器，那么对32位操作系统而言，它的寻址空间（虚拟存储空间）为4G（2的32次方）。
   操作系统的核心是内核，独立于普通的应用程序，可以访问受保护的内存空间，也有访问底层硬件设备的所有权限。为了保证用户进程不能直接操作内核（kernel），保证内核的安全，操作系统将虚拟空间划分为两部分，一部分为内核空间，一部分为用户空间。
- **进程切换**
   为了控制进程的执行，内核必须有能力挂起正在CPU上运行的进程，并恢复以前挂起的某个进程的执行。这种行为被称为进程切换。因此可以说，任何进程都是在操作系统内核的支持下运行的，是与内核紧密相关的，并且进程切换是非常耗费资源的。
- **进程阻塞**
   正在执行的进程，由于期待的某些事件未发生，如请求系统资源失败、等待某种操作的完成、新数据尚未到达或无新工作做等，则由系统自动执行阻塞原语(Block)，使自己由运行状态变为阻塞状态。可见，进程的阻塞是进程自身的一种主动行为，也因此只有处于运行态的进程（获得了CPU资源），才可能将其转为阻塞状态。当进程进入阻塞状态，是不占用CPU资源的。
- **文件描述符**
   文件描述符（File descriptor）是计算机科学中的一个术语，是一个用于表述指向文件的引用的抽象化概念。
   文件描述符在形式上是一个非负整数。实际上，它是一个索引值，指向内核为每一个进程所维护的该进程打开文件的记录表。当程序打开一个现有文件或者创建一个新文件时，内核向进程返回一个文件描述符。在程序设计中，一些涉及底层的程序编写往往会围绕着文件描述符展开。但是文件描述符这一概念往往只适用于UNIX、Linux这样的操作系统。
- **缓存I/O**
   缓存I/O又称为标准I/O，大多数文件系统的默认I/O操作都是缓存I/O。在Linux的缓存I/O机制中，操作系统会将I/O的数据缓存在文件系统的页缓存中，即数据会先被拷贝到操作系统内核的缓冲区中，然后才会从操作系统内核的缓冲区拷贝到应用程序的地址空间。



#### 1 select

我们先分析一下select函数

```
int select(int maxfdp1,fd_set *readset,fd_set *writeset,fd_set *exceptset,const struct timeval *timeout);
```

**【参数说明】**
 **int maxfdp1** 指定待测试的文件描述字个数，它的值是待测试的最大描述字加1。
 **fd_set \*readset , fd_set \*writeset , fd_set \*exceptset**
 `fd_set`可以理解为一个集合，这个集合中存放的是文件描述符(file descriptor)，即文件句柄。中间的三个参数指定我们要让内核测试读、写和异常条件的文件描述符集合。如果对某一个的条件不感兴趣，就可以把它设为空指针。
 **const struct timeval \*timeout** `timeout`告知内核等待所指定文件描述符集合中的任何一个就绪可花多少时间。其timeval结构用于指定这段时间的秒数和微秒数。

**【返回值】**
 **int** 若有就绪描述符返回其数目，若超时则为0，若出错则为-1

##### select运行机制

select()的机制中提供一种`fd_set`的数据结构，实际上是一个long类型的数组，每一个数组元素都能与一打开的文件句柄（不管是Socket句柄,还是其他文件或命名管道或设备句柄）建立联系，建立联系的工作由程序员完成，当调用select()时，由内核根据IO状态修改fd_set的内容，由此来通知执行了select()的进程哪一Socket或文件可读。

从流程上来看，使用select函数进行IO请求和同步阻塞模型没有太大的区别，甚至还多了添加监视socket，以及调用select函数的额外操作，效率更差。但是，使用select以后最大的优势是用户可以在一个线程内同时处理多个socket的IO请求。用户可以注册多个socket，然后不断地调用select读取被激活的socket，即可达到在同一个线程内同时处理多个IO请求的目的。而在同步阻塞模型中，必须通过多线程的方式才能达到这个目的。

##### select机制的问题

1. 每次调用select，都需要把`fd_set`集合从用户态拷贝到内核态，如果`fd_set`集合很大时，那这个开销也很大
2. 同时每次调用select都需要在内核遍历传递进来的所有`fd_set`，如果`fd_set`集合很大时，那这个开销也很大
3. 为了减少数据拷贝带来的性能损坏，内核对被监控的`fd_set`集合大小做了限制，并且这个是通过宏控制的，大小不可改变(限制为1024)



#### 2  poll

poll的机制与select类似，与select在本质上没有多大差别，管理多个描述符也是进行轮询，根据描述符的状态进行处理，但是poll没有最大文件描述符数量的限制。也就是说，poll只解决了上面的问题3，并没有解决问题1，2的性能开销问题。

下面是pll的函数原型：



```cpp
int poll(struct pollfd *fds, nfds_t nfds, int timeout);

typedef struct pollfd {
        int fd;                         // 需要被检测或选择的文件描述符
        short events;                   // 对文件描述符fd上感兴趣的事件
        short revents;                  // 文件描述符fd上当前实际发生的事件
} pollfd_t;
```

poll改变了文件描述符集合的描述方式，使用了`pollfd`结构而不是select的`fd_set`结构，使得poll支持的文件描述符集合限制远大于select的1024

**【参数说明】**

**struct pollfd \*fds** `fds`是一个`struct pollfd`类型的数组，用于存放需要检测其状态的socket描述符，并且调用poll函数之后`fds`数组不会被清空；一个`pollfd`结构体表示一个被监视的文件描述符，通过传递`fds`指示 poll() 监视多个文件描述符。其中，结构体的`events`域是监视该文件描述符的事件掩码，由用户来设置这个域，结构体的`revents`域是文件描述符的操作结果事件掩码，内核在调用返回时设置这个域

**nfds_t nfds** 记录数组`fds`中描述符的总数量

**【返回值】**
 **int** 函数返回fds集合中就绪的读、写，或出错的描述符数量，返回0表示超时，返回-1表示出错；



#### 3  epoll

epoll在Linux2.6内核正式提出，是基于事件驱动的I/O方式，相对于select来说，epoll没有描述符个数限制，使用一个文件描述符管理多个描述符，将用户关心的文件描述符的事件存放到内核的一个事件表中，这样在用户空间和内核空间的copy只需一次。

Linux中提供的epoll相关函数如下：



```csharp
int epoll_create(int size);
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);
```

**1. epoll_create** 函数创建一个epoll句柄，参数`size`表明内核要监听的描述符数量。调用成功时返回一个epoll句柄描述符，失败时返回-1。

**2. epoll_ctl** 函数注册要监听的事件类型。四个参数解释如下：

- `epfd` 表示epoll句柄

- ```
  op
  ```

   表示fd操作类型，有如下3种

  - EPOLL_CTL_ADD   注册新的fd到epfd中
  - EPOLL_CTL_MOD 修改已注册的fd的监听事件
  - EPOLL_CTL_DEL 从epfd中删除一个fd

- `fd` 是要监听的描述符

- `event` 表示要监听的事件

epoll_event 结构体定义如下：



```cpp
struct epoll_event {
    __uint32_t events;  /* Epoll events */
    epoll_data_t data;  /* User data variable */
};

typedef union epoll_data {
    void *ptr;
    int fd;
    __uint32_t u32;
    __uint64_t u64;
} epoll_data_t;
```

**3. epoll_wait** 函数等待事件的就绪，成功时返回就绪的事件数目，调用失败时返回 -1，等待超时返回 0。

- `epfd` 是epoll句柄
- `events` 表示从内核得到的就绪事件集合
- `maxevents` 告诉内核events的大小
- `timeout` 表示等待的超时事件

epoll是Linux内核为处理大批量文件描述符而作了改进的poll，是Linux下多路复用IO接口select/poll的增强版本，它能显著提高程序在大量并发连接中只有少量活跃的情况下的系统CPU利用率。原因就是获取事件的时候，它无须遍历整个被侦听的描述符集，只要遍历那些被内核IO事件异步唤醒而加入Ready队列的描述符集合就行了。

epoll除了提供select/poll那种IO事件的水平触发（Level Triggered）外，还提供了边缘触发（Edge Triggered），这就使得用户空间程序有可能缓存IO状态，减少epoll_wait/epoll_pwait的调用，提高应用程序效率。

- **水平触发（LT）：**默认工作模式，即当epoll_wait检测到某描述符事件就绪并通知应用程序时，应用程序可以不立即处理该事件；下次调用epoll_wait时，会再次通知此事件
- **边缘触发（ET）：** 当epoll_wait检测到某描述符事件就绪并通知应用程序时，应用程序必须立即处理该事件。如果不处理，下次调用epoll_wait时，不会再次通知此事件。（直到你做了某些操作导致该描述符变成未就绪状态了，也就是说边缘触发只在状态由未就绪变为就绪时只通知一次）。

LT和ET原本应该是用于脉冲信号的，可能用它来解释更加形象。Level和Edge指的就是触发点，Level为只要处于水平，那么就一直触发，而Edge则为上升沿和下降沿的时候触发。比如：0->1 就是Edge，1->1 就是Level。

ET模式很大程度上减少了epoll事件的触发次数，因此效率比LT模式下高。



### 4 Linux内核中的select\poll\epoll的区别？

![image-20210817171343722](D:\1书本笔记\java实战项目\image-20210817171343722.png)

![image-20210817171526038](D:\1书本笔记\java实战项目\image-20210817171526038.png)

### 5 PIO与DMA的认识?

```
   PIO的英文拼写是“Programming Input/Output Model”，PIO模式是一种通过CPU执行I/O端口指令来进行
数据的读写的数据交换模式。是最早先的硬盘数据传输模式，数据传输速率低下，CPU占有率也很高，大量
传输数据时会因为占用过多的CPU资源而导致系统停顿，无法进行其它的操作。PIO数据传输模式又分为PIO
 mode 0、PIO mode 1、PIO mode 2、PIO mode 3、PIO mode 4几种模式，数据传输速率从3.3MB/s到
16.6MB/s不等。受限于传输速率低下和极高的CPU占有率，这种数据传输模式很快就被淘汰。
DMA模式
    DMA的英文拼写是“Direct Memory Access”，汉语的意思就是直接内存访问，是一种不经过CPU而直接
从内存了存取数据的数据交换模式。PIO模式下硬盘和内存之间的数据传输是由CPU来控制的；而在DMA模式
下，CPU只须向DMA控制器下达指令，让DMA控制器来处理数的传送，数据传送完毕再把信息反馈给CPU，这
样就很大程度上减轻了CPU资源占有率。DMA模式与PIO模式的区别就在于，DMA模式不过分依赖CPU，可以
大大节省系统资源，二者在传输速度上的差异并不十分明显。DMA模式又可以分为Single-Word DMA（单字
节DMA）和Multi-Word DMA（多字节DMA）两种，其中所能达到的最大传输速率也只有16.6MB/s。
DMA模式有着更快的速度和更低的CPU占用率.
```

### 6 简述I/O的同步和非同步的概念？

- **同步与异步**

  - **同步：** 同步就是发起一个调用后，被调用者未处理完请求之前，调用不返回。
  - **异步：** 异步就是发起一个调用后，立刻得到被调用者的回应表示已接收到请求，但是被调用者并没有返回结果，此时我们可以处理其他的请求，被调用者通常依靠事件，回调等机制来通知调用者其返回结果。

  同步和异步的区别最大在于异步的话调用者不需要等待处理结果，被调用者会通过回调等机制来通知调用者其返回结果。



### 7 简述I/O阻塞和非阻塞的认识？

- **阻塞和非阻塞**
  - **阻塞：** 阻塞就是发起一个请求，调用者一直等待请求结果返回，也就是当前线程会被挂起，无法从事其他任务，只有当条件就绪才能继续。
  - **非阻塞：** 非阻塞就是发起一个请求，调用者不用一直等着结果返回，可以先去干其他事情。

# Java I/O

### 1 简述BIO/NIO/AIO的认识？

链接：https://blog.csdn.net/m0_38109046/article/details/89449305

**那么同步阻塞、同步非阻塞和异步非阻塞又代表什么意思呢？**

举个生活中简单的例子，你妈妈让你烧水，小时候你比较笨啊，在哪里傻等着水开（**同步阻塞**）。等你稍微再长大一点，你知道每次烧水的空隙可以去干点其他事，然后只需要时不时来看看水开了没有（**同步非阻塞**）。后来，你们家用上了水开了会发出声音的壶，这样你就只需要听到响声后就知道水开了，在这期间你可以随便干自己的事情，你需要去倒水了（**异步非阻塞**）。

#### 1 BIO

同步阻塞I/O模式，数据的读取写入必须阻塞在一个线程内等待其完成。

##### 1.1 传统 BIO

BIO通信（一请求一应答）模型图如下(图源网络，原出处不明)：

![img](https://img-blog.csdnimg.cn/2019042212100021.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L20wXzM4MTA5MDQ2,size_16,color_FFFFFF,t_70)

采用 **BIO 通信模型** 的服务端，通常由一个独立的 Acceptor 线程负责监听客户端的连接。我们一般通过在 `while(true)` 循环中服务端会调用 `accept()` 方法等待接收客户端的连接的方式监听请求，请求一旦接收到一个连接请求，就可以建立通信套接字在这个通信套接字上进行读写操作，此时不能再接收其他客户端连接请求，只能等待同当前连接的客户端的操作执行完成， 不过可以通过多线程来支持多个客户端的连接，如上图所示。

如果要让 **BIO 通信模型** 能够同时处理多个客户端请求，就必须使用多线程（主要原因是 `socket.accept()`、 `socket.read()`、 `socket.write()` 涉及的三个主要函数都是同步阻塞的），也就是说它在接收到客户端连接请求之后为每个客户端创建一个新的线程进行链路处理，处理完成之后，通过输出流返回应答给客户端，线程销毁。这就是典型的 **一请求一应答通信模型** 。我们可以设想一下如果这个连接不做任何事情的话就会造成不必要的线程开销，不过可以通过 **线程池机制** 改善，线程池还可以让线程的创建和回收成本相对较低。使用`FixedThreadPool` 可以有效的控制了线程的最大数量，保证了系统有限的资源的控制，实现了N(客户端请求数量):M(处理客户端请求的线程数量)的伪异步I/O模型（N 可以远远大于 M），下面一节"伪异步 BIO"中会详细介绍到。

**我们再设想一下当客户端并发访问量增加后这种模型会出现什么问题？**

在 Java 虚拟机中，线程是宝贵的资源，线程的创建和销毁成本很高，除此之外，线程的切换成本也是很高的。尤其在 Linux 这样的操作系统中，线程本质上就是一个进程，创建和销毁线程都是重量级的系统函数。如果并发访问量增加会导致线程数急剧膨胀可能会导致线程堆栈溢出、创建新线程失败等问题，最终导致进程宕机或者僵死，不能对外提供服务。

##### 1.2 伪异步 IO

为了解决同步阻塞I/O面临的一个链路需要一个线程处理的问题，后来有人对它的线程模型进行了优化一一一后端通过一个线程池来处理多个客户端的请求接入，形成客户端个数M：线程池最大线程数N的比例关系，其中M可以远远大于N.通过线程池可以灵活地调配线程资源，设置线程的最大值，防止由于海量并发接入导致线程耗尽。

伪异步IO模型图(图源网络，原出处不明)：

![img](https://img-blog.csdnimg.cn/20190422121019666.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L20wXzM4MTA5MDQ2,size_16,color_FFFFFF,t_70)

采用线程池和任务队列可以实现一种叫做伪异步的 I/O 通信框架，它的模型图如上图所示。当有新的客户端接入时，将客户端的 Socket 封装成一个Task（该任务实现java.lang.Runnable接口）投递到后端的线程池中进行处理，JDK 的线程池维护一个消息队列和 N 个活跃线程，对消息队列中的任务进行处理。由于线程池可以设置消息队列的大小和最大线程数，因此，它的资源占用是可控的，无论多少个客户端并发访问，都不会导致资源的耗尽和宕机。

**伪异步I/O通信框架采用了线程池实现，因此避免了为每个请求都创建一个独立线程造成的线程资源耗尽问题。不过因为它的底层任然是同步阻塞的BIO模型，因此无法从根本上解决问题。**



在活动连接数不是特别高（小于单机1000）的情况下，这种模型是比较不错的，可以让每一个连接专注于自己的 I/O 并且编程模型简单，也不用过多考虑系统的过载、限流等问题。线程池本身就是一个天然的漏斗，可以缓冲一些系统处理不了的连接或请求。但是，当面对十万甚至百万级连接的时候，传统的 BIO 模型是无能为力的。因此，我们需要一种更高效的 I/O 处理模型来应对更高的并发量。



#### 2 NIO

##### 2.1 NIO 简介

NIO是一种同步非阻塞的I/O模型，在Java 1.4 中引入了NIO框架，对应 java.nio 包，提供了 Channel , Selector，Buffer等抽象。

NIO中的N可以理解为Non-blocking，不单纯是New。它支持面向缓冲的，基于通道的I/O操作方法。 NIO提供了与传统BIO模型中的 `Socket` 和 `ServerSocket` 相对应的 `SocketChannel` 和 `ServerSocketChannel` 两种不同的套接字通道实现,两种通道都支持阻塞和非阻塞两种模式。阻塞模式使用就像传统中的支持一样，比较简单，但是性能和可靠性都不好；非阻塞模式正好与之相反。对于低负载、低并发的应用程序，可以使用同步阻塞I/O来提升开发速率和更好的维护性；对于高负载、高并发的（网络）应用，应使用 NIO 的非阻塞模式来开发。

##### 2.2 NIO的特性/NIO与IO区别

如果是在面试中回答这个问题，我觉得首先肯定要从 NIO 流是非阻塞 IO 而 IO 流是阻塞 IO 说起。然后，可以从 NIO 的3个核心组件/特性为 NIO 带来的一些改进来分析。如果，你把这些都回答上了我觉得你对于 NIO 就有了更为深入一点的认识，面试官问到你这个问题，你也能很轻松的回答上来了。

1)Non-blocking IO（非阻塞IO）

**IO流是阻塞的，NIO流是不阻塞的。**

Java NIO使我们可以进行非阻塞IO操作。比如说，单线程中从通道读取数据到buffer，同时可以继续做别的事情，当数据读取到buffer中后，线程再继续处理数据。写数据也是一样的。另外，非阻塞写也是如此。一个线程请求写入一些数据到某通道，但不需要等待它完全写入，这个线程同时可以去做别的事情。

Java IO的各种流是阻塞的。这意味着，当一个线程调用 `read()` 或 `write()` 时，该线程被阻塞，直到有一些数据被读取，或数据完全写入。该线程在此期间不能再干任何事情了

2)Buffer(缓冲区)

**IO 面向流(Stream oriented)，而 NIO 面向缓冲区(Buffer oriented)。**

Buffer是一个对象，它包含一些要写入或者要读出的数据。在NIO类库中加入Buffer对象，体现了新库与原I/O的一个重要区别。在面向流的I/O中·可以将数据直接写入或者将数据直接读到 Stream 对象中。虽然 Stream 中也有 Buffer 开头的扩展类，但只是流的包装类，还是从流读到缓冲区，而 NIO 却是直接读到 Buffer 中进行操作。

在NIO厍中，所有数据都是用缓冲区处理的。在读取数据时，它是直接读到缓冲区中的; 在写入数据时，写入到缓冲区中。任何时候访问NIO中的数据，都是通过缓冲区进行操作。

最常用的缓冲区是 ByteBuffer,一个 ByteBuffer 提供了一组功能用于操作 byte 数组。除了ByteBuffer,还有其他的一些缓冲区，事实上，每一种Java基本类型（除了Boolean类型）都对应有一种缓冲区。

3)Channel (通道)

NIO 通过Channel（通道） 进行读写。

通道是双向的，可读也可写，而流的读写是单向的。无论读写，通道只能和Buffer交互。因为 Buffer，通道可以异步地读写。

4)Selectors(选择器)

NIO有选择器，而IO没有。

选择器用于使用单个线程处理多个通道。因此，它需要较少的线程来处理这些通道。线程之间的切换对于操作系统来说是昂贵的。 因此，为了提高系统效率选择器是有用的。

![img](https://img-blog.csdnimg.cn/20190422121139668.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L20wXzM4MTA5MDQ2,size_16,color_FFFFFF,t_70)

##### 2.3 NIO 读数据和写数据方式

通常来说NIO中的所有IO都是从 Channel（通道） 开始的。

- 从通道进行数据读取 ：创建一个缓冲区，然后请求通道读取数据。
- 从通道进行数据写入 ：创建一个缓冲区，填充数据，并要求通道写入数据。

数据读取和写入操作图示：

![img](https://img-blog.csdnimg.cn/20190422121151244.png)

##### 2.4 NIO核心组件简单介绍

NIO 包含下面几个核心的组件：

- Channel(通道)
- Buffer(缓冲区)
- Selector(选择器)

整个NIO体系包含的类远远不止这三个，只能说这三个是NIO体系的“核心API”。我们上面已经对这三个概念进行了基本的阐述，这里就不多做解释了。

#### 3 AIO

AIO 也就是 NIO 2。在 Java 7 中引入了 NIO 的改进版 NIO 2,它是异步非阻塞的IO模型。异步 IO 是基于事件和回调机制实现的，也就是应用操作之后会直接返回，不会堵塞在那里，**当后台处理完成，操作系统会通知相应的线程进行后续的操作**。

AIO 是异步IO的缩写，虽然 NIO 在网络操作中，提供了非阻塞的方法，但是 NIO 的 IO 行为还是同步的。对于 NIO 来说，我们的业务线程是在 IO 操作准备好时，得到通知，接着就由这个线程自行进行 IO 操作，IO操作本身是同步的。**除了 AIO 其他的 IO 类型都是同步的**

### 2   简述BIO/NIO/AIO的区别？

* BIO：线程发起IO请求，不管内核是否准备好IO操作，从发起请求起，线程一直阻塞，直到操作完成。
* NIO：线程发起IO请求，立即返回；内核在做好IO操作的准备之后，通过调用注册的回调函数通知线程做IO操作，线程开始阻塞，直到操作完成。
* AIO：线程发起IO请求，立即返回；内存做好IO操作的准备之后，做IO操作，直到操作完成或者失败，通过调用注册的回调函数通知线程做IO操作完成或者失败。

**BIO是一个连接一个线程。**
**NIO是一个请求一个线程。**
**AIO是一个有效请求一个线程。**

* BIO：同步并阻塞，服务器实现模式为一个连接一个线程，即客户端有连接请求时服务器端就需要启动一个线程进行处理，如果这个连接不做任何事情会造成不必要的线程开销，当然可以通过线程池机制改善。
* NIO：同步非阻塞，服务器实现模式为一个请求一个线程，即客户端发送的连接请求都会注册到多路复用器上，多路复用器轮询到连接有I/O请求时才启动一个线程进行处理。
* AIO：异步非阻塞，服务器实现模式为一个有效请求一个线程，**客户端的I/O请求都是由OS先完成了再通知服务器应用去启动线程进行处理**。

### 3 BIO/NIO/AIO的适用场景分析

1. BIO方式适用于连接数目比较小且固定的架构，这种方式对服务器资源要求比较高，并发局限于应用中，JDK1.4以前的唯一选择，但程序直观简单易理解。
2. NIO方式适用于连接数目多且连接比较短（轻操作）的架构，比如聊天服务器，并发局限于应用中，编程比较复杂，JDK1.4开始支持。
3. AIO方式使用于连接数目多且连接比较长（重操作）的架构，比如相册服务器，充分调用OS参与并发操作，编程比较复杂，JDK7开始支持。

### 4 简述Java I/O中涉及到设计模式

 Java IO 流在整体设计上还涉及**装饰者（Decorator）和适配器（Adapter）两种设计模式。**

1. 对于 IO 流涉及的**装饰者设计模式**例子如下：



```cpp
//把InputStreamReader装饰成BufferedReader来成为具备缓冲能力的Reader。
BufferedReader bufferedReader = new BufferedReader(inputStreamReader);
```

1. 对于 IO 流涉及的**适配器设计模式**例子如下：



```cpp
//把FileInputStream文件字节流适配成InputStreamReader字符流来操作文件字符串。
FileInputStream fileInput = new FileInputStream(file); 
InputStreamReader inputStreamReader = new InputStreamReader(fileInput);
```

而对于上面涉及的两种设计模式通俗总结如下：

- **装饰者模式**就是给一个对象增加一些新的功能，而且是动态的，要求装饰对象和被装饰对象实现同一个接口，装饰对象持有被装饰对象的实例**（各种字符流间装饰，各种字节流间装饰）。**
- **适配器模式**就是将某个类的接口转换成我们期望的另一个接口表示，目的是消除由于接口不匹配所造成的类的兼容性问题**（字符流与字节流间互相适配）。**



# Reactor模式

链接：https://cloud.tencent.com/developer/article/1488120

### 1 简述对Reactor模式的认识？

Reactor模式 是一种「事件驱动」模式。Reactor模型基于事件驱动，特别适合处理海量的I/O事件。



### 2 Reactor模式的优缺点？

#### 1. **优点**

1）响应快，不必为单个同步时间所阻塞，虽然Reactor本身依然是同步的；

2）编程相对简单，可以最大程度的避免复杂的多线程及同步问题，并且避免了多线程/进程的切换开销；

3）可扩展性，可以方便的通过增加Reactor实例个数来充分利用CPU资源；

4）可复用性，reactor框架本身与具体事件处理逻辑无关，具有很高的复用性；

#### 2. **缺点**

1）相比传统的简单模型，Reactor增加了一定的复杂性，因而有一定的门槛，并且不易于调试。

2）Reactor模式需要底层的Synchronous Event Demultiplexer支持，比如Java中的Selector支持，操作系统的select系统调用支持，如果要自己实现Synchronous Event Demultiplexer可能不会有那么高效。

3） Reactor模式在IO读写数据时还是在同一个线程中实现的，即使使用多个Reactor机制的情况下，那些共享一个Reactor的Channel如果出现一个长时间的数据读写，会影响这个Reactor中其他Channel的相应时间，比如在大文件传输时，IO操作就会影响其他Client的相应时间，因而对这种操作，使用传统的Thread-Per-Connection或许是一个更好的选择，或则此时使用改进版的Reactor模式如Proactor模式。



### 3 Reactor中的几类角色?

Reactor模型中定义的三种角色：

- Reactor：负责监听和分配事件，将I/O事件分派给对应的Handler。新的事件包含连接建立就绪、读就绪、写就绪等。
- Acceptor：处理客户端新连接，并分派请求到处理器链中。
- Handler：将自身与事件绑定，执行非阻塞读/写任务，完成channel的读入，完成处理业务逻辑后，负责将结果写出channel。可用资源池来管理。



### 4 Reactor中单线程及其执行流程？

Reactor线程负责多路分离套接字，accept新连接，并分派请求到handler。[Redis](https://cloud.tencent.com/product/crs?from=10680)使用单Reactor单进程的模型。

![image-20210817221545530](D:\1书本笔记\java实战项目\image-20210817221545530.png)

消息处理流程：

1. Reactor对象通过select监控连接事件，收到事件后通过dispatch进行转发。
2. 如果是连接建立的事件，则由acceptor接受连接，并创建handler处理后续事件。
3. 如果不是建立连接事件，则Reactor会分发调用Handler来响应。
4. handler会完成read->业务处理->send的完整业务流程。

单Reactor单线程模型只是在代码上进行了组件的区分，但是整体操作还是单线程，不能充分利用硬件资源。handler业务处理部分没有异步。

对于一些小容量应用场景，可以使用单Reactor单线程模型。但是对于高负载、大并发的应用场景却不合适，主要原因如下：

1. 即便Reactor线程的CPU负荷达到100%，也无法满足海量消息的编码、解码、读取和发送。
2. 当Reactor线程负载过重之后，处理速度将变慢，这会导致大量客户端连接超时，超时之后往往会进行重发，这更加重Reactor线程的负载，最终会导致大量消息积压和处理超时，成为系统的性能瓶颈。
3. 一旦Reactor线程意外中断或者进入死循环，会导致整个系统通信模块不可用，不能接收和处理外部消息，造成节点故障。



### 5 Reactor中多线程及其执行流程？

该模型在事件处理器（Handler）部分采用了多线程（线程池）。

![image-20210817222117687](D:\1书本笔记\java实战项目\image-20210817222117687.png)

消息处理流程：

1. Reactor对象通过Select监控客户端请求事件，收到事件后通过dispatch进行分发。
2. 如果是建立连接请求事件，则由acceptor通过accept处理连接请求，然后创建一个Handler对象处理连接完成后续的各种事件。
3. 如果不是建立连接事件，则Reactor会分发调用连接对应的Handler来响应。
4. Handler只负责响应事件，不做具体业务处理，通过Read读取数据后，会分发给后面的Worker线程池进行业务处理。
5. Worker线程池会分配独立的线程完成真正的业务处理，如何将响应结果发给Handler进行处理。
6. Handler收到响应结果后通过send将响应结果返回给Client。

相对于第一种模型来说，在处理业务逻辑，也就是获取到IO的读写事件之后，交由线程池来处理，handler收到响应后通过send将响应结果返回给客户端。这样可以降低Reactor的性能开销，从而更专注的做事件分发工作了，提升整个应用的吞吐。

但是这个模型存在的问题：

1. 多线程数据共享和访问比较复杂。如果子线程完成业务处理后，把结果传递给主线程Reactor进行发送，就会涉及共享数据的互斥和保护机制。
2. Reactor承担所有事件的监听和响应，只在主线程中运行，可能会存在性能问题。例如并发百万客户端连接，或者服务端需要对客户端握手进行安全认证，但是认证本身非常损耗性能。

为了解决性能问题，产生了第三种主从Reactor多线程模型。

### 6 主从Reactor中多线程及其执行流程？

比起第二种模型，它是将Reactor分成两部分：

1. mainReactor负责监听server socket，用来处理网络IO连接建立操作，将建立的socketChannel指定注册给subReactor。
2. subReactor主要做和建立起来的socket做数据交互和事件业务处理操作。通常，subReactor个数上可与CPU个数等同。

**Nginx、Swoole、Memcached和Netty都是采用这种实现**。

![image-20210817222246667](D:\1书本笔记\java实战项目\image-20210817222246667.png)

消息处理流程：

1. 从主线程池中随机选择一个Reactor线程作为acceptor线程，用于绑定监听端口，接收客户端连接
2. acceptor线程接收客户端连接请求之后创建新的SocketChannel，将其注册到主线程池的其它Reactor线程上，由其负责接入认证、IP黑白名单过滤、握手等操作
3. 步骤2完成之后，业务层的链路正式建立，将SocketChannel从主线程池的Reactor线程的多路复用器上摘除，重新注册到Sub线程池的线程上，并创建一个Handler用于处理各种连接事件
4. 当有新的事件发生时，SubReactor会调用连接对应的Handler进行响应
5. Handler通过Read读取数据后，会分发给后面的Worker线程池进行业务处理
6. Worker线程池会分配独立的线程完成真正的业务处理，如何将响应结果发给Handler进行处理
7. Handler收到响应结果后通过Send将响应结果返回给Client



# 其它

## 1 Lambda表达式

一、Lambda表达式的由来

链接：https://www.zhihu.com/question/20125256

### 1 **什么是Lambda?**

我们知道，对于一个Java变量，我们可以赋给其一个**“值”**。

![img](https://pic3.zhimg.com/50/v2-ab6545c49383236a4af3f28a47886090_720w.jpg?source=1940ef5c)![img](https://pic3.zhimg.com/80/v2-ab6545c49383236a4af3f28a47886090_720w.jpg?source=1940ef5c)

如果你想把**“一块代码”**赋给一个Java变量，应该怎么做呢？

比如，我想把右边那块代码，赋给一个叫做aBlockOfCode的Java变量：

![img](https://pic1.zhimg.com/50/v2-1cc87e82fba0872c2cae3fee08e8fe41_720w.jpg?source=1940ef5c)![img](https://pic1.zhimg.com/80/v2-1cc87e82fba0872c2cae3fee08e8fe41_720w.jpg?source=1940ef5c)

在Java 8之前，这个是做不到的。但是Java 8问世之后，利用Lambda特性，就可以做到了。

![img](https://pica.zhimg.com/50/v2-145a556d86806c3163391a13428e3f03_720w.jpg?source=1940ef5c)![img](https://pica.zhimg.com/80/v2-145a556d86806c3163391a13428e3f03_720w.jpg?source=1940ef5c)

当然，这个并不是一个很简洁的写法。所以，为了使这个赋值操作更加elegant, 我们可以移除一些没用的声明。

![img](https://pic2.zhimg.com/50/v2-a712753b42972e094a548ae02fa82987_720w.jpg?source=1940ef5c)![img](https://pic2.zhimg.com/80/v2-a712753b42972e094a548ae02fa82987_720w.jpg?source=1940ef5c)

这样，我们就成功的非常优雅的把“一块代码”赋给了一个变量。**而“这块代码”，或者说“这个被赋给一个变量的函数”，就是一个Lambda表达式**。

但是这里仍然有一个问题，就是变量aBlockOfCode的类型应该是什么？

在Java 8里面，**所有的Lambda的类型都是一个接口，而Lambda表达式本身，也就是”那段代码“，需要是这个接口的实现。**这是我认为理解Lambda的一个关键所在，简而言之就是，**Lambda表达式本身就是一个接口的实现**。直接这样说可能还是有点让人困扰，我们继续看看例子。我们给上面的aBlockOfCode加上一个类型：

![img](https://pic3.zhimg.com/50/v2-55de66060b4cb70193ddc7fea201b257_720w.jpg?source=1940ef5c)![img](https://pic3.zhimg.com/80/v2-55de66060b4cb70193ddc7fea201b257_720w.jpg?source=1940ef5c)

这种只有**一个接口函数需要被实现的接口类型，我们叫它”函数式接口“。**为了避免后来的人在这个接口中增加接口函数导致其有多个接口函数需要被实现，变成"非函数接口”，我们可以在这个上面加上一个声明**@FunctionalInterface**, 这样别人就无法在里面添加新的接口函数了：

![img](https://pic2.zhimg.com/50/v2-2c57e7411de227d1eb09c327d01fb766_720w.jpg?source=1940ef5c)![img](https://pic2.zhimg.com/80/v2-2c57e7411de227d1eb09c327d01fb766_720w.jpg?source=1940ef5c)

这样，我们就得到了一个完整的Lambda表达式声明：

![img](https://pic3.zhimg.com/50/v2-02eedc528fcee115f5ed0b7b045846d7_720w.jpg?source=1940ef5c)![img](https://pic3.zhimg.com/80/v2-02eedc528fcee115f5ed0b7b045846d7_720w.jpg?source=1940ef5c)

### 2 Lambda结合FunctionalInterface Lib, forEach, stream()，method reference等新特性可以使代码变的更加简洁！



直接上例子。

假设Person的定义和List<Person>的值都给定。

![image-20210715195841452](D:\1书本笔记\java实战项目\image-20210715195841452.png)

现在需要你打印出guiltyPersons List里面所有LastName以"Z"开头的人的FirstName。

**原生态Lambda写法**：定义两个函数式接口，定义一个静态函数，调用静态函数并给参数赋值Lambda表达式。

![image-20210715195904029](D:\1书本笔记\java实战项目\image-20210715195904029.png)

这个代码实际上已经比较简洁了，但是我们还可以更简洁么？

当然可以。在Java 8中有一个**函数式接口的包**，里面定义了大量可能用到的函数式接口（[java.util.function (Java Platform SE 8 )](https://link.zhihu.com/?target=https%3A//docs.oracle.com/javase/8/docs/api/java/util/function/package-summary.html)）。所以，我们在这里压根都不需要定义NameChecker和Executor这两个函数式接口，直接用Java 8函数式接口包里的Predicate<T>和Consumer<T>就可以了——因为他们这一对的接口定义和NameChecker/Executor其实是一样的。

![image-20210715200017550](D:\1书本笔记\java实战项目\image-20210715200017550.png)

**第一步简化 - 利用函数式接口包：**

![image-20210715200034820](D:\1书本笔记\java实战项目\image-20210715200034820.png)

静态函数里面的for each循环其实是非常碍眼的。这里可以利用Iterable自带的forEach()来替代。forEach()本身可以接受一个Consumer<T> 参数。

**第二步简化 - 用Iterable.forEach()取代foreach loop：**

![image-20210715200132350](D:\1书本笔记\java实战项目\image-20210715200132350.png)

由于静态函数其实只是对List进行了一通操作，这里我们可以甩掉静态函数，直接使用stream()特性来完成。stream()的几个方法都是接受Predicate<T>，Consumer<T>等参数的（[java.util.stream (Java Platform SE 8 )](https://link.zhihu.com/?target=https%3A//docs.oracle.com/javase/8/docs/api/java/util/stream/package-summary.html)）。你理解了上面的内容，stream()这里就非常好理解了，并不需要多做解释。

**第三步简化 - 利用stream()替代静态函数：**

![img](https://pic1.zhimg.com/50/v2-e196d987f852b9b8e26a6a9dac648a06_720w.jpg?source=1940ef5c)![img](https://pic1.zhimg.com/80/v2-e196d987f852b9b8e26a6a9dac648a06_720w.jpg?source=1940ef5c)

对比最开始的Lambda写法，这里已经非常非常简洁了。但是如果，我们的要求变一下，变成print这个人的全部信息，及p -> System.out.println(p); 那么还可以利用Method reference来继续简化。所谓Method reference, 就是用已经写好的别的Object/Class的method来代替Lambda expression。格式如下：

![img](https://pic3.zhimg.com/50/v2-12622326a5682285ce235d96291f3bb8_720w.jpg?source=1940ef5c)![img](https://pic3.zhimg.com/80/v2-12622326a5682285ce235d96291f3bb8_720w.jpg?source=1940ef5c)

**第四步简化 - 如果是println(p)，则可以利用Method reference代替forEach中的Lambda表达式：**

![img](https://pic2.zhimg.com/50/v2-f29e6569d0265b91794565ae81d54265_720w.jpg?source=1940ef5c)![img](https://pic2.zhimg.com/80/v2-f29e6569d0265b91794565ae81d54265_720w.jpg?source=1940ef5c)

这基本上就是能写的最简洁的版本了。

### 3 Lambda表达式的使用

Lambda表达式的用法

参考链接：https://www.cnblogs.com/franson-2016/p/5593080.html

#### **简介**

(**译者注**:虽然看着很先进，其实Lambda表达式的本质只是一个"[**语法糖**](http://zh.wikipedia.org/wiki/语法糖)",由编译器推断并帮你转换包装为常规的代码,因此你可以使用更少的代码来实现同样的功能。本人建议不要乱用,因为这就和某些很高级的黑客写的代码一样,简洁,难懂,难以调试,维护人员想骂娘.)
Lambda表达式是Java SE 8中一个重要的新特性。lambda表达式允许你通过表达式来代替功能接口。 lambda表达式就和方法一样,它提供了一个正常的参数列表和一个使用这些参数的主体(body,可以是一个表达式或一个代码块)。
Lambda表达式还增强了集合库。 Java SE 8添加了2个对集合数据进行批量操作的包: java.util.function 包以及java.util.stream 包。 流(stream)就如同迭代器(iterator),但附加了许多额外的功能。 总的来说,lambda表达式和 stream 是自Java语言添加泛型(Generics)和注解(annotation)以来最大的变化。 在本文中,我们将从简单到复杂的示例中见认识lambda表达式和stream的强悍。

#### **环境准备**

如果还没有安装Java 8,那么你应该先安装才能使用lambda和stream(译者建议在**虚拟机**中安装,测试使用)。 像NetBeans 和IntelliJ IDEA 一类的工具和IDE就支持Java 8特性,包括lambda表达式,可重复的注解,紧凑的概要文件和其他特性。
下面是Java SE 8和NetBeans IDE 8的下载链接:
**[Java Platform (JDK 8)](http://www.oracle.com/technetwork/java/javase/downloads/index.html)**: 从Oracle下载Java 8,也可以和NetBeans IDE一起下载
[**NetBeans IDE 8**](https://netbeans.org/downloads/index.html): 从NetBeans官网下载NetBeans IDE

#### **Lambda表达式的语法**

基本语法:
**(parameters) -> expression**
或
**(parameters) ->{ statements; }**

下面是Java lambda表达式的简单例子:

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
// 1. 不需要参数,返回值为 5  
() -> 5  
  
// 2. 接收一个参数(数字类型),返回其2倍的值  
x -> 2 * x  
  
// 3. 接受2个参数(数字),并返回他们的差值  
(x, y) -> x – y  
  
// 4. 接收2个int型整数,返回他们的和  
(int x, int y) -> x + y  
  
// 5. 接受一个 string 对象,并在控制台打印,不返回任何值(看起来像是返回void)  
(String s) -> System.out.print(s)  
```

#### **基本的Lambda例子**

现在,我们已经知道什么是lambda表达式,让我们先从一些基本的例子开始。 在本节中,我们将看到lambda表达式如何影响我们编码的方式。 假设有一个玩家List ,程序员可以使用 for 语句 ("for 循环")来遍历,在Java SE 8中可以转换为另一种形式:

```
String[] atp = {"Rafael Nadal", "Novak Djokovic",  
       "Stanislas Wawrinka",  
       "David Ferrer","Roger Federer",  
       "Andy Murray","Tomas Berdych",  
       "Juan Martin Del Potro"};  
List<String> players =  Arrays.asList(atp);  
  
// 以前的循环方式  
for (String player : players) {  
     System.out.print(player + "; ");  
}  
  
// 使用 lambda 表达式以及函数操作(functional operation)  
players.forEach((player) -> System.out.print(player + "; "));  
   
// 在 Java 8 中使用双冒号操作符(double colon operator)  
players.forEach(System.out::println);  
```

正如您看到的,lambda表达式可以将我们的代码缩减到一行。 另一个例子是在图形用户界面程序中,匿名类可以使用lambda表达式来代替。 同样,在实现Runnable接口时也可以这样使用:

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
// 使用匿名内部类  
btn.setOnAction(new EventHandler<ActionEvent>() {  
          @Override  
          public void handle(ActionEvent event) {  
              System.out.println("Hello World!");   
          }  
    });  
   
// 或者使用 lambda expression  
btn.setOnAction(event -> System.out.println("Hello World!"));  
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

下面是使用lambdas 来实现 Runnable接口 的示例:

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
// 1.1使用匿名内部类  
new Thread(new Runnable() {  
    @Override  
    public void run() {  
        System.out.println("Hello world !");  
    }  
}).start();  
  
// 1.2使用 lambda expression  
new Thread(() -> System.out.println("Hello world !")).start();  
  
// 2.1使用匿名内部类  
Runnable race1 = new Runnable() {  
    @Override  
    public void run() {  
        System.out.println("Hello world !");  
    }  
};  
  
// 2.2使用 lambda expression  
Runnable race2 = () -> System.out.println("Hello world !");  
   
// 直接调用 run 方法(没开新线程哦!)  
race1.run();  
race2.run();  
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 


Runnable 的 lambda表达式,使用块格式,将五行代码转换成单行语句。 接下来,在下一节中我们将使用lambdas对集合进行排序。

#### **使用Lambdas排序集合**

在Java中,Comparator 类被用来排序集合。 在下面的例子中,我们将根据球员的 name, surname, name 长度 以及最后一个字母。 和前面的示例一样,先使用匿名内部类来排序,然后再使用lambda表达式精简我们的代码。
在第一个例子中,我们将根据name来排序list。 使用旧的方式,代码如下所示:

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
String[] players = {"Rafael Nadal", "Novak Djokovic",   
    "Stanislas Wawrinka", "David Ferrer",  
    "Roger Federer", "Andy Murray",  
    "Tomas Berdych", "Juan Martin Del Potro",  
    "Richard Gasquet", "John Isner"};  
   
// 1.1 使用匿名内部类根据 name 排序 players  
Arrays.sort(players, new Comparator<String>() {  
    @Override  
    public int compare(String s1, String s2) {  
        return (s1.compareTo(s2));  
    }  
});  
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

使用lambdas,可以通过下面的代码实现同样的功能:

```
// 1.2 使用 lambda expression 排序 players  
Comparator<String> sortByName = (String s1, String s2) -> (s1.compareTo(s2));  
Arrays.sort(players, sortByName);  
  
// 1.3 也可以采用如下形式:  
Arrays.sort(players, (String s1, String s2) -> (s1.compareTo(s2)));  
```

 

其他的排序如下所示。 和上面的示例一样,代码分别通过匿名内部类和一些lambda表达式来实现Comparator :

 

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
// 1.1 使用匿名内部类根据 surname 排序 players  
Arrays.sort(players, new Comparator<String>() {  
    @Override  
    public int compare(String s1, String s2) {  
        return (s1.substring(s1.indexOf(" ")).compareTo(s2.substring(s2.indexOf(" "))));  
    }  
});  
  
// 1.2 使用 lambda expression 排序,根据 surname  
Comparator<String> sortBySurname = (String s1, String s2) ->   
    ( s1.substring(s1.indexOf(" ")).compareTo( s2.substring(s2.indexOf(" ")) ) );  
Arrays.sort(players, sortBySurname);  
  
// 1.3 或者这样,怀疑原作者是不是想错了,括号好多...  
Arrays.sort(players, (String s1, String s2) ->   
      ( s1.substring(s1.indexOf(" ")).compareTo( s2.substring(s2.indexOf(" ")) ) )   
    );  
  
// 2.1 使用匿名内部类根据 name lenght 排序 players  
Arrays.sort(players, new Comparator<String>() {  
    @Override  
    public int compare(String s1, String s2) {  
        return (s1.length() - s2.length());  
    }  
});  
  
// 2.2 使用 lambda expression 排序,根据 name lenght  
Comparator<String> sortByNameLenght = (String s1, String s2) -> (s1.length() - s2.length());  
Arrays.sort(players, sortByNameLenght);  
  
// 2.3 or this  
Arrays.sort(players, (String s1, String s2) -> (s1.length() - s2.length()));  
  
// 3.1 使用匿名内部类排序 players, 根据最后一个字母  
Arrays.sort(players, new Comparator<String>() {  
    @Override  
    public int compare(String s1, String s2) {  
        return (s1.charAt(s1.length() - 1) - s2.charAt(s2.length() - 1));  
    }  
});  
  
// 3.2 使用 lambda expression 排序,根据最后一个字母  
Comparator<String> sortByLastLetter =   
    (String s1, String s2) ->   
        (s1.charAt(s1.length() - 1) - s2.charAt(s2.length() - 1));  
Arrays.sort(players, sortByLastLetter);  
  
// 3.3 or this  
Arrays.sort(players, (String s1, String s2) -> (s1.charAt(s1.length() - 1) - s2.charAt(s2.length() - 1)));  
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

就是这样,简洁又直观。 在下一节中我们将探索更多lambdas的能力,并将其与 stream 结合起来使用。

#### 重点：**使用Lambdas和Streams**

Stream是对集合的包装,通常和lambda一起使用。 使用lambdas可以支持许多操作,如 map, filter, limit, sorted, count, min, max, sum, collect 等等。 同样,Stream使用**懒运算**,他们并不会真正地读取所有数据,遇到像getFirst() 这样的方法就会结束链式语法。 在接下来的例子中,我们将探索lambdas和streams 能做什么。 我们创建了一个Person类并使用这个类来添加一些数据到list中,将用于进一步流操作。 Person 只是一个简单的POJO类:

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class Person {  
  
private String firstName, lastName, job, gender;  
private int salary, age;  
  
public Person(String firstName, String lastName, String job,  
                String gender, int age, int salary)       {  
          this.firstName = firstName;  
          this.lastName = lastName;  
          this.gender = gender;  
          this.age = age;  
          this.job = job;  
          this.salary = salary;  
}  
// Getter and Setter   
// . . . . .  
}  
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

接下来,我们将创建两个list,都用来存放Person对象:

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
List<Person> javaProgrammers = new ArrayList<Person>() {  
  {  
    add(new Person("Elsdon", "Jaycob", "Java programmer", "male", 43, 2000));  
    add(new Person("Tamsen", "Brittany", "Java programmer", "female", 23, 1500));  
    add(new Person("Floyd", "Donny", "Java programmer", "male", 33, 1800));  
    add(new Person("Sindy", "Jonie", "Java programmer", "female", 32, 1600));  
    add(new Person("Vere", "Hervey", "Java programmer", "male", 22, 1200));  
    add(new Person("Maude", "Jaimie", "Java programmer", "female", 27, 1900));  
    add(new Person("Shawn", "Randall", "Java programmer", "male", 30, 2300));  
    add(new Person("Jayden", "Corrina", "Java programmer", "female", 35, 1700));  
    add(new Person("Palmer", "Dene", "Java programmer", "male", 33, 2000));  
    add(new Person("Addison", "Pam", "Java programmer", "female", 34, 1300));  
  }  
};  
  
List<Person> phpProgrammers = new ArrayList<Person>() {  
  {  
    add(new Person("Jarrod", "Pace", "PHP programmer", "male", 34, 1550));  
    add(new Person("Clarette", "Cicely", "PHP programmer", "female", 23, 1200));  
    add(new Person("Victor", "Channing", "PHP programmer", "male", 32, 1600));  
    add(new Person("Tori", "Sheryl", "PHP programmer", "female", 21, 1000));  
    add(new Person("Osborne", "Shad", "PHP programmer", "male", 32, 1100));  
    add(new Person("Rosalind", "Layla", "PHP programmer", "female", 25, 1300));  
    add(new Person("Fraser", "Hewie", "PHP programmer", "male", 36, 1100));  
    add(new Person("Quinn", "Tamara", "PHP programmer", "female", 21, 1000));  
    add(new Person("Alvin", "Lance", "PHP programmer", "male", 38, 1600));  
    add(new Person("Evonne", "Shari", "PHP programmer", "female", 40, 1800));  
  }  
};  
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

现在我们使用forEach方法来迭代输出上述列表:

```
System.out.println("所有程序员的姓名:");  
javaProgrammers.forEach((p) -> System.out.printf("%s %s; ", p.getFirstName(), p.getLastName()));  
phpProgrammers.forEach((p) -> System.out.printf("%s %s; ", p.getFirstName(), p.getLastName()));  
```

 

我们同样使用forEach方法,增加程序员的工资5%:

 

```
System.out.println("给程序员加薪 5% :");  
Consumer<Person> giveRaise = e -> e.setSalary(e.getSalary() / 100 * 5 + e.getSalary());  
  
javaProgrammers.forEach(giveRaise);  
phpProgrammers.forEach(giveRaise);  
```

 

另一个有用的方法是过滤器filter() ,让我们显示月薪超过1400美元的PHP程序员:

```
System.out.println("下面是月薪超过 $1,400 的PHP程序员:")  
phpProgrammers.stream()  
          .filter((p) -> (p.getSalary() > 1400))  
          .forEach((p) -> System.out.printf("%s %s; ", p.getFirstName(), p.getLastName()));  
```

 

我们也可以定义过滤器,然后重用它们来执行其他操作:

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
// 定义 filters  
Predicate<Person> ageFilter = (p) -> (p.getAge() > 25);  
Predicate<Person> salaryFilter = (p) -> (p.getSalary() > 1400);  
Predicate<Person> genderFilter = (p) -> ("female".equals(p.getGender()));  
  
System.out.println("下面是年龄大于 24岁且月薪在$1,400以上的女PHP程序员:");  
phpProgrammers.stream()  
          .filter(ageFilter)  
          .filter(salaryFilter)  
          .filter(genderFilter)  
          .forEach((p) -> System.out.printf("%s %s; ", p.getFirstName(), p.getLastName()));  
  
// 重用filters  
System.out.println("年龄大于 24岁的女性 Java programmers:");  
javaProgrammers.stream()  
          .filter(ageFilter)  
          .filter(genderFilter)  
          .forEach((p) -> System.out.printf("%s %s; ", p.getFirstName(), p.getLastName()));  
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

使用limit方法,可以限制结果集的个数:

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
System.out.println("最前面的3个 Java programmers:");  
javaProgrammers.stream()  
          .limit(3)  
          .forEach((p) -> System.out.printf("%s %s; ", p.getFirstName(), p.getLastName()));  
  
  
System.out.println("最前面的3个女性 Java programmers:");  
javaProgrammers.stream()  
          .filter(genderFilter)  
          .limit(3)  
          .forEach((p) -> System.out.printf("%s %s; ", p.getFirstName(), p.getLastName())); 
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

排序呢? 我们在stream中能处理吗? 答案是肯定的。 在下面的例子中,我们将根据名字和薪水排序Java程序员,放到一个list中,然后显示列表:

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
System.out.println("根据 name 排序,并显示前5个 Java programmers:");  
List<Person> sortedJavaProgrammers = javaProgrammers  
          .stream()  
          .sorted((p, p2) -> (p.getFirstName().compareTo(p2.getFirstName())))  
          .limit(5)  
          .collect(toList());  
  
sortedJavaProgrammers.forEach((p) -> System.out.printf("%s %s; %n", p.getFirstName(), p.getLastName()));  
   
System.out.println("根据 salary 排序 Java programmers:");  
sortedJavaProgrammers = javaProgrammers  
          .stream()  
          .sorted( (p, p2) -> (p.getSalary() - p2.getSalary()) )  
          .collect( toList() );  
  
sortedJavaProgrammers.forEach((p) -> System.out.printf("%s %s; %n", p.getFirstName(), p.getLastName()));  
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

如果我们只对最低和最高的薪水感兴趣,比排序后选择第一个/最后一个 更快的是min和max方法:

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
System.out.println("工资最低的 Java programmer:");  
Person pers = javaProgrammers  
          .stream()  
          .min((p1, p2) -> (p1.getSalary() - p2.getSalary()))  
          .get()  
  
System.out.printf("Name: %s %s; Salary: $%,d.", pers.getFirstName(), pers.getLastName(), pers.getSalary())  
  
System.out.println("工资最高的 Java programmer:");  
Person person = javaProgrammers  
          .stream()  
          .max((p, p2) -> (p.getSalary() - p2.getSalary()))  
          .get()  
  
System.out.printf("Name: %s %s; Salary: $%,d.", person.getFirstName(), person.getLastName(), person.getSalary())  
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

[

](http://blog.csdn.net/renfufei/article/details/24600507)

上面的例子中我们已经看到 collect 方法是如何工作的。 结合 map 方法,我们可以使用 collect 方法来将我们的结果集放到一个字符串,一个 Set 或一个TreeSet中:

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
System.out.println("将 PHP programmers 的 first name 拼接成字符串:");  
String phpDevelopers = phpProgrammers  
          .stream()  
          .map(Person::getFirstName)  
          .collect(joining(" ; ")); // 在进一步的操作中可以作为标记(token)     
  
System.out.println("将 Java programmers 的 first name 存放到 Set:");  
Set<String> javaDevFirstName = javaProgrammers  
          .stream()  
          .map(Person::getFirstName)  
          .collect(toSet());  
  
System.out.println("将 Java programmers 的 first name 存放到 TreeSet:");  
TreeSet<String> javaDevLastName = javaProgrammers  
          .stream()  
          .map(Person::getLastName)  
          .collect(toCollection(TreeSet::new));  
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

Streams 还可以是并行的(parallel)。 示例如下:

```
System.out.println("计算付给 Java programmers 的所有money:");  
int totalSalary = javaProgrammers  
          .parallelStream()  
          .mapToInt(p -> p.getSalary())  
          .sum();  
```

 

 

我们可以使用summaryStatistics方法获得stream 中元素的各种汇总数据。 接下来,我们可以访问这些方法,比如getMax, getMin, getSum或getAverage:

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
//计算 count, min, max, sum, and average for numbers  
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);  
IntSummaryStatistics stats = numbers  
          .stream()  
          .mapToInt((x) -> x)  
          .summaryStatistics();  
  
System.out.println("List中最大的数字 : " + stats.getMax());  
System.out.println("List中最小的数字 : " + stats.getMin());  
System.out.println("所有数字的总和   : " + stats.getSum());  
System.out.println("所有数字的平均值 : " + stats.getAverage()); 
```



## 2 compareTo()和compare的用法

总结：

* 如果你想升序，那么o1比o2小就是我想要的；所以返回-1，类比成false；**表示我不想调整顺序**

* 如果你想降序，那么o1比o2小不是我想要的；所以返回1，类比成true；**表示我想调整顺序**

特别的对于compare（o1,o2）方法（默认升序）

有这种用法

```java
          //  第一个参数在前面，返回-1， 表示o1比o2小就是我想要的，我不想调整顺序 ，所以是升序
			@Override
            public int compare(Integer o1, Integer o2) {
                return o1-o2;
            }
		  //  第一个参数在后面，返回1， 表示o1比o2小不是我想要的，我想调整顺序 ，所以是降序
            @Override
            public int compare(Integer o1, Integer o2) {
                return o2-o1;
            }
```

# 反射

### 1 简述对反射的理解

- 根据类名创建实例（类名可以从配置文件读取，不用new，达到解耦）
- 用Method.invoke执行方法

主要内容：

- JVM是如何构建一个实例的
- .class文件
- 类加载器
- Class类
- 反射API

#### 1**JVM是如何构建一个实例的**？

通过new创建实例和反射创建实例，都绕不开Class对象。

首先一个类创建的过程有两种方式：1 直接new 出来   2  通过反射调用方法区的class对象出来。

 

![image-20210722094927241](D:\1书本笔记\java实战项目\image-20210722094927241.png)

**. class文件**

​		经过类加载器的loadClass（）方法把  .java文件编译成对应的  .class文件加载到JVM内存的方法区的过程。（JVM可以根据字节数组创造Class对象）

![image-20210722095701114](D:\1书本笔记\java实战项目\image-20210722095701114.png)

#### 2**类加载器**

略：(具体的内容请看JVM)

#### 3**class类**

.class文件被类加载器加载到内存中，并且JVM根据其字节数组创建了对应的Class对象。所以，我们来研究一下Class对象

Class对象类至少包括以下信息（按顺序）：

- 权限修饰符
- 类名
- 参数化类型（泛型信息）
- 接口
- 注解
- **字段（重点）** Field
- **构造器（重点）** Constructor
- **方法（重点）** Method

源码：

![image-20210722095853563](D:\1书本笔记\java实战项目\image-20210722095853563.png)

针对字段、方法、构造器，因为信息量太大了，JDK还单独写了三个类，比如Method类：

![image-20210722100045871](D:\1书本笔记\java实战项目\image-20210722100045871.png)

也就是说原来的类经过加载后，其信息都被“解构”后，它的具体信息保存在Method,Constructor,Feild等类里面。比如Method类中的一个例子

![image-20210722100612165](D:\1书本笔记\java实战项目\image-20210722100612165.png)

#### 4 反射的使用方法

##### 1  加载类的class

Class.forName("类的相对路径")

比如：Class<?> class1 = Class.forName("com.app.Person");

当然不通过反射也可以通过：

​		//第二张方法：class 	Class<?> class2 = Person.class;  

​		//第三种方法：getClass 	Person person = new Person();   	Class<?> class3 = person.getClass();

##### 2 获取所有方法 getMethods()

​	//创建类 		Class<?> class1 = Class.forName("com.app.Person"); 		

​	//获取所有的公共的方法 		Method[] methods = 	class1.getMethods() ;

##### 3 获取所有的构造函数：getConstructors()

​	//创建类 		Class<?> class1 = Class.forName("com.app.Person"); 		

​	//获取所有的构造函数 		Constructor<?>[] constructors = class1.getConstructors() ;

##### 4 获取所有的属性：getDeclaredFields();

​			//创建类 		Class<?> class1 = Class.forName("com.app.Person"); 			

​		//取得本类的全部属性 		Field[] field = class1.getDeclaredFields();

##### 5  newinstance()方法

newinstance()方法是通过类的无参构造函数创造的。

​			//创建类 		Class<?> class1 = Class.forName("com.app.Person");

​			//创建实例化：相当于 new 了一个对象 		Object object = class1.newInstance() ; 	

  		//向下转型 		Person person = (Person) object ;

##### 6 getDeclaredFields 和 getFields 的区别

> getDeclaredFields()获得某个类的所有申明的字段，即包括public、private和proteced，但是不包括父类的申明字段。
>
> getFields()获得某个类的所有的公共（public）的字段，包括父类。

​			//创建类 		Class<?> class1 = Class.forName("com.app.Person");; 

​			//获得所有的字段属性：包括public、private和proteced  		Field[] declaredFields = class1.getDeclaredFields() ; 		

​      	//获得 public的字段属性  	Field[] fields = class1.getFields() ; 

#####  7 反射的实战

###### 1  创建对象实例

```java
public class Person{

	private String id ;

	private String name ;


	//构造函数1
	public Person( ){
		System.out.println( "构造函数  无参" );
	}

	//构造函数2
	public Person( String id ){
		this.id = id ;
		System.out.println( "构造函数 id : " + id );
	}

	//构造函数3
	public Person( String id  , String name ){
		this.id = id ;
		this.name = name ;
		System.out.println( "构造函数 id : " + id  + " name: " + name );
	}


	public String getId() {
		return id;
	}

	public void setId(String id) {
		this.id = id;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

}

	public static void main(String[] args) {

		try {
			
			//创建类
			Class<?> class1 = Class.forName("com.app.Person");

			//无参构造函数
			Object object = class1.newInstance() ;
			
			//有参构造函数：一个参数
			Constructor<?> constructor =  class1.getDeclaredConstructor( String.class ) ;
			constructor.newInstance( "1000" ) ;
			
			//有参构造函数：二个参数
			Constructor<?> constructor2 =  class1.getDeclaredConstructor( String.class , String.class ) ;
			constructor2.newInstance( "1001" , "jack" ) ;

		} catch (InstantiationException e) {
			e.printStackTrace();
		} catch (IllegalAccessException e) {
			e.printStackTrace();
		} catch (InvocationTargetException e) {
			e.printStackTrace();
		} catch (IllegalArgumentException e) {
			e.printStackTrace();
		} catch (NoSuchMethodException e) {
			e.printStackTrace();
		} catch (SecurityException e) {
			e.printStackTrace() ;
		} catch (ClassNotFoundException e) {
			e.printStackTrace();
		}

	}

结果：
构造函数  无参
构造函数 id : 1000
构造函数 id : 1001 name: jack    

```





###### 2 反射操作属性和方法

```java
//首先假设有个Person类
public class Person  implements InterFace {

	private String id ;
	private String name ;
	public String age ;
	
	//构造函数1
	public Person( ){
	}
	//构造函数2
	public Person( String id ){
		this.id = id ;
	}
	//构造函数3
	public Person( String id  , String name ){
		this.id = id ;
		this.name = name ;
	}
	public String getId() {
		return id;
	}
	public void setId(String id) {
		this.id = id;
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public String getAge() {
		return age;
	}
	public void setAge(String age) {
		this.age = age;
	}
	/**
	 * 静态方法
	 */
	public static void update(){
	}

	@Override
	public void read() {
	}

}

//反射实战
	public static void main(String[] args) {

		try {
			//创建类
			Class<?> class1 = Class.forName("com.app.Person");

			//创建实例
			Object person = class1.newInstance();

			//获得id 属性
			Field idField = class1.getDeclaredField( "id" ) ;

			//打破封装  实际上setAccessible是启用和禁用访问安全检查的开关,并不是为true就能访问为false就不能访问  
			//由于JDK的安全检查耗时较多.所以通过setAccessible(true)的方式关闭安全检查就可以达到提升反射速度的目的  
			idField.setAccessible( true );

			//给id 属性赋值
			idField.set(  person , "100") ;

			//获取 setName() 方法
			Method setName = class1.getDeclaredMethod( "setName", String.class ) ;
			//打破封装 
			setName.setAccessible( true );

			//调用setName 方法。
			setName.invoke( person , "jack" ) ;

			//获取name 字段
			Field nameField = class1.getDeclaredField( "name" ) ;
			//打破封装 
			nameField.setAccessible( true );

			//打印 person 的 id 属性值
			String id_ = (String) idField.get( person ) ;
			System.out.println( "id: " + id_ );

			//打印 person 的 name 属性值
			String name_ = ( String)nameField.get( person ) ;
			System.out.println( "name: " + name_ );
			
			//获取 getName 方法
			Method getName = class1.getDeclaredMethod( "getName" ) ;
			//打破封装 
			getName.setAccessible( true );
			
			//执行getName方法，并且接收返回值
			String name_2 = (String) getName.invoke( person  ) ;
			System.out.println( "name2: " + name_2 );

		} catch (IllegalArgumentException e) {
			e.printStackTrace();
		} catch (InvocationTargetException e) {
			e.printStackTrace();
		} catch (NoSuchMethodException e) {
			e.printStackTrace();
		} catch (InstantiationException e) {
			e.printStackTrace();
		} catch (IllegalAccessException e) {
			e.printStackTrace();
		} catch (NoSuchFieldException e) {
			e.printStackTrace();
		} catch (SecurityException e) {
			e.printStackTrace() ;
		} catch (ClassNotFoundException e) {
			e.printStackTrace();
		}

	}
结果：
id: 100
name: jack
name2: jack
```

###### 3 静态属性、静态方法调用

```java
//比如有一个工具类
package com.app;

public class Util {

	public static String name = "json" ;

	/**
	 * 没有返回值，没有参数
	 */
	public static void getTips(){
		System.out.println( "执行了---------1111");
	}

	/**
	 * 有返回值，没有参数
	 */
	public static String getTip(){
		System.out.println( "执行了---------2222");
		return "tip2" ;
	}

	/**
	 * 没有返回值，有参数
	 * @param name
	 */
	public static void getTip( String name ){
		System.out.println( "执行了---------3333 参数： " + name );
	}

	/**
	 * 有返回值，有参数
	 * @param id
	 * @return
	 */
	public static String getTip( int id ){
		System.out.println( "执行了---------4444 参数： " + id );
		if ( id == 0 ){
			return "tip1 444 --1 " ;
		}else{
			return "tip1 444 --2" ;
		}
	}

}


	public static void main(String[] args) {

		try {
			//创建类
			Class<?> class1 = Class.forName("com.app.Util");

			//获取 nameField 属性
			Field nameField = class1.getDeclaredField( "name" ) ;
			//获取 nameField 的值
			String name_ = (String) nameField.get( nameField ) ;
			//输出值
			System.out.println( name_ );


			//没有返回值，没有参数
			Method getTipMethod1 = class1.getDeclaredMethod( "getTips"  ) ; 
			getTipMethod1.invoke( null  ) ;
			
			//有返回值，没有参数
			Method getTipMethod2 = class1.getDeclaredMethod( "getTip"  ) ; 
			String result_2 = (String) getTipMethod2.invoke( null  ) ;
			System.out.println( "返回值： "+ result_2 );
			
			//没有返回值，有参数
			Method getTipMethod3 = class1.getDeclaredMethod( "getTip" , String.class  ) ; 
			String result_3 = (String) getTipMethod3.invoke( null , "第三个方法"  ) ;
			System.out.println( "返回值： "+ result_3 );
			
			//有返回值，有参数
			Method getTipMethod4 = class1.getDeclaredMethod( "getTip" , int.class ) ; 
			String result_4 = (String) getTipMethod4.invoke( null  , 1 ) ;
			System.out.println( "返回值： "+ result_4 );
			
		} catch (InvocationTargetException e) {
			e.printStackTrace();
		} catch (IllegalArgumentException e) {
			e.printStackTrace();
		} catch (NoSuchMethodException e) {
			e.printStackTrace();
		} catch (IllegalAccessException e) {
			e.printStackTrace();
		} catch (NoSuchFieldException e) {
			e.printStackTrace();
		} catch (SecurityException e) {
			e.printStackTrace() ;
		} catch (ClassNotFoundException e) {
			e.printStackTrace();
		}

	}

结果：
执行了---------1111
执行了---------2222
返回值： tip2
执行了---------3333 参数： 第三个方法
返回值： null
执行了---------4444 参数： 1
返回值： tip1 444 --2

```

###### 4 当参数是 int 类型 和 Integer 类型，反射获取方法不一样

- 当参数是 int 类型时

```
    /**
	 * 没有返回值，有参数
	 * @param id
	 */
	public static void getTip( int id ){
		
	}
```

获取方法的时候需要用：`int.class`。不能使用 `Integer.class`. 会报错。

```
Method getTipMethod4 = class.getDeclaredMethod( "getTip" , int.class ) ; 
String result_4 = (String) getTipMethod4.invoke( null  , 1 ) ;
System.out.println( "返回值： "+ result_4 );
```

- 当参数是 Integer类型时

```
    /**
	 * 没有返回值，有参数
	 * @param id
	 */
	public static void getTip( Integer id ){
		
	}
```

获取方法的时候需要用：`Integer .class`。不能使用 `int.class`. 会报错。

```
Method getTipMethod4 = class.getDeclaredMethod( "getTip" , Integer .class ) ; 
String result_4 = (String) getTipMethod4.invoke( null  , 1 ) ;
System.out.println( "返回值： "+ result_4 );
```

##### 8 总结：

- Class类提供了四个public方法，用于获取某个类的构造方法。

```
Constructor getConstructor(Class[] params)     根据构造函数的参数，返回一个具体的具有public属性的构造函数
Constructor getConstructors()     返回所有具有public属性的构造函数数组
Constructor getDeclaredConstructor(Class[] params)     根据构造函数的参数，返回一个具体的构造函数（不分public和非public属性）
Constructor getDeclaredConstructors()    返回该类中所有的构造函数数组（不分public和非public属性）
```

- 四种获取成员方法的方法

```
Method getMethod(String name, Class[] params)    根据方法名和参数，返回一个具体的具有public属性的方法
Method[] getMethods()    返回所有具有public属性的方法数组
Method getDeclaredMethod(String name, Class[] params)    根据方法名和参数，返回一个具体的方法（不分public和非public属性）
Method[] getDeclaredMethods()    返回该类中的所有的方法数组（不分public和非public属性）
```

- 四种获取成员属性的方法

```
Field getField(String name)    根据变量名，返回一个具体的具有public属性的成员变量
Field[] getFields()    返回具有public属性的成员变量的数组
Field getDeclaredField(String name)    根据变量名，返回一个成员变量（不分public和非public属性）
Field[] getDelcaredField()    返回所有成员变量组成的数组（不分public和非public属性）
```

### 2 反射的应用场景？

Ioc , 





### 3 反射的优缺点？

**优点：**

1 增加程序的灵活性，避免将程序写死到代码里

2 代码简洁，提高代码的复用率，外部调用方便**

3 对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法**

**缺点：**

**1 性能问题**

1.使用反射基本上是一种解释操作，用于字段和方法接入时要远慢于直接代码。因此Java反射机制主要应用在对灵活性和扩展性要求很高的系统框架上,普通程序不建议使用。

2.反射包括了一些动态类型，所以JVM无法对这些代码进行优化。因此，反射操作的效率要比那些非反射操作低得多。我们应该避免在经常被 执行的代码或对性能要求很高的程序中使用反射。

2 **使用反射会模糊程序内部逻辑**

程序人员希望在源代码中看到程序的逻辑，反射等绕过了源代码的技术，因而会带来维护问题。反射代码比相应的直接代码更复杂。

**3 安全限制**

使用反射技术要求程序必须在一个没有安全限制的环境中运行。如果一个程序必须在有安全限制的环境中运行，如Applet，那么这就是个问题了

4 **内部暴露**

由于反射允许代码执行一些在正常情况下不被允许的操作（比如访问私有的属性和方法），所以使用反射可能会导致意料之外的副作用－－代码有功能上的错误，降低可移植性。反射代码破坏了抽象性，因此当平台发生改变的时候，代码的行为就有可能也随着变化。


**Java反射可以访问和修改私有成员变量，那封装成private还有意义么？**
既然小偷可以访问和搬走私有成员家具，那封装成防盗门还有意义么？这是一样的道理，并且Java从应用层给我们提供了安全管理机制——安全管理器，每个Java应用都可以拥有自己的安全管理器，它会在运行阶段检查需要保护的资源的访问权限及其它规定的操作权限，保护系统免受恶意操作攻击，以达到系统的安全策略。所以其实反射在使用时，内部有安全控制，如果安全设置禁止了这些，那么反射机制就无法访问私有成员。





# 设计模式

### 1 单例模式

单例模式应该是23种设计模式中最简单的一种模式了。它有以下几个要素：

- 私有的构造方法
- 指向自己实例的私有静态引用
- 以自己实例为返回值的静态的公有的方法

```
/**
 * 单例模式
 *1 懒汉模式，只有在调用方法的时候，才会new实例对象
 * 线程不安全
 */
class Single1{
    private static Single1 a;

    public  static Single1 f(){
        if(a==null){
            a=new Single1();
        }
        return  a;
    }
}

/**
 * 单例模式
 *2 饿汉模式，一开始就new了实例对象
 * 线程安全
 */
class Single2{
    private static Single2 a=new Single2();

    public  static Single2 f(){

        return  a;
    }
}

/**
 *3 懒汉模式+双重校验：线程安全
 * 首先实例a应该为volatile修饰的，保证其不可被排序。
 * 其次：
 *      第一个if的作用：用来避免a已经被实例化后的加锁操作。
 *      第二个if的作用：是在if条件之后进行加锁的可以保证只有一个实例进入。
 */
class Single3{
    private volatile static Single3 a;

    public  static Single3 f(){
        if(a==null){
            synchronized (Single3.class) {
                if (a == null) {
                    a = new Single3();
                }
            }
        }
        return  a;
    }
}

/**
 * 4 用静态代码块的方式来初始化代码
 * 为什么用静态代码块是线程安全的？
 * 得益于JVM的类的初始化结构：
 * 首先要了解类加载过程中的最后一个阶段：即类的初始化，类的初始化阶本质就是执行类构造器的<clinit>方法。
 *
 * <clinit>方法：这不是由程序员写的程序，而是根据代码由javac编译器生成的。它是由类里面所有的类变量的赋值动作和静态代码块组成的。JVM内部会保证一个类的<clinit>方法在多线程环境下被正确的加锁同步，也就是说如果多个线程同时去进行“类的初始化”，那么只有一个线程会去执行类的<clinit>方法，其他的线程都要阻塞等待，直到这个线程执行完<clinit>方法。然后执行完<clinit>方法后，其他线程唤醒，但是不会再进入<clinit>()方法。也就是说同一个加载器下，一个类型只会初始化一次。
 *
 * 那么回到这个代码中，这里的静态变量的赋值操作进行编译之后实际上就是一个<clinit>代码，当我们执行getInstance方法的时候，会导致SingleTonHolder类的加载，类加载的最后会执行类的初始化，但是即使在多线程情况下，这个类的初始化的<clinit>代码也只会被执行一次，所以他只会有一个实例。
 *
 * 那么再增加一句，之所以这里变量定义的时候不需要volatile，因为只有一个线程会执行具体的类的初始化代码<clinit>，也就是即使有指令重排序，因为根本没有第二个线程给你去影响，所以无所谓。
 */
class Single4{
    private Single4(){

    }
    private  static  class Singleton{
        private static  final Single4 INSTANCE=new Single4();
    }

    public static Single4 getUniqueInstance(){
        return  Singleton.INSTANCE;
    }

}

/**
 * 6 用枚举来实现单例模式
 */
public enum Single5{
    INSTANCE;
}
Enum类不允许反序列化，为了保证枚举永远是单例的

	代码相当简洁，我们也可以像常规类一样编写enum类，为其添加变量和方法，访问方式也更简单，使用SingletonEnum.INSTANCE进行访问，这样也就避免调用getInstance方法，更重要的是使用枚举单例的写法，我们完全不用考虑序列化和反射的问题。枚举序列化是由jvm保证的，每一个枚举类型和定义的枚举变量在JVM中都是唯一的，在枚举类型的序列化和反序列化上，Java做了特殊的规定：在序列化时Java仅仅是将枚举对象的name属性输出到结果中，反序列化的时候则是通过java.lang.Enum的valueOf方法来根据名字查找枚举对象。同时，编译器是不允许任何对这种序列化机制的定制的并禁用了writeObject、readObject、readObjectNoData、writeReplace和readResolve等方法，从而保证了枚举实例的唯一性，这里我们不妨再次看看Enum类的valueOf方法：

```

### 2 工厂模式

![image-20210813105410573](D:\1书本笔记\java实战项目\image-20210813105410573.png)

特点：

1. 提供一种创建对象的最佳方式，在创建对象时不提供对外暴露创建逻辑，并且通过一个共同的接口
2. 来指向新创建的对象
3. 定义一个创建对象的接口，让子类来决定实例化哪一个具体的工厂类，延迟到子类去执行
4. 主要解决选择接口的问题
5. 扩展性高，只增加相应工厂类即可，知道名称即可创建对象，屏蔽具体的实现，调用者只关心接口
6. 增加需求时，需要增加具体类与工厂实现，导致类个数成倍增加，增加系统复杂度
7. 只有需要生成复杂类对象时才需要使用工厂模式，且简单工厂模式不属于23种设计模式  

#### 1 简单工厂模式

**简单工厂模式只有一个接口用来约束，用于被普通产品给实现。**

​	简单工厂模式属于创建型模式又叫做**静态工厂方法模式**，它属于类创建型模式。在简单工厂模式中，可以根据参数的不同返回不同类的实例。
​	**简单工厂模式专门定义一个类来负责创建其他类的实例，被创建的实例通常都具有共同的父类**。  

![image-20210813104005318](D:\1书本笔记\java实战项目\image-20210813104005318.png)

![image-20210813104057765](D:\1书本笔记\java实战项目\image-20210813104057765.png)

![image-20210813104107128](D:\1书本笔记\java实战项目\image-20210813104107128.png)

![image-20210813104116333](D:\1书本笔记\java实战项目\image-20210813104116333.png)

**优点：**
在不改变源代码的情况下，可以方便的进行扩展，比如突然增加了产品：面条。我们码，直接创建具体产品类实现抽象产品，在创建具体工厂类实现接口工厂，就可以通来得到我们新增加的产品了，符合了开闭原则。

**工厂类含有必要的判断逻辑**，可以决定在什么时候创建哪一个产品类的实例，客户端**产品对象的责任**，而仅仅“消费”产品；简单工厂模式通过这种做法**实现了对责任的分的工厂类用于创建对象**。

客户端无须知道所创建的具体产品类的类名，只需要知道具体产品类所**对应的参数**即的类名，通过简单工厂模式可以减少使用者的记忆量。通过引入配置文件，可以在不码的情况下更换和增加新的具体产品类，在一定程度上提高了系统的灵活性。



**缺点**：当业务需要的类型变多，目前只有食物，当产生饮料，日用品等类别时，我们来实现，造成代码重复的坏味道。并且代码臃肿。

适用环境

在以下情况下可以使用简单工厂模式：  

1. 工厂类负责创建的**对象比较少**：由于创建的对象较少，不会造成工厂方法中的业务逻辑太过复杂。
2. 客户端只知道传入工厂类的参数，对于如何创建对象不关心：客户端既不需要关心创建细节，甚至连类名都不需要记住，**只需要知道类型所对应的参数**。  

#### 2 工厂方法模式

![image-20210813135225033](D:\1书本笔记\java实战项目\image-20210813135225033.png)

（工厂方法模式，在我看来相当于在简单工厂模式的基础上，添加了**工厂类接口**，对调用者的实现更具体了）

定义：定义一个用于创建对象的接口，让子类决定实例化哪一个类，工厂方法使一个类的实例化延迟到其子类。

类型：创建类模式  

![image-20210813105247834](D:\1书本笔记\java实战项目\image-20210813105247834.png)

![image-20210813105259911](D:\1书本笔记\java实战项目\image-20210813105259911.png)



![image-20210813105314472](D:\1书本笔记\java实战项目\image-20210813105314472.png)

![image-20210813105329479](D:\1书本笔记\java实战项目\image-20210813105329479.png)

![image-20210813105343350](D:\1书本笔记\java实战项目\image-20210813105343350.png)

![image-20210813105355273](D:\1书本笔记\java实战项目\image-20210813105355273.png)



#### 3 抽象工厂模式

![image-20210813135237434](D:\1书本笔记\java实战项目\image-20210813135237434.png)

![image-20210813105710572](D:\1书本笔记\java实战项目\image-20210813105710572.png)

![image-20210813105757782](D:\1书本笔记\java实战项目\image-20210813105757782.png)

![image-20210813105813039](D:\1书本笔记\java实战项目\image-20210813105813039.png)

![image-20210813105827599](D:\1书本笔记\java实战项目\image-20210813105827599.png)

![image-20210813105840305](D:\1书本笔记\java实战项目\image-20210813105840305.png)

![image-20210813105850404](D:\1书本笔记\java实战项目\image-20210813105850404.png)

![image-20210813105858781](D:\1书本笔记\java实战项目\image-20210813105858781.png)

![image-20210813105914569](D:\1书本笔记\java实战项目\image-20210813105914569.png)



#### 4 抽象工厂模式与工厂方法模式的区别  

![image-20210813105739383](D:\1书本笔记\java实战项目\image-20210813105739383.png)

### 3 代理模式

![image-20210813135156122](D:\1书本笔记\java实战项目\image-20210813135156122.png)



#### 1 静态代理

- 定义：通过在代码中显示定义了一个业务实现类的代理，在代理类中实现了同名的被代理类的方法，通过调用代理类的方法，实现对被代理类方法的增强。代理和被代理对象在代理在代理之前是确定的，他们都是实现的相同的接口或者继承相同的抽象类

![preview](https://segmentfault.com/img/bVby06d?w=1063&h=464/view)

```java
/**
 * 被代理的接口类
 * @author zhiyuan.shen
 */
public interface Subject {

    /**
     * 具体方法
     */
    void doAction();

}

/**
 * @author zhiyuan.shen
 */
public class RealSubject implements Subject {
    @Override
    public void doAction() {
        System.out.println("service impl class.");
    }
}

/**
 * 代理类
 * @author zhiyuan.shen
 */
public class Proxy implements Subject {

    private Subject subject;

    public Proxy(Subject subject) {
        this.subject = subject;
    }

    @Override
    public void doAction () {
        System.out.println("before");
        subject.doAction();
        System.out.println("after");
    }

}
```

- 测试与应用类

```java
/**
 * @author zhiyuan.shen
 */
public class Test {

    public static void main(String[] args) {
        //创建服务类
        RealSubject realSubject = new RealSubject();
        //自己执行方法
        realSubject.doAction();

        System.out.println("----------");

        //创建代理类
        Proxy proxy = new Proxy(realSubject);
        //代理执行
        proxy.doAction();

    }
}
```

- 输出结果

```asciidoc
service impl class.
----------
before
service impl class.
after
```

- 静态代理角色介绍
  - 共同接口：真实的对象和代理类共同实现的接口，规范方法定义。
  - 真实对象：实现共同接口，可以独立运行，具备完整功能的对象。
  - 代理类：对真实对象的增强，组合了真实对象。

#### 2 动态代理

- 定义：通过接口中的方法名在动态生成的代理中动态调用实现类中的同名方法，一定是接口。JDK动态代理利用了JDK API，动态地在内存中构建代理对象，从而实现对目标对象的代理功能。

JDK中生成代理对象主要涉及的类和方法：

java.lang.reflect Proxy类，使用的方法：

```java
public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h)
```

这三个参数的含义：

- ClassLoader loader：目标对象的类加载器
- Class<?>[] interfaces：目标对象实现的接口
- InvocationHandler h：事件处理器，代理对象的具体代理操作

java.lang.reflect InvocationHandler接口，使用的方法：

```java
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable;
```

在invoke实现代理类的具体代理操作。

所谓Dynamic Proxy是这样一种class:

- 他是在运行时生成的class
- 该class需要实现一组interface
- 使用动态代理类时，必须实现InvocationHandler接口
- 实现JDK动态代理的步骤
  1. 创建一个实现接口InvocationHandler的类，它必须实现invoke方法
  2. 创建被代理的类以及接口
  3. 调用Proxy的静态方法，创建一个代理类 newProxyInstance(ClassLoader loader, Class[] interfaces, InvocationHandler h)
  4. 通过代理调用方法
- 示例

```java
/**
 * 移动的车接口
 * @author zhiyuan.shen 
 */
public interface MoveAble {
    void move();
}

/**
 * 具体的实现类
 * @author zhiyuan.shen
 */
public class Car implements MoveAble {
    /**
     * 实现开车
     */
    @Override
    public void move() {
        try {
            System.out.println("car moving.");
            Thread.sleep(new Random().nextInt(1000));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

/**
 * 时间处理器
 * @author zhiyuan.shen
 */
public class TimeHandler implements InvocationHandler{

    private Object target;

    public TimeHandler(Object target) {
        this.target = target;
    }

    /**
     *
     * @param proxy 被代理的对象
     * @param method 被代理对象的方法
     * @param args 被代理方法的参数
     * @return 方法的返回值
     * @throws Throwable
     */
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        long startTime = System.currentTimeMillis();
        System.out.println("car start...");
        method.invoke(target);
        System.out.println("car end...");
        long endTime = System.currentTimeMillis();
        System.out.println("used " + (endTime - startTime) + " ms！");
        return null;
    }

}
```

- 测试与应用类

```java
public class Main {

    public static void main(String[] args) {
        Car car = new Car();

        InvocationHandler handler = new TimeHandler(car);
        Class<?> clazz = car.getClass();
        /**
         * loader 类加载器
         * interfaces 实现接口
         * InvocationHandler
         *
         * ---动态代理实现思路---
         * 实现功能：通过Proxy的newProxyInstance返回代理对象
         * 1.声明一段源码（动态产生代理）
         * 2.编译源码（JDK Complider API）, 产生一个新的类（代理类）
         * 3.将这个类load到内存当中，产生一个新的对象（代理对象）
         * 4.return 代理对象
         */
        MoveAble moveAble = (MoveAble)Proxy.newProxyInstance(clazz.getClassLoader(), clazz.getInterfaces(), handler);
        moveAble.move();
    }
}
```

- 输出结果

```gams
car start...
car end...
used 120 ms?
```

### 4 适配器模式

#### 1 类的适配器模式

在软件开发中采用类似于电源适配器的设计和编码技巧被称为适配器模式。
通常情况下，客户端可以通过目标类的接口访问它所提供的服务。有时，现有的类可以满足客户类的功能需要，但是它所提供的接口不一定是客户类所期望的，这可能是因为现有类中方法名与目标类中定义的方法名不一致等原因所导致的。

在这种情况下，现有的接口需要转化为客户类期望的接口，这样保证了对现有类的重用。如果不进行这样的转化，客户类就不能利用现有类所提供的功能，适配器模式可以完成这样的转化。

在适配器模式中可以定义一个包装类，包装不兼容接口的对象，这个包装类指的就是适配器(Adapter)，它所包装的对象就是适配者(Adaptee)，即被适配的类。  

适配器提供客户类需要的接口，适配器的实现就是把客户类的请求转化为对适配者的相应接口的调用。也就是说：当客户类调用适配器的方法时，在适配器类的内部将调用适配者类的方法，而这个过程对客户类是透明的，客户类并不直接访问适配者类。因此，适配器可以使由于接口不兼容而不能交互的类可以一起工作。这就是适配器模式的模式动机  

![image-20210813135135758](D:\1书本笔记\java实战项目\image-20210813135135758.png)

![image-20210813140329031](D:\1书本笔记\java实战项目\image-20210813140329031.png)

![image-20210813140347090](D:\1书本笔记\java实战项目\image-20210813140347090.png)

![image-20210813140404259](D:\1书本笔记\java实战项目\image-20210813140404259.png)

#### 2 对象的适配器模式

![image-20210813140510922](D:\1书本笔记\java实战项目\image-20210813140510922.png)

![image-20210813140601071](D:\1书本笔记\java实战项目\image-20210813140601071.png)

![image-20210813140617261](D:\1书本笔记\java实战项目\image-20210813140617261.png)

![image-20210813140630301](D:\1书本笔记\java实战项目\image-20210813140630301.png)

![image-20210813140642074](D:\1书本笔记\java实战项目\image-20210813140642074.png)

![image-20210813140651772](D:\1书本笔记\java实战项目\image-20210813140651772.png)

### 5 原型模式

![image-20210813142729978](D:\1书本笔记\java实战项目\image-20210813142729978.png)

#### 1 简单形式的原型模式

![image-20210813142803406](D:\1书本笔记\java实战项目\image-20210813142803406.png)

![image-20210813142812431](D:\1书本笔记\java实战项目\image-20210813142812431.png)

![image-20210813142824912](D:\1书本笔记\java实战项目\image-20210813142824912.png)

![image-20210813142837867](D:\1书本笔记\java实战项目\image-20210813142837867.png)

#### 2  登记形式的原型模式  

![image-20210813142930285](D:\1书本笔记\java实战项目\image-20210813142930285.png)

作为原型模式的第二种形式，它多了一个原型管理器(PrototypeManager)角色，该角色的作用是：创建具体原型类的对象，并记录每一个被创建的对象  

![image-20210813143012359](D:\1书本笔记\java实战项目\image-20210813143012359.png)

![image-20210813143025901](D:\1书本笔记\java实战项目\image-20210813143025901.png)

![image-20210813143042130](D:\1书本笔记\java实战项目\image-20210813143042130.png)

![image-20210813143058227](D:\1书本笔记\java实战项目\image-20210813143058227.png)

![image-20210813143113516](D:\1书本笔记\java实战项目\image-20210813143113516.png)

![image-20210813143128706](D:\1书本笔记\java实战项目\image-20210813143128706.png)

### 6 装饰者模式  

定义：动态地给一个对象添加一些额外的职责，就增加功能来说，装饰者模式比生成子类更为灵活。  

![image-20210813143301848](D:\1书本笔记\java实战项目\image-20210813143301848.png)

![image-20210813143312578](D:\1书本笔记\java实战项目\image-20210813143312578.png)

![image-20210813143959995](D:\1书本笔记\java实战项目\image-20210813143959995.png)

![image-20210813144009233](D:\1书本笔记\java实战项目\image-20210813144009233.png)

![image-20210813144019735](D:\1书本笔记\java实战项目\image-20210813144019735.png)

![image-20210813144030247](D:\1书本笔记\java实战项目\image-20210813144030247.png)

![image-20210813144040108](D:\1书本笔记\java实战项目\image-20210813144040108.png)

![image-20210813144050124](D:\1书本笔记\java实战项目\image-20210813144050124.png)

### 7 建造者模式

![image-20210813144214776](D:\1书本笔记\java实战项目\image-20210813144214776.png)

### 8 备忘录模式

### 9  策略模式

### 10  观察者模式

