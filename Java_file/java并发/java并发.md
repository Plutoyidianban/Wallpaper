# java并发

## 1 进程和线程基础

### 1 线程的理解，线程和进程的区别？

进程：**进程就是应⽤程序在内存中分配的空间**，也就是正在运⾏的程序  ,进程让操作系统的并发成为了可能  

线程: **进程让操作系统的并发性成为了可能，⽽线程让进程的内部并发成为了可能** . 

**多进程⽅式确实可以实现并发，但使⽤多线程，有以下⼏个好处：**
	进程间的通信⽐较复杂，⽽线程间的通信⽐较简单，通常情况下，我们需要使
	⽤共享资源，这些资源在线程间的通信⽐较容易。
	进程是重量级的，⽽线程是轻量级的，故多线程⽅式的系统开销更⼩  

**区别：**

​	进程是⼀个独⽴的运⾏环境，⽽线程是在进程中执⾏的⼀个任务。他们两个本质的区别是是否单独占有内存地址空间及其它系统资源（⽐如I/O）  

1. 进程单独占有⼀定的内存地址空间，所以进程间存在内存隔离，数据是分开的，数据共享复杂但是同步简单，各个进程之间互不⼲扰；⽽线程共享所属进程占有的内存地址空间和资源，数据共享简单，但是同步复杂 。
2.  进程单独占有⼀定的内存地址空间，⼀个进程出现问题不会影响其他进程，不影响主程序的稳定性，可靠性⾼；⼀个线程崩溃可能影响整个程序的稳定性，可靠性较低。  
3. 进程单独占有⼀定的内存地址空间，进程的创建和销毁不仅需要保存寄存器和栈信息，还需要资源的分配回收以及⻚调度，开销较⼤；线程只需要保存寄存器和栈信息，开销较⼩。  

另外⼀个重要区别是，**进程是操作系统进⾏资源分配的基本单位**，⽽**线程是操作系统进⾏调度的基本单位**，即CPU分配时间的单位 。  

### 2 线程的生命周期状态转移

#### 1 线程状态的转换

![image-20210727143035662](D:\1书本笔记\java实战项目\image-20210727143035662.png)

##### 1 NEW

一般调用start()方法，就直接进去new状态了

##### 2 RUNNABLE

##### 3 BLOCKED

##### 4 WAITING

等待状态。处于等待状态的线程变成RUNNABLE状态**需要其他线程唤醒**  

调⽤如下3个⽅法会使线程进⼊等待状态：

1. Object.wait()：使当前线程处于等待状态直到另⼀个线程唤醒它；
2. Thread.join()：等待线程执⾏完毕，底层调⽤的是Object实例的wait⽅法；
3. LockSupport.park()：除⾮获得调⽤许可，否则禁⽤当前线程进⾏线程调度。  

##### 5 TIMED_WAITING

超时等待状态。线程等待⼀个具体的时间，时间到后会被⾃动唤醒  

调⽤如下⽅法会使线程进⼊超时等待状态：

1. Thread.sleep(long millis)：使当前线程睡眠指定时间；  

2. Object.wait(long timeout)：线程休眠指定时间，等待期间可以通过notify()/notifyAll()唤醒；
3. Thread.join(long millis)：等待当前线程最多执⾏millis毫秒，如果millis为0，则会⼀直执⾏；
4. LockSupport.parkNanos(long nanos)： 除⾮获得调⽤许可，否则禁⽤当前线程进⾏线程调度指定时间；
5. LockSupport.parkUntil(long deadline)：同上，也是禁⽌线程进⾏调度指定时间；  

##### 6 TERMINATED

终⽌状态。此时线程已执⾏完毕。  



### 3 yield()与wait()的区别？

**1 wait() 方法** 

​		wait 方法是属于 **Object** 类中的，wait 过程中线程**会释放对象锁**，只有当其他线程调用 **notify** 才能唤醒此线程。**wait 使用时必须先获取对象锁，即必须在 synchronized 修饰的代码块中使用**，那么相应的 **notify 方法**同样必须在 **synchronized 修饰的代码块中使用**，如果没有在synchronized 修饰的代码块中使用时运行时会抛出IllegalMonitorStateException的异常

**2 yield() 方法**

​	和 sleep 一样都是 **Thread 类**的方法，都是暂停当前正在执行的线程对象，**不会释放资源锁**，和 **sleep 不同的是 yield方法并不会让线程进入阻塞状态**，而是让线程**重回就绪状态**，它只需要等待重新获取CPU执行时间，所以**执行yield()的线程有可能在进入到可执行状态后马上又被执行**。还有一点和 **sleep 不同的是 yield 方法只能使同优先级或更高优先级的线程有执行的机会**


### 4 sleep()、wait()、join()方法的区别？

**1 sleep()方法**

​	sleep 方法是属于 **Thread** 类中的，sleep 过程中线程**不会释放锁**，只会***\*阻塞线程\****，让出cpu给其他线程，但是他的监控状态依然保持着，当指定的时间到了又会自动恢复运行状态，***\*可中断\****，sleep 给其他线程运行机会时不考虑线程的优先级，因此***\*会给低优先级的线程以运行的机会\****

**2 wait() 方法** 

​		wait 方法是属于 **Object** 类中的，wait 过程中线程**会释放对象锁**，只有当其他线程调用 **notify** 才能唤醒此线程。**wait 使用时必须先获取对象锁，即必须在 synchronized 修饰的代码块中使用**，那么相应的 **notify 方法**同样必须在 **synchronized 修饰的代码块中使用**，如果没有在synchronized 修饰的代码块中使用时运行时会抛出IllegalMonitorStateException的异常

**3 join() 方法**

等待调用join方法的线程结束之后，程序再继续执行，一般用于***\*等待异步线程执行完结果之后才能继续运行的场景\****。例如：主线程创建并启动了子线程，如果子线程中药进行大量耗时运算计算某个数据值，而主线程要取得这个数据值才能运行，这时就要用到 join 方法了

### 5 run和start()的区别？

1. 调用 **start() 方法是用来启动线程的**，**轮到该线程执行时**，会自动调用 run()；**直接调用 run() 方法，无法达到启动多线程的目的**，相当于主线程线性执行 Thread 对象的 run() 方法。
2. **一个线程对线的 start() 方法只能调用一次**，多次调用会抛出 java.lang.IllegalThreadStateException 异常；run() 方法没有限制。

例子：

![image-20210727153503203](D:\1书本笔记\java实战项目\image-20210727153503203.png)

![image-20210727153512981](D:\1书本笔记\java实战项目\image-20210727153512981.png)

**由此可以看出，直接调用run方法，无法达到启动多个线程的目的的，而start()方法，可以**



#### 1 反复调用同一个线程的start() 方法，是否可行？

​	不可以，因为start()函数的源码里面，一开始有一个threadStatus的判断条件，当线程第一次启动的时候，threadStatus为0，可以正常启动，但是当线程第二次启动的时候，threadStatus不为0，直接抛出异常了。

![image-20210727144955853](D:\1书本笔记\java实战项目\image-20210727144955853.png)

#### 2 假如一个线程执行完毕（此时处于Terminated状态），再次调用这个线程的start()方法，是否可行？

不可以，因为start()函数的源码里面，一开始有一个threadStatus的判断条件，当线程第一次启动的时候，threadStatus为0，可以正常启动，但是当线程第二次启动的时候，threadStatus不为0，直接抛出异常了。

![image-20210727144955853](D:\1书本笔记\java实战项目\image-20210727144955853.png)



### 6 join(long) 与 sleep(long)的区别？

　    sleep(long)方法在睡眠时不释放对象锁

　　join(long)方法在等待的过程中释放对象锁

### 7 os中的线程模型？

堆和方法区里面放的都是各个线程共享的变量，而本地方法栈，程序计数器，虚拟机栈都是各个线程共享的。

![image-20210727162849086](D:\1书本笔记\java实战项目\image-20210727162849086.png)



### 8 os中如何调度线程？

**CPU一般会有多个核心，每个核心都调度一个线程执行**。

CPU有几个核心，最多同时可调度几个线程(多核能让电脑更快就是这个原理)。

OS的功能就是要在合适的时候分配CPU核心来调度合适的线程。

为了能实现多任务并发，OS不允许一个OS核心长期固定调度一个线程。

**OS是如何调度CPU核心来执行各个线程呢？**



**OS会根据线程的优先级分配每次调度最多执行的时间片，这个时间一到，无论如何都要重新调度一次线程**（也许还是调度到这个线程，这个不重要）。

除了时间片以外，线程会等待某些条件（磁盘读取文件，网卡发送完数据，线程休眠， 等待用户操作）这样也会把这个线程挂起，OS会重新找一个新的线程继续执行，只到挂起的这个线程的条件满足了，重新把这个线程放到可调度队列里面，这个线程又有机会被OS调度CPU核心来执行。

当我们打开电脑的任务管理器，你会发现很多线程的CPU占有率为0%, 说明这些线程都由于某些条件而挂起了，没有被OS调度。

每个线程“随时随地”都可能被OS中断执行，并调度到其它的线程执行。

**OS是如何保证一个线程在调度出去后，再重新调度回来能继续之前的数据状态来执行呢？**



**OS是这么做的：每个线程都会有一个运行时的环境（**运行时CPU的**每个寄存器的值、栈独立。栈的内存数据不会变**。**数据段、堆共用，可能调度**回来会**变**）。

当OS要把某个CPU核心调度出去给其它线程的时候，首先会把当前线程的运行环境（寄存器的值等）保存到内存，然后调度到其它线程，等再次调度回来的时候，再把原来保存到内存的寄存器的值，再设置会CPU核心的寄存器里面，这样就回到了调度出去之前的进度。

因为多线程之间共用了代码段（代码段只读，不会改），数据段(全局变量调度回来后，可能被其它线程篡改，不是调度之前的那个值了)，堆(调度回来后，动态内存分配的对象内存数据可能被其它线程出篡改)，调度回来后，栈上的数据是不变的，因为每个线程都有自己的栈空间。线程调度前后哪些会变，哪些不变你要清楚。这样你写多线程代码的时候才能清晰。

线程调度的开销就是：保存上下文执行环境，内核态运行算法决定接下来调度那个线程，切换这个线程的上下文环境。

### 9 线程锁的核心原理是什么?

​		多线程切换的时候，栈、代码段的数据不会变，数据段与堆的数据切换前后可能会发生改变，这个就造成了"竞争", 如果某些关键数据，在执行代码的时候，不允许这种竞争性的改变，怎么办呢？这个时候多线程就给了一个机制，这个机制就是锁，那么锁的原理是什么？接下来我来这你详细讲解。

![image-20210727193457647](D:\1书本笔记\java实战项目\image-20210727193457647.png)

​		当线程A调用FuncA()，线程B也调用FUNCA(),OS如何设计锁能保证他们竞争的唯一性的呢？我们把具体过程来分析一下。

假设线程A调用funcA()；它获取了锁，执行到中间某个代码的时候，时间片用完了，被OS调度出去，OS调度线程B来执行funcA(), 当线程B跑到lock(锁)，发现这个锁已经被线程A拿了，此时**，线程B会主动把自己挂起到锁这个“事件”上(等着锁释放)**。

​		OS从新调度线程执行，当重新调度到线程A的时候，线程A执行，执行完成以后，释放掉这个锁，那么线程B又从等待这个锁的队列，到线程调度的就绪队列，又可被OS调度到，等线程A调度出去后，线程B去lock这个锁，就占用了这个锁，然后继续执行。这样就保证了lock/unlock之间的代码永远只有一个线程跑进去了。这样保护了这段代码里面相关的数据和逻辑。

### 10 用户线程和内核线程的区别？

线程的实现可以分两类：**用户级线程，内核级线程**和混合式线程。

**用户级线程**是指不需要内核支持而在用户程序中实现的线程，它的内核的切换是由用户态程序自己控制内核的切换，不需要内核的干涉。但是它不能像内核级线程一样更好的运用多核CPU。

优点：

（1） 线程的调度不需要内核直接参与，控制简单。

（2） 可以在不支持线程的操作系统中实现。

（3） 同一进程中只能同时有一个线程在运行，如果有一个线程使用了系统调用而阻塞，那么整个进程都会被挂起，可以节约更多的系统资源。

缺点：

（1） **一个用户级线程的阻塞将会引起整个进程的阻塞**。

（2） 用户级线程不能利用系统的多重处理，仅有一个用户级线程可以被执行。

**内核级线程**:切换由内核控制，当线程进行切换的时候，由用户态转化为内核态。切换完毕要从内核态返回用户态。可以很好的运用多核CPU，就像Windows电脑的四核八线程，双核四线程一样。

优点：

（1）**当有多个处理机时，一个进程的多个线程可以同时执行**。

（2） 由于内核级线程只有很小的数据结构和堆栈，切换速度快，当然它本身也可以用多线程技术实现，提高系统的运行速率。

缺点：

（1） 线程在用户态的运行，而线程的调度和管理在内核实现，在控制权从一个线程传送到另一个线程需要用户态到内核态再到用户态的模式切换，比较占用系统资源。（就是必须要受到内核的监控）

关联性

（1） 它们之间的差别在于性能。

（2） 内核支持线程是OS内核可感知的，而用户级线程是OS内核不可感知的。

（3） 用户级线程的创建、撤消和调度不需要OS内核的支持。

（4） 用户级线程执行系统调用指令时将导致其所属进程被中断，而内核支持线程执行系统调用指令时，只导致该线程被中断。

（5） 在只有用户级线程的系统内，CPU调度还是以进程为单位，处于运行状态的进程中的多个线程，由用户程序控制线程的轮换运行；在有内核支持线程的系统内，CPU调度则以线程为单位，由OS的线程调度程序负责线程的调度。

（6） 用户级线程的程序实体是运行在用户态下的程序，而内核支持线程的程序实体则是可以运行在任何状态下的程序。

### 11 简述对协程的认识？ 

为什么需要协程的存在？

​		我们知道操作系统在线程等待IO的时候，会阻塞当前线程，切换到其它线程，这样在当前线程等待IO的过程中，其它线程可以继续执行。当系统线程较少的时候没有什么问题，但是当线程数量非常多的时候，却产生了问题。**一是系统线程会占用非常多的内存空间，二是过多的线程切换会占用大量的系统时间。**

![image-20210727210907647](D:\1书本笔记\java实战项目\image-20210727210907647.png)

![image-20210727210917071](D:\1书本笔记\java实战项目\image-20210727210917071.png)

​		协程刚好可以解决上述2个问题。协程运行在线程之上，当一个协程执行完成后，可以选择主动让出，让另一个协程运行在当前线程之上。**协程并没有增加线程数量，只是在线程的基础之上通过分时复用的方式运行多个协程**，而且协程的切换在用户态完成，切换的代价比线程从用户态到内核态的代价小很多。

**协程的注意事项**

实际上协程并不是什么银弹，协程只有在等待IO的过程中才能重复利用线程，上面我们已经讲过了，线程在等待IO的过程中会陷入阻塞状态，意识到问题没有？

假设协程运行在线程之上，并且协程调用了一个阻塞IO操作，这时候会发生什么？实际上操作系统并不知道协程的存在，它只知道线程，**因此在协程调用阻塞IO操作的时候，操作系统会让线程进入阻塞状态，当前的协程和其它绑定在该线程之上的协程都会陷入阻塞而得不到调度，这往往是不能接受的。**

因此在协程中不能调用导致线程阻塞的操作。也就是说，**协程只有和异步IO结合起来，才能发挥最大的威力**。

那么如何处理在协程中调用阻塞IO的操作呢？一般有2种处理方式：

1. **在调用阻塞IO操作的时候，重新启动一个线程去执行这个操作，等执行完成后，协程再去读取结果。这其实和多线程没有太大区别。**
2. **对系统的IO进行封装，改成异步调用的方式，这需要大量的工作，最好寄希望于编程语言原生支持。**

协程对计算密集型的任务也没有太大的好处，计算密集型的任务本身不需要大量的线程切换，因此协程的作用也十分有限，反而还增加了协程切换的开销。

以上就是协程的注意事项。这里顺带一提JavaScript的异步变同步的调用方式，如果协程能够实现该类型的语法，不仅可以把异步操作变为同步，同时在IO操作的时候还能够不占用CPU，写起来非常方便。

**异步变同步的调用方式只是一种编程方式，不管是用线程还是用协程都可以实现这种编程方式，好处是不用在处理非常多的回调。**

#### 为什么协程的切换很廉价

关于这个问题，我找了很多资料，得到的答案都没有太高的说服力，我了解了一下线程切换的过程：

- 线程在进行切换的时候，需要将CPU中的寄存器的信息存储起来，然后读入另外一个线程的数据，这个会花费一些时间
- CPU的高速缓存中的数据，也可能失效，需要重新加载
- 线程的切换会涉及到用户模式到内核模式的切换，据说每次模式切换都需要执行上千条指令，很耗时。

协程的切换之所以快，可能的原因是：

- 在切换的时候，寄存器需要保存和加载的数据量比较小。
- 高速缓存可以有效利用
- 没有用户模式到内核模式的切换操作。
- 更有效率的调度，因为协程是非抢占式的，前一个协程执行完毕或者堵塞，才会让出CPU，而线程则一般使用了时间片的算法，会进行很多没有必要的切换（为了尽量让用户感知不到某个线程卡）。



### 12 java线程本质？

Java创建线程的步骤：

**Java是通过JVM调用OS系统创建线程**

Thread.start()经历了：Thread.start() --> navtive start0() --> JVM ---> Thread.c : start0() --> JVM 实例化一个C++对象 JavaThread --> OS pthread_create() 创建线程  --> 线程创建完毕 --> JVM --> Thread.run()方法

Java级别中的**线程其实就是操作系统级别的线程**






![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/37bbb07687c24aa690f23e9134bf37ad~tplv-k3u1fbpfcp-zoom-1.image)

### 13 如何判断线程是否持有对象锁？

Thread类中有一个方法：Thread.holdsLock(lock)来检测线程是否持有锁

### 14 如何结束线程？

\1.  以上，我们可以看出，interrupt方法不一定会真正”中断”线程，它只是一种协作机制，如果 不明白线程在做什么，不应该贸然的调用线程的interrupt方法，以为这样就能取消线程。

\2.  对于以线程提供服务的程序模块而言，它应该封装取消/关闭操作，提供单独的取消/关闭方法给调用者，类似于InterruptReadDemo中演示的cancel方法，外部调用者应该调用这些方法而不是直接调用interrupt。

\3.  Java并发库的一些代码就提供了单独的取消/关闭方法，比如说，Future接口提供了如下方法以取消任务：boolean cancel(boolean mayInterruptIfRunning);

\4.  再比如，ExecutorService提供了如下两个关闭方法：

```text
void shutdown();
List<Runnable> shutdownNow();
```

\5.  Future和ExecutorService的API文档对这些方法都进行了详细说明，这是我们应该学习的方式。



在java中有以下3种方法可以终止正在运行的线程：

1. 使用退出标志，使线程正常退出，也就是当run方法完成后线程终止。
2. 使用stop方法强行终止，但是不推荐这个方法，因为stop和suspend及resume一样都是过期作废的方法。
3. 使用interrupt方法中断线程。

##### **1. 停止不了的线程**

**interrupt()方法**的使用效果并不像for+break语句那样，马上就停止循环。调用interrupt方法是在当前线程中打了一个停止标志，**并不是真的停止线程。**，**interrupt()不能中断在运行中的线程，它只能改变中断状态而已。**

![image-20210727215256108](D:\1书本笔记\java实战项目\image-20210727215256108.png)

##### **2. 判断线程是否停止状态**

https://www.cnblogs.com/greta/p/5624839.html

Thread.java类中提供了两种方法：

1. this.interrupted(): 测试当前线程是否已经中断；
2. this.isInterrupted(): 测试线程是否已经中断；

那么这两个方法有什么图区别呢？
我们先来看看this.interrupted()方法的解释：测试当前线程是否已经中断，当前线程是指运行this.interrupted()方法的线程。

1. 方法interrupted()的确判断出当前线程是否是停止状态。但为什么第2个布尔值是false呢？ 官方帮助文档中对interrupted方法的解释：**测试当前线程是否已经中断。线程的中断状态由该方法清除。** 换句话说，如果连续两次调用该方法，则第二次调用返回false。
2. isInterrupted()方法,判断当前线程是否中断，不会改变线程的中断状态。

（总结：**interrupt()**方法，**interrupted()**方法，**sleep()**方法，会改变线程的中断状态，inInterrupted()方法不会改变线程的中断状态。）

##### 3  能停止的线程--异常法

异常法，应该是最常见的方法。

![image-20210727220731003](D:\1书本笔记\java实战项目\image-20210727220731003.png)

##### 4 在沉睡中停止

![image-20210727220758913](D:\1书本笔记\java实战项目\image-20210727220758913.png)

从打印的结果来看， 如果在**sleep状态下停止某一线程**，会进入catch语句，并且**清除停止状态值**，使之变为false。

##### 5 为什么不用stop方法停止线程

​		调用stop()方法时会抛出java.lang.ThreadDeath异常

​		**stop()方法**以及作废，因为如果强制让线程停止有可能使一些清理性的工作得不到完成。另外一个情况就是对**锁定的对象进行了解锁**，导致数据得不到同步的处理，出现**数据不一致**的问题。

​		使用**stop()释放锁**将会给**数据造成不一致性**的结果。如果出现这样的情况，**程序处理的数据**就有可能遭到破坏，最终导致程序执行的流程错误，一定要特别注意：

```java
exmple:
public class SynchronizedObject {
    private String name = "a";
    private String password = "aa";

    public synchronized void printString(String name, String password){
        try {
            this.name = name;
            Thread.sleep(100000);
            this.password = password;
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}

public class MyThread extends Thread {
    private SynchronizedObject synchronizedObject;
    public MyThread(SynchronizedObject synchronizedObject){
        this.synchronizedObject = synchronizedObject;
    }

    public void run(){
        synchronizedObject.printString("b", "bb");
    }
}

public class Run {
    public static void main(String args[]) throws InterruptedException {
        SynchronizedObject synchronizedObject = new SynchronizedObject();
        Thread thread = new MyThread(synchronizedObject);
        thread.start();
        Thread.sleep(500);
        thread.stop();
        System.out.println(synchronizedObject.getName() + "  " + synchronizedObject.getPassword());
    }
}

result:
b  aa
```



##### 6 也可以使用return来终止线程

将方法interrupt()与return结合使用也能实现停止线程的效果：

```java
public class MyThread extends Thread {
    public void run(){
        while (true){
            if(this.isInterrupted()){
                System.out.println("线程被停止了！");
                return;
            }
            System.out.println("Time: " + System.currentTimeMillis());
        }
    }
}

public class Run {
    public static void main(String args[]) throws InterruptedException {
        Thread thread = new MyThread();
        thread.start();
        Thread.sleep(2000);
        thread.interrupt();
    }
}

result:
Time: 1467072288503
Time: 1467072288503
Time: 1467072288503
线程被停止了！
```

### 15 线程的中断(interrupt)机制

链接：

### 15  为什么要在InterrupttedException的时候清除掉中断状态？

​		这个问题没有找到官方的解释，估计只有Java设计者们才能回答了。但这里的解释似乎比较合理：一个中断应该只被处理一次（你catch了这个InterruptedException，说明你能处理这个异常，你不希望上层调用者看到这个中断）。



### 16  使用Interrupt一定会抛出异常吗？

1. 如果线程被Object.wait, Thread.join和Thread.sleep三种方法之一阻塞，此时调用该线程的interrupt()方法，那么该线程将抛出一个 InterruptedException中断异常（该线程必须事先预备好处理此异常），从而提早地终结被阻塞状态。(**抛出异常**)
2. 如果线程没有被阻塞，这时调用 interrupt()将不起作用，直到执行到wait(),sleep(),join()时,才马上会抛出 InterruptedException。（**不抛出异常**）

***\*（总结一下：调用interrupt()方法，立刻改变的是中断状态，但如果不是在阻塞态，就不会抛出异常；如果在进入阻塞态后，中断状态为已中断，就会立刻抛出异常）\****

1. sleep() &interrupt()

线程A正在使用sleep()暂停着: Thread.sleep(100000)，如果要**取消它的等待状态**,可以在正在执行的线程里(比如这里是B)调用a.interrupt()［a是线程A对应到的Thread实例］，**令线程A放弃睡眠操作**。即，在线程B中执行a.interrupt()，处于阻塞中的线程a将放弃睡眠操作。

当在sleep中时线程被调用interrupt()时，就马上会放弃暂停的状态并抛出InterruptedException。抛出异常的,是A线程。

2. wait() &interrupt()

线程A调用了wait()进入了等待状态，**也可以用interrupt()取消**。不过这时候要注意锁定的问题。线程在进入等待区,会把锁定解除,当对等待中的线程调用interrupt()时，会先重新获取锁定，再抛出异常。***\*在获取锁定之前，是无法抛出异常的。\****

3. join() &interrupt()

当线程以join()等待其他线程结束时，当它被调用**interrupt()**，它与sleep()时一样，会马上跳到catch块里.。

**注意，调用的interrupt()方法，一定是调用被阻塞线程的interrupt方法。如在线程a中调用线程t.interrupt()。**

### 17 什么场景适合多线程，什么场景适合单线程？

##### 1 应用场景

多线程 ： 略

redis 基本都是单线程的

##### 2 多项程是不是肯定比单线程好？

多线程编程的缺点：
（1）线程切换是有开销的，这会导致程序运行变慢。
（2）多线程程序必须非常小心地同步代码，否则会引起死锁。
（3）多线程程序极难调试，并且一些bug非常隐蔽，可能你99次运行都是对的，但是有1次是错的。不像单线程程序那么容易暴露问题。

### 18 如何减少上下文切换？

减少上下文切换的方法有无锁并发编程、CAS算法、使用最少线程和使用协程。

- **无锁并发编程**。多线程竞争锁时，会引起上下文切换，所以多线程处理数据时，可以使用一些方法来避免使用锁。如将数据的ID按照Hash算法取模分段，不同的线程处理不同段的数据。
- **CAS算法**。Java的Atomic包使用CAS算法来更新数据，而不需要加锁。
- **使用最少线程**。避免创建不需要的线程，比如任务很少，但是创建了很多线程来处理，这样会造成大量线程都处于等待状态。
- **协程**。在单线程里实现多任务的调度，并在单线程里维持多个任务间的切换。



### 19 并行和并发的区别

**并发（Concurrent），在操作系统中，是指一个时间段中有几个程序都处于已启动运行到运行完毕之间，且这几个程序都是在同一个处理机上运行。**(**同一个时间段内**都是在同一台电脑上完成了**从开始到结束的动作**。)



**并行（Parallel），当系统有一个以上CPU时，当一个CPU执行一个进程时，另一个CPU可以执行另一个进程，两个进程互不抢占CPU资源，可以同时进行，这种方式我们称之为并行(Parallel)。**(这里面有一个很重要的点，那就是系统要有多个CPU才会出现并行。在有多个CPU的情况下，才会出现真正意义上的『同时进行』。)

### 20 线程的死锁和避免死锁的方式？

https://blog.csdn.net/ls5718/article/details/51896159

死锁就是多个线程同时彼此循环等待，都等着另一方释放其占有的资源给自己用

**发生死锁的具体原因如下:**

1. 因为系统资源不足。
2. 进程运行推进的顺序不合适。    
3. 资源分配不当。
4.  死锁产生的必要条件
   产生死锁必须同时满足以下四个条件，只要其中任一条件不成立，死锁就不会发生。
   1. 互斥条件：进程要求对所分配的资源（如打印机）进行排他性控制，即在一段时间内某 资源仅为一个进程所占有。此时若有其他进程请求该资源，则请求进程只能等待。
   2. 不剥夺条件：进程所获得的资源在未使用完毕之前，不能被其他进程强行夺走，即只能 由获得该资源的进程自己来释放（只能是主动释放)。
   3. 请求和保持条件：进程已经保持了至少一个资源，但又提出了新的资源请求，而该资源 已被其他进程占有，此时请求进程被阻塞，但对自己已获得的资源保持不放。
   4. 循环等待条件：存在一种进程资源的循环等待链，链中每一个进程已获得的资源同时被 链中下一个进程所请求。即存在一个处于等待状态的进程集合{Pl, P2, ..., pn}，其中Pi等 待的资源被P(i+1)占有（i=0, 1, ..., n-1)，Pn等待的资源被P0占有

**如何避免死锁**

在有些情况下死锁是可以避免的。三种用于避免死锁的技术：

1. 加锁顺序（线程按照一定的顺序加锁）

2. 加锁时限（线程尝试获取锁的时候加上一定的时限，超过时限则放弃对该锁的请求，并释放自己占有的锁）

3. 死锁检测 ：死锁检测是**一个更好的死锁预防机制**，它主要是针对那些不可能实现按序加锁并且锁超时也不可行的场景。

   每当一个线程获得了锁，会在**线程和锁相关的数据结构中**（map、graph等等）将其记下。除此之外，**每当有线程请求锁**，也需要**记录在这个数据结构中**。

   当一个**线程请求锁失败**时，这个线程可以遍历锁的关系图看看是否有死锁发生。例如，线程A请求锁7，但是锁7这个时候被线程B持有，这时线程A就可以检查一下线程B是否已经请求了线程A当前所持有的锁。如果线程B确实有这样的请求，那么就是发生了死锁。
   
   使用 jps + jstack
   
   - jps -l
   - jstack -l 12316

![image-20210727230509040](D:\1书本笔记\java实战项目\image-20210727230509040.png)

### 21 线程中锁池和等待池的认识？

**锁池和等待池**
在java中，每个对象都有两个池，**锁(monitor)池和等待池**

1. 锁池:假设线程A已经拥有了某个对象(注意:不是类)的锁，而其它的线程想要调用这个对象的某个synchronized方法(或者synchronized块)，由于这些线程在进入对象的synchronized方法之前必须先获得该对象的锁的拥有权，但是该对象的锁目前正被线程A拥有，所以这些线程就进入了该对象的锁池中。
2. 等待池:假设一个线程A调用了某个对象的wait()方法，线程A就会释放该对象的锁后，进入到了该对象的等待池中



### 22 notify和notifyAll的区别

* 如果线程调用了**对象的 wait()方法**，那么**线程便会处于该对象的等待池中**，等待池中的线程不会去竞争该对象的锁。
* 当有线程调用了对象的 notifyAll()方法（唤醒所有 wait 线程）或 notify()方法（只随机唤醒一个 wait 线程），被唤醒的的线程便会进入该对象的锁池中，锁池中的线程会去竞争该对象锁。也就是说，调用了notify后只要一个线程会由等待池进入锁池，而notifyAll会将该对象等待池内的所有线程移动到锁池中，等待锁竞争
* 优先级高的线程竞争到对象锁的概率大，假若某线程没有竞争到该对象锁，它还会留在锁池中，唯有线程再次调用 wait()方法，它才会重新回到等待池中。而竞争到对象锁的线程则继续往下执行，直到执行完了 synchronized 代码块，它会释放掉该对象锁，这时锁池中的线程会继续竞争该对象锁。
  wait() ,notifyAll(),notify() 三个方法都是Object类中的方法. 

关于wait() ,notifyAll(),notify() 三个方法的理解

1. wait() 
   public final void wait() throws InterruptedException,IllegalMonitorStateException
   该方法用来将当前线程置入休眠状态，直到接到通知或被中断为止。在调用 wait()之前，线程必须要获得该对象的对象级别锁，即只能在同步方法或同步块中调用 wait()方法。进入 wait()方法后，当前线程释放锁。在从 wait()返回前，线程与其他线程竞争重新获得锁。如果调用 wait()时，没有持有适当的锁，则抛出 IllegalMonitorStateException，它是 RuntimeException 的一个子类，因此，不需要 try-catch 结构。

2. notify() 
   public final native void notify() throws IllegalMonitorStateException
   该方法也要在同步方法或同步块中调用，即在调用前，线程也必须要获得该对象的对象级别锁，的如果调用 notify()时没有持有适当的锁，也会抛出 IllegalMonitorStateException。

   该方法用来通知那些可能等待该对象的对象锁的其他线程。如果有多个线程等待，则线程规划器任意挑选出其中一个 wait()状态的线程来发出通知，并使它等待获取该对象的对象锁（notify 后，**当前线程不会马上释放该对象锁，wait 所在的线程并不能马上获取该对象锁**，要等到程序退出 synchronized 代码块后，**当前线程才会释放锁**，**wait所在的线程也才可以获取该对象锁**），但不惊动其他同样在等待被该对象notify的线程们。当第一个获得了该对象锁的 wait 线程运行完毕以后，它会释放掉该对象锁，此时如果该对象没有再次使用 notify 语句，则即便该对象已经空闲，其他 wait 状态等待的线程由于没有得到该对象的通知，会继续阻塞在 wait 状态，直到这个对象发出一个 notify 或 notifyAll。这里需要注意：它们等待的是被 notify 或 notifyAll，而不是锁。这与下面的 notifyAll()方法执行后的情况不同。

3. notifyAll() 
   public final native void notifyAll() throws IllegalMonitorStateException
   该方法与 notify ()方法的工作方式相同，**重要的一点差异是**：

   notifyAll **使所有原来在该对象上 wait 的线程统统退出 wait 的状态**（即全部被唤醒，不再等待 notify 或 notifyAll，但由于此时还没有获取到该对象锁，因此还不能继续往下执行），**变成等待获取该对象上的锁，一旦该对象锁被释放**（notifyAll 线程退出调用了 notifyAll 的 synchronized 代码块的时候），他们就会去竞争。如果其中一个线程获得了该对象锁，它就会继续往下执行，在它退出 synchronized 代码块，释放锁后，**其他的已经被唤醒的线程将会继续竞争获取该锁，一直进行下去，直到所有被唤醒的线程都执行完毕**。

### 23 阻塞和挂起的区别

线程与进程的阻塞

      线程在运行的过程中因为某些原因而发生阻塞，阻塞状态的线程的特点是：该线程放弃CPU的使用，暂停运行，只有等到导致阻塞的原因消除之后才回复运行。或者是被其他的线程中断，该线程也会退出阻塞状态，同时抛出InterruptedException。
    
       正在执行的进程由于发生某时间（如I/O请求、申请缓冲区失败等）暂时无法继续执行。此时引起进程调度，OS把处理机分配给另一个就绪进程，而让受阻进程处于暂停状态，一般将这种状态称为阻塞状态。

进程的挂起

        挂起进程在操作系统中可以定义为暂时被淘汰出内存的进程，机器的资源是有限的，在资源不足的情况下，操作系统对在内存中的程序进行合理的安排，其中有的进程被暂时调离出内存，当条件允许的时候，会被操作系统再次调回内存，重新进入等待被执行的状态即就绪态，系统在超过一定的时间没有任何动作.

共同点： 
           1. 进程都暂停执行 
                        2. 进程都释放CPU，即两个过程都会涉及上下文切换

不同点： 
              1. 对系统资源占用不同：虽然都释放了CPU，但阻塞的进程仍处于内存中，而**挂起的进程通过“对换”技术被换出到外存（磁盘）中**。 
                            2. 发生的时机不同：阻塞一般在进程等待资源（IO资源、信号量等）时发生；而挂起是由于用户和系统的需要，例如，终端用户需要暂停程序研究其执行情况或对其进行修改、OS为了提高内存利用率需要将暂时不能运行的进程（处于就绪或阻塞队列的进程）调出到磁盘 
                            3. 恢复时机不同：阻塞要在等待的资源得到满足（例如获得了锁）后，才会进入就绪状态，等待被调度而执行；被挂起的进程由将其挂起的对象（如用户、系统）在时机符合时（调试结束、被调度进程选中需要重新执行）将其主动激活

### 24 线程池

#### 1 什么是线程池

​		线程池的基本思想是一种对象池，在程序启动时就开辟一块内存空间，里面存放了众多(未死亡)的线程，池中线程执行调度由池管理器来处理。当有线程任务时，从池中取一个，执行完成后线程对象归池，这样可以避免反复创建线程对象所带来的性能开销，节省了系统的资源。

#### 2 使用线程池的好处

1. 减少了创建和销毁线程的次数，每个工作线程都可以被重复利用，可执行多个任务。

2. 运用线程池能有效的控制线程最大并发数，可以根据系统的承受能力，调整线程池中工作线线程的数目，防止因为消耗过多的内存，而把服务器累趴下(每个线程需要大约1MB内存，线程开的越多，消耗的内存也就越大，最后死机)。

3. 对线程进行一些简单的管理，比如：延时执行、定时循环执行的策略等，运用线程池都能进行很好的实现

#### 3、线程池的主要组件

![image-20210728103001353](D:\1书本笔记\java实战项目\image-20210728103001353.png)

一个线程池包括以下四个基本组成部分：

1. 线程池管理器（ThreadPool）：用于创建并管理线程池，包括 创建线程池，销毁线程池，添加新任务；
2. 工作线程（WorkThread）：线程池中线程，在没有任务时处于等待状态，可以循环的执行任务；
3. 任务接口（Task）：每个任务必须实现的接口，以供工作线程调度任务的执行，它主要规定了任务的入口，任务执行完后的收尾工作，任务的执行状态等；
4. 任务队列（taskQueue）：用于存放没有处理的任务。提供一种缓冲机制。



#### 4. 创建一个线程池需要输入几个参数

- **corePoolSize（线程池的基本大小）**：当提交一个任务到线程池时，线程池会创建一个线程来执行任务，即使其他空闲的基本线程能够执行新任务也会创建线程，等到需要执行的任务数大于线程池基本大小时就不再创建。如果调用了线程池的prestartAllCoreThreads方法，线程池会提前创建并启动所有基本线程。
- **maximumPoolSize（线程池最大大小）**：线程池允许创建的最大线程数。如果队列满了，并且已创建的线程数小于最大线程数，则线程池会再创建新的线程执行任务。值得注意的是如果使用了无界的任务队列这个参数就没什么效果。
- **runnableTaskQueue（任务队列）**：用于保存等待执行的任务的阻塞队列。
- **ThreadFactory**：用于设置创建线程的工厂，可以通过线程工厂给每个创建出来的线程设置更有意义的名字，Debug和定位问题时非常又帮助。
- **RejectedExecutionHandler（拒绝策略）**：当队列和线程池都满了，说明线程池处于饱和状态，那么必须采取一种策略处理提交的新任务。这个策略默认情况下是AbortPolicy，表示无法处理新任务时抛出异常。以下是JDK1.5提供的四种策略。n  AbortPolicy：直接抛出异常。
- **keepAliveTime（线程活动保持时间）**：线程池的工作线程空闲后，保持存活的时间。所以如果任务很多，并且每个任务执行的时间比较短，可以调大这个时间，提高线程的利用率。
- **TimeUnit（线程活动保持时间的单位）**：可选的单位有天（DAYS），小时（HOURS），分钟（MINUTES），毫秒(MILLISECONDS)，微秒(MICROSECONDS, 千分之一毫秒)和毫微秒(NANOSECONDS, 千分之一微秒)。

#### 5 execute()与submit()的区别？

1. execute()方法没有返回值，所以无法判断任务知否被线程池执行成功
2. submit()方法返回一个future,那么我们可以通过这个future来判断任务是否执行成功，通过future的get方法来获取返回值

#### 6 shutdown()或shutdownNow()的区别

**shutdown只是将线程池的状态设置为SHUTWDOWN状态，正在执行的任务会继续执行下去，没有被执行的则中断。**

**而shutdownNow则是将线程池的状态设置为STOP，试图停止正在执行的任务则被停止，没被执行任务的则返回。**

**`shutdown()`**

  当线程池调用该方法时,线程池的状态则立刻变成SHUTDOWN状态。此时，则不能再往线程池中添加任何任务，否则将会抛出RejectedExecutionException异常。但是，此时线程池不会立刻退出，直到添加到线程池中的任务都已经处理完成，才会退出。 

将线程池状态置为`SHUTDOWN`,**并不会立即停止**：

- 停止接收外部submit的任务
- **内部正在跑的任务和队列里等待的任务，会执行完**
- 等到第二步完成后，才真正停止

**`shutdownNow()`**

​			执行该方法，线程池的状态立刻变成STOP状态，并试图停止所有正在执行的线程，不再处理还在池队列中等待的任务，当然，它会返回那些未执行的任务。 
​		   它试图终止线程的方法是通过调用Thread.interrupt()方法来实现的，但是大家知道，这种方法的作用有限，如果线程中没有sleep 、wait、Condition、定时锁等应用, interrupt()方法是无法中断当前的线程的。所以，ShutdownNow()并不代表线程池就一定立即就能退出，它可能必须要等待所有正在执行的任务都执行完成了才能退出。 

将线程池状态置为`STOP`。**企图**立即停止，事实上不一定：

- 跟shutdown()一样，先停止接收外部提交的任务
- **忽略队列里等待的任务**
- 尝试将正在跑的任务`interrupt`中断
- 返回未执行的任务列表

它试图终止线程的方法是通过调用Thread.interrupt()方法来实现的，但是大家知道，这种方法的作用有限，如果线程中没有sleep 、wait、Condition、定时锁等应用, interrupt()方法是**无法中断当前的线程的**。所以，**ShutdownNow()并不代表线程池就一定立即就能退出，它也可能必须要等待所有正在执行的任务都执行完成了才能退出**。

但是大多数时候是能立即退出的


#### 7  三种阻塞队列的区别？

ArrayBlockingQueue ， LinkedBlockingQueue ，SynchronousQueue

##### 1 ArrayBlockingQueue 

​		ArrayBlockingQueue是一个有界缓存等待队列，可以指定缓存队列的大小，当正在执行的线程数等于corePoolSize时，多余的元素缓存在ArrayBlockingQueue队列中等待有空闲的线程时继续执行，当ArrayBlockingQueue已满时，加入ArrayBlockingQueue失败，会开启新的线程去执行，当线程数已经达到最大的maximumPoolSizes时，再有新的元素尝试加入ArrayBlockingQueue时会执行拒绝策略

##### 2 LinkedBlockingQueue 

​		LinkedBlockingQueue是一个无界缓存等待队列。当前执行的线程数量达到corePoolSize的数量时，剩余的元素会在阻塞队列里等待。（所以在使用此阻塞队列时maximumPoolSizes就相当于无效了），每个线程完全独立于其他线程。生产者和消费者使用独立的锁来控制数据的同步，即在高并发的情况下可以并行操作队列中的数据。

注：这个队列需要注意的是，虽然通常称其为一个无界队列，但是可以人为指定队列大小，而且由于其用于记录队列大小的参数是int类型字段，所以通常意义上的无界其实就是队列长度为 Integer.MAX_VALUE，且在不指定队列大小的情况下也会默认队列大小为 Integer.MAX_VALUE，等同于如下：

##### 3 SynchronousQueue

​		SynchronousQueue没有容量，是无缓冲等待队列，是一个不存储元素的阻塞队列，会直接将任务交给消费者，必须等队列中的添加元素被消费后才能继续添加新的元素。

拥有公平（FIFO）和非公平(LIFO)策略，非公平侧罗会导致一些数据永远无法被消费的情况？

使用SynchronousQueue阻塞队列一般要求maximumPoolSizes为无界(Integer.MAX_VALUE)，避免线程拒绝执行操作。


##### 4  添加新线程的执行步骤？

![image-20210728150011223](D:\1书本笔记\java实战项目\image-20210728150011223.png)

1. 线程数量未达到corePoolSize，则新建一个线程(核心线程)执行任务

2. 线程数量达到了corePools，则将任务移入队列等待

3. 队列已满，新建线程(非核心线程)执行任务

4. 队列已满，总线程数又达到了maximumPoolSize，就会由(RejectedExecutionHandler)抛出异常

   **新建线程  ->  达到核心数 -> 加入队列 -> 新建线程（非核心） -> 达到最大数 -> 触发拒绝策略**

#### 8 四种拒绝策略？



1. AbortPolicy：丢弃任务并抛出RejectedExecutionException异常

```
 public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
            throw new RejectedExecutionException("Task " + r.toString() +
                                                 " rejected from " +
                                                 e.toString());
        }
复制代码
```

1. DiscardPolicy：丢弃任务，但是不抛出异常。

```
  public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        }
复制代码
```

1. DisCardOldSetPolicy：丢弃队列最前面的任务，然后提交新来的任务

```
public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
            if (!e.isShutdown()) {
                e.getQueue().poll();
                e.execute(r);
            }
        }
复制代码
```

1. CallerRunPolicy：由调用线程（提交任务的线程，主线程）处理该任务

```
 public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
            if (!e.isShutdown()) {
                r.run();
            }
        }
```



#### 9  Java通过Excutors提供四种线程池

> 1. CachedThreadPool()：可缓存线程池。

- 线程数无限制
- 有空闲线程则复用空闲线程，若无空闲线程则新建线程 一定程序减少频繁创建/销毁线程，减少系统开销

> 2. FixedThreadPool()：定长线程池。

- 可控制线程最大并发数（同时执行的线程数）
- 超出的线程会在队列中等待

> 3. ScheduledThreadPool()：

- 定时线程池。
- 支持定时及周期性任务执行。

> 4. SingleThreadExecutor()：单线程化的线程池。

- 有且仅有一个工作线程执行任务
- 所有任务按照指定顺序执行，即遵循队列的入队出队规则



##### 1. newCachedThreadPool

newCachedThreadPool创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程

##### 2. newFixedThreadPool

newFixedThreadPool创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待，指定线程池中的线程数量和最大线程数量一样，也就线程数量固定不变

##### 3. newscheduledThreadPool

newscheduledThreadPool创建一个定长线程池，支持定时及周期性任务执行。延迟执行示例代码如下.表示延迟1秒后每3秒执行一次

##### 4. newSingleThreadExecutor

newSingleThreadExecutor创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行

#### 6、线程池参数设置

参数的设置跟系统的负载有直接的关系，下面为系统负载的相关参数：

- tasks，每秒需要处理的的任务数（针对系统需求）
- threadtasks，每个线程每钞可处理任务数（针对线程本身）
- responsetime，系统允许任务最大的响应时间，比如每个任务的响应时间不得超过2秒。

##### corePoolSize

系统每秒有tasks个任务需要处理理，则每个线程每钞可处理threadtasks个任务。，则需要的线程数为：tasks/threadtasks，即tasks/threadtasks个线程数。

假设系统每秒任务数为100 ~ 1000，每个线程每钞可处理10个任务，则需要100 / 10至1000 / 10，即10 ~ 100个线程。那么corePoolSize应该设置为大于10，具体数字最好根据8020原则，因为系统每秒任务数为100 ~ 1000，即80%情况下系统每秒任务数小于1000 * 20% = 200，则corePoolSize可设置为200 / 10 = 20。

##### queueCapacity

任务队列的长度要根据核心线程数，以及系统对任务响应时间的要求有关。队列长度可以设置为 **所有核心线程每秒处理任务数 * 每个任务响应时间 = 每秒任务总响应时间** ，即(corePoolSize*threadtasks)*responsetime： (20x10)x2=400，即队列长度可设置为400。

##### maxPoolSize

当系统负载达到最大值时，核心线程数已无法按时处理完所有任务，这时就需要增加线程。每秒200个任务需要20个线程，那么当每秒达到1000个任务时，则需要（tasks - queueCapacity）/ threadtasks 即(1000-400)/10，即60个线程，可将maxPoolSize设置为60。

**队列长度设置过大，会导致任务响应时间过长**，切忌以下写法：

```
LinkedBlockingQueue queue = new LinkedBlockingQueue();
复制代码
```

这实际上是将队列长度设置为Integer.MAX_VALUE，将会导致线程数量永远为corePoolSize，再也不会增加，当任务数量陡增时，任务响应时间也将随之陡增。

##### keepAliveTime

当负载降低时，可减少线程数量，当线程的空闲时间超过keepAliveTime，会自动释放线程资源。默认情况下线程池停止多余的线程并最少会保持corePoolSize个线程。

##### allowCoreThreadTimeout

默认情况下核心线程不会退出，可通过将该参数设置为true，让核心线程也退出。

一般说来，大家认为线程池的大小经验值应该这样设置：（其中N为CPU的个数）

- 如果是CPU密集型应用，则线程池大小设置为N+1
- 如果是IO密集型应用，则线程池大小设置为2N+1

至于这里为什么是N+1和2N+1需要看下这个博客。https://zhuanlan.zhihu.com/p/116426107

**关于核心线程数的设置**

所以为了正确设置线程池的大小，我们应该计算出 我们的计算型任务（用 C表示）占总任务的比例（总任务个数=计算型任务个数+阻塞型任务个数（用W表示））
 假设当前 CPU 的核数 为 N = 4。
 rate = C/(C+W)

这样 CORE_SIZE = N/rate。

如 当前有8个计算型任务，2个阻塞型任务，比例就是 0.8，也就是4/5，

计算可得 CORE_SIZE = 5 时，能最大程度的利用 CPU。







#### 7、线程池的五种状态



![image-20210825141330257](D:\1书本笔记\java实战项目\image-20210825141330257.png)

1. 线程池的初始化状态是RUNNING，能够接收新任务，以及对已添加的任务进行处理。
2. 线程池处在SHUTDOWN状态时，不接收新任务，但能处理已添加的任务。  调用线程池的shutdown()接口时，线程池由RUNNING -> SHUTDOWN。
3. 线程池处在STOP状态时，不接收新任务，不处理已添加的任务，并且会中断正在处理的任务。 调用线程池的shutdownNow()接口时，线程池由(RUNNING or SHUTDOWN ) -> STOP。
4. 当所有的任务已终止，ctl记录的”任务数量”为0，线程池会变为TIDYING状态。当线程池变为TIDYING状态时，会执行钩子函数terminated()。terminated()在ThreadPoolExecutor类中是空的，若用户想在线程池变为TIDYING时，进行相应的处理；可以通过重载terminated()函数来实现。
5. 当线程池在SHUTDOWN状态下，阻塞队列为空并且线程池中执行的任务也为空时，就会由 SHUTDOWN -> TIDYING。
6. 当线程池在STOP状态下，线程池中执行的任务为空时，就会由STOP -> TIDYING。 线程池彻底终止，就变成TERMINATED状态。线程池处在TIDYING状态时，执行完terminated()之后，就会由 TIDYING -> TERMINATED。

#### 8、关闭线程池

线程池提供两种关闭线程池方法：shutDown()和shutdownNow()

**shutDown()**

当线程池调用该方法时,线程池的状态则立刻变成SHUTDOWN状态。此时，则不能再往线程池中添加任何任务，否则将会抛出RejectedExecutionException异常。但是，此时线程池不会立刻退出，直到添加到线程池中的任务都已经处理完成，才会退出。

**shutdownNow()**

根据JDK文档描述，大致意思是：执行该方法，线程池的状态立刻变成STOP状态，并试图停止所有正在执行的线程，不再处理还在池队列中等待的任务，当然，它会返回那些未执行的任务。

它试图终止线程的方法是通过调用Thread.interrupt()方法来实现的，但是大家知道，这种方法的作用有限，如果线程中没有sleep、wait、Condition、定时锁等应用, interrupt()方法是无法中断当前的线程的。所以，ShutdownNow()并不代表线程池就一定立即就能退出，它可能必须要等待所有正在执行的任务都执行完成了才能退出。

#### 9、各种场景下怎么设置线程数

##### 1、高并发、任务执行时间短的业务怎样使用线程池？

线程池线程数可以设置为CPU核数+1，减少线程上下文的切换

##### 2、并发不高、任务执行时间长的业务怎样使用线程池？

这个需要判断执行时间是耗在哪个地方

- **假如是业务时间长集中在IO操作上，也就是IO密集型的任务**，因为IO操作并不占用CPU，所以不要让所有的CPU闲下来，可以适当加大线程池中的线程数目（2 * CPU核数），让CPU处理更多的业务。（2*N）
- **假如是业务时间长集中在计算操作上，也就是CPU密集型任务，和（1）CPU核数+1 一样吧**，线程池中的线程数设置得少一些，减少线程上下文的切换 ( N+1)

##### 3、并发高、业务执行时间长的业务怎样使用线程池？

解决这种类型任务的关键不在于线程池而在于整体架构的设计

#### 10 、为什么不推荐使用JUC的线程池？

![image-20210728164457382](D:\1书本笔记\java实战项目\image-20210728164457382.png)

阿里发布的 Java开发手册中强制线程池不允许使用 Executors 去创建，而是通过 ThreadPoolExecutor 的方式。这是为什么？
有4个原因，前2个是主要原因。具体如下：

一、缓存队列 LinkedBlockingQueue 没有设置固定容量大小
1.1、Executors.newFixedThreadPool()
创建固定大小的线程池

public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}
ThreadPoolExecutor 部分参数：

corePoolSize ：线程池中核心线程数的最大值。此处为 nThreads个。
maximumPoolSize ：线程池中能拥有最多线程数 。此处为 nThreads 个。
**LinkedBlockingQueue 用于缓存任务的阻塞队列 。 此处没有设置容量大小**，默认是 Integer.MAX_VALUE，可以认为是无界的。
问题分析：
从源码中可以看出， 虽然表面上 newFixedThreadPool() 中定义了 核心线程数 和 最大线程数 都是固定 nThreads 个，但是当 线程数量超过 nThreads 时，多余的线程会保存到 LinkedBlockingQueue 中，而 LinkedBlockingQueue 没是无界的，导致其无限增大，最终内存撑爆。

1.2、Executors.newSingleThreadExecutor()
创建单个线程池 ，线程池中只有一个线程。

public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
       					 (new ThreadPoolExecutor(1, 1,
                              				 0L, TimeUnit.MILLISECONDS,
                              				  new LinkedBlockingQueue<Runnable>()));
}
创建单个线程池 ，线程池中只有一个线程。

优点： 创建一个单线程的线程池，保证线程的顺序执行 ；
缺点： 与 newFixedThreadPool() 相同。

总结：
**newFixedThreadPool()、newSingleThreadExecutor() 底层代码 中 LinkedBlockingQueue 没有设置容量大小，默认是 Integer.MAX_VALUE， 可以认为是无界的。线程池中 多余的线程会被缓存到 LinkedBlockingQueue中，最终内存撑爆**。

二 、最大线程数量是 Integer.MAX_VALUE
2.1、Executors.newCachedThreadPool()
缓存线程池，线程池的数量不固定，可以根据需求自动的更改数量

public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
ThreadPoolExecutor 部分参数：

corePoolSize ：线程池中核心线程数的最大值。此处为 0 个。
maximumPoolSize ：线程池中能拥有最多线程数 。此处为 Integer.MAX_VALUE 。可以认为是无限大 。
优点： 很灵活，弹性的线程池线程管理，用多少线程给多大的线程池，不用后及时回收，用则新建 ；
缺点： 从源码中可以看出，**SynchronousQueue() 只能存一个队列**，可以认为所有 放到 newCachedThreadPool() 中的线程，不会缓存到队列中，而是直接运行的， **由于最大线程数是 Integer.MAX_VALUE ，这个数量级可以认为是无限大了， 随着执行线程数量的增多 和 线程没有及时结束，最终会将内存撑爆**。

2.2、Executors.newScheduledThreadPool()
创建固定大小的线程，可以延迟或定时的执行任务

public static ScheduledExecutorService newScheduledThreadPool(
        int corePoolSize, ThreadFactory threadFactory) {
    return new ScheduledThreadPoolExecutor(corePoolSize, threadFactory);
}

// ScheduledThreadPoolExecutor 类的源码：
public ScheduledThreadPoolExecutor(int corePoolSize,
                                   ThreadFactory threadFactory) {
    super(corePoolSize, Integer.MAX_VALUE, 0, TimeUnit.NANOSECONDS,
          new DelayedWorkQueue(), threadFactory);
}
优点： 创建一个固定大小线程池，可以定时或周期性的执行任务 ；
缺点： 与 newCachedThreadPool() 相同。

总结：
**newCachedThreadPool()、newScheduledThreadPool() 的 底层代码 中 的 最大线程数（maximumPoolSize） 是 Integer.MAX_VALUE，可以认为是无限大，如果线程池中，执行中的线程没有及时结束，并且不断地有线程加入并执行，最终会将内存撑爆。**

#### 11 问题？

（这部分估计需要看源码。。）

##### 1、非核心线程延迟死亡，如何实现？

通过阻塞队列poll()，让线程阻塞等待一段时间，如果没有取到任务，则线程死亡

##### 2、线程池为什么能维持线程不释放，随时运行各种任务？

```
for (;;) {
          
            try {
                Runnable r = timed ?
                    workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
                    workQueue.take();
                if (r != null)
                    return r;
                timedOut = true;
            } catch (InterruptedException retry) {
                timedOut = false;
            }
        }
    }
复制代码
```

在死循环中工作队列workQueue会一直去拿任务:

- 核心线程的会一直卡在 workQueue.take()方法，让线程一直等待，直到获取到任务，然后返回。
- 非核心线程会 workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) ，如果超时还没有拿到，下一次循环判断compareAndDecrementWorkerCount就会返回null,Worker对象的run()方法循环体的判断为null,任务结束，然后线程被系统回收。

通过阻塞队列take()，让线程一直等待，直到获取到任务

##### 3、如何释放核心线程？

将allowCoreThreadTimeOut设置为true。可用下面代码实验

```
{
    // 允许释放核心线程，等待时间为100毫秒
    es.allowCoreThreadTimeOut(true);
    for(......){
        // 向线程池里添加任务，任务内容为打印当前线程池线程数
        Thread.currentThread().sleep(200);
    }
}
复制代码
```

线程数会一直为1。 如果allowCoreThreadTimeOut为false，线程数会逐渐达到饱和，然后大家一起阻塞等待。

##### 4、非核心线程能成为核心线程吗？

线程池不区分核心线程于非核心线程，只是根据当前线程池容量状态做不同的处理来进行调整，因此看起来像是有核心线程于非核心线程，实际上是满足线程池期望达到的并发状态。

##### 5、Runnable在线程池里如何执行？

线程执行Worker，Worker不断从阻塞队列里获取任务来执行。





### 10 对ThreadLocal的认识？

https://zhuanlan.zhihu.com/p/102571059

#### **1、ThreadLocal是什么**

​		从名字我们就可以看到ThreadLocal叫做线程变量，意思是ThreadLocal中填充的变量属于**当前**线程，该变量**对其他线程而言是隔离**的。ThreadLocal为变量在每个线程中都创建了一个副本，那么每个线程可以访问自己内部的副本变量。

从字面意思来看非常容易理解，但是从实际使用的角度来看，就没那么容易了，作为一个面试常问的点，使用场景那也是相当的丰富：

**1、在进行对象跨层传递的时候，使用ThreadLocal可以避免多次传递，打破层次间的约束。**

**2、线程间数据隔离**

**3、进行事务操作，用于存储线程事务信息。**

**4、数据库连接，Session会话管理。**



#### **2、ThreadLocal怎么用**

既然ThreadLocal的作用是每一个线程创建一个副本，我们使用一个例子来验证一下：

![img](https://pics0.baidu.com/feed/14ce36d3d539b600ff663d8e75a8c62fc75cb759.jpeg?token=44530368d6f896c24c1566224aa81a47&s=B8C3A144D2B4806F165DF8030000E0C1)

从结果我们可以看到，每一个线程都有各自的local值，我们设置了一个休眠时间，就是为了另外一个线程也能够及时的读取当前的local值。

这就是TheadLocal的基本使用，是不是非常的简单。那么为什么会在数据库连接的时候使用的比较多呢？

![img](https://pics6.baidu.com/feed/3c6d55fbb2fb43165898204f805cb52608f7d37a.jpeg?token=8af7124abdc7ed9108b00bba0a1b48fd&s=B8C1B34C43B4BD6C1E499C0E0200E081)

上面是一个数据库连接的管理类，我们使用数据库的时候首先就是建立数据库连接，然后用完了之后关闭就好了，这样做有一个很严重的问题，如果有1个客户端频繁的使用数据库，那么就需要建立多次链接和关闭，我们的服务器可能会吃不消，怎么办呢？如果有一万个客户端，那么服务器压力更大。

这时候最好ThreadLocal，因为ThreadLocal在每个线程中对连接会创建一个副本，且在线程内部任何地方都可以使用，线程之间互不影响，这样一来就不存在线程安全问题，也不会严重影响程序执行性能。是不是很好用。

以上主要是讲解了一个基本的案例，然后还分析了为什么在数据库连接的时候会使用ThreadLocal。下面我们从源码的角度来分析一下，ThreadLocal的工作原理。





#### **3、ThreadLocal源码分析**

OK，其实内部源码很简单，现在我们总结一波

（1）每个Thread维护着一个ThreadLocalMap的引用

（2）ThreadLocalMap是ThreadLocal的内部类，用Entry来进行存储

（3）ThreadLocal创建的副本是存储在自己的threadLocals中的，也就是自己的ThreadLocalMap。

（4）ThreadLocalMap的键值为ThreadLocal对象，而且可以有多个threadLocal变量，因此保存在map中

（5）在进行get之前，必须先set，否则会报空指针异常，当然也可以初始化一个，但是必须重写initialValue()方法。

（6）ThreadLocal本身并不存储值，它只是作为一个key来让线程从ThreadLocalMap获取value。OK，现在从源码的角度上不知道你能理解不，对于ThreadLocal来说关键就是内部的ThreadLocalMap。





#### **4、ThreadLocal内存泄漏问题**

前提的几个概念：

1 **内存泄露**

​	内存泄露为程序在申请内存后，无法释放已申请的内存空间，一次内存泄露危害可以忽略，但内存泄露堆积后果很严重，无论多少内存,迟早会被占光，

​	广义并通俗的说，就是：不再会被使用的对象或者变量占用的内存不能被回收，就是内存泄露。

2 **强引用与弱引用**

​		**强引用**   使用最普遍的引用，一个对象具有强引用，不会被垃圾回收器回收。当内存空间不足，Java虚拟机宁愿抛出OutOfMemoryError错误，使程序异常终止，也不回收这种对象。

**如果想取消强引用和某个对象之间的关联，可以显式地将引用赋值为null，这样可以使JVM在合适的时间就会回收该对象。**

​		**弱引用**，JVM进行垃圾回收时，无论内存是否充足，都会回收被弱引用关联的对象。在java中，用java.lang.ref.WeakReference类来表示。可以在缓存中使用弱引用。

![image-20210729211013189](D:\1书本笔记\java实战项目\image-20210729211013189.png)

**那为什么使用弱引用而不是强引用？？**

我们看看Key使用的

**key 使用强引用**

当hreadLocalMap的key为强引用回收ThreadLocal时，因为ThreadLocalMap还持有ThreadLocal的强引用，如果没有手动删除，ThreadLocal不会被回收，导致Entry内存泄漏。

**key 使用弱引用**

当ThreadLocalMap的key为弱引用回收ThreadLocal时，由于ThreadLocalMap持有ThreadLocal的弱引用，即使没有手动删除，ThreadLocal也会被回收。当key为null，在下一次ThreadLocalMap调用set(),get()，remove()方法的时候会被清除value值。



由于Thread中包含变量ThreadLocalMap，因此ThreadLocalMap与Thread的生命周期是一样长，如果都没有手动删除对应key，都会导致内存泄漏。

​		但是使用**弱引用**可以多一层保障：弱引用ThreadLocal不会内存泄漏，对应的value在下一次ThreadLocalMap调用set(),get(),remove()的时候会被清除。

因此，ThreadLocal内存泄漏的根源是：**由于ThreadLocalMap的生命周期跟Thread一样长，如果没有手动删除对应key就会导致内存泄漏，而不是因为弱引用**。

3 **ThreadLocal正确的使用方法**

- 每次使用完ThreadLocal都调用它的remove()方法清除数据
- 将ThreadLocal变量定义成private static，这样就一直存在ThreadLocal的强引用，也就能保证任何时候都能通过ThreadLocal的弱引用访问到Entry的value值，进而清除掉 。

#### 5  ThreadLocalMap的认识？





#### 6 Thread、ThreadLocal 以及ThreadLocalMap的关联性？

![image-20210729213535577](D:\1书本笔记\java实战项目\image-20210729213535577.png)

![image-20210729213331664](D:\1书本笔记\java实战项目\image-20210729213331664.png)

​		`ThreadLocal`并不维护`ThreadLocalMap`，并不是一个存储数据的容器，它只是相当于一个工具包，提供了操作该容器的方法，如get、set、remove等。**而`ThreadLocal`内部类`ThreadLocalMap`才是存储数据的容器**，**并且该容器由`Thread`维护**。

**每一个`Thread`对象均含有一个`ThreadLocalMap`类型的成员变量`threadLocals`**，它存储本线程中所有ThreadLocal对象及其对应的值。



#### 7 ThreadLocal的内存泄露以及解决办法？

​		在ThreadLocalMap中，只有key是弱引用，value仍然是一个强引用。当某一条线程中的ThreadLocal使用完毕，没有强引用指向它的时候，这个key指向的对象就会被垃圾收集器回收，从而这个key就变成了null；然而，此时value和value指向的对象之间仍然是强引用关系，只要这种关系不解除，value指向的对象永远不会被垃圾收集器回收，从而导致内存泄漏！

不过不用担心，ThreadLocal提供了这个问题的解决方案。

每次操作set、get、remove操作时，**ThreadLocal都会将key为null的Entry删除，从而避免内存泄漏**。下面的两张图里面的ThreadLocal**方法就中的remove方法，就可以将当前线程的Key给清除**。

那么问题又来了，如果一个线程运行周期较长，而且将一个大对象放入LocalThreadMap后便不再调用set、get、remove方法，此时该仍然可能会导致内存泄漏。

这个问题确实存在，没办法通过ThreadLocal解决，而是需要程序员在完成ThreadLocal的使用后要养成手动调用remove的习惯，从而避免内存泄漏。

因此，**ThreadLocal内存泄漏的根源是：由于ThreadLocalMap的生命周期跟Thread一样长，如果没有手动删除对应key就会导致内存泄漏，而不是因为弱引用**。

**ThreadLocal正确的使用方法**

- 每次使用完ThreadLocal都调用它的remove()方法清除数据
- 将ThreadLocal变量定义成private static，这样就一直存在ThreadLocal的强引用，也就能保证任何时候都能通过ThreadLocal的弱引用访问到Entry的value值，进而清除掉 。



![image-20210729213706547](D:\1书本笔记\java实战项目\image-20210729213706547.png)

![image-20210729213721033](D:\1书本笔记\java实战项目\image-20210729213721033.png)

**ThreadLocal的使用场景**

Web系统Session的存储就是ThreadLocal一个典型的应用场景。

Web容器采用线程隔离的多线程模型，也就是每一个请求都会对应一条线程，线程之间相互隔离，没有共享数据。这样能够简化编程模型，程序员可以用单线程的思维开发这种多线程应用。

当请求到来时，可以将当前Session信息存储在ThreadLocal中，在请求处理过程中可以随时使用Session信息，每个请求之间的Session信息互不影响。当请求处理完成后通过remove方法将当前Session信息清除即可。

## 2 同步

### 1 synchronized锁

#### 1 synchronized锁的作用

**1.1 原子性**

**所谓原子性就是指一个操作或者多个操作，要么全部执行并且执行的过程不会被任何因素打断，要么就都不执行。**

在Java中，对基本数据类型的变量的读取和赋值操作是原子性操作，即这些操作是不可被中断的，要么执行，要么不执行。但是像i++、i+=1等操作字符就不是原子性的，它们是分成**读取、计算、赋值**几步操作，原值在这些步骤还没完成时就可能已经被赋值了，那么最后赋值写入的数据就是脏数据，无法保证原子性。

被synchronized修饰的类或对象的所有操作都是原子的，因为在执行操作之前必须先获得类或对象的锁，直到执行完才能释放，这中间的过程无法被中断（除了已经废弃的stop()方法），即保证了原子性。

**注意！面试时经常会问比较synchronized和volatile，它们俩特性上最大的区别就在于原子性，volatile不具备原子性。**

**1.2 可见性**

**可见性是指多个线程访问一个资源时，该资源的状态、值信息等对于其他线程都是可见的。**

synchronized和volatile都具有可见性，其中synchronized对一个类或对象加锁时，一个线程如果要访问该类或对象必须先获得它的锁，而这个锁的状态对于其他任何线程都是可见的，并且在释放锁之前会将对变量的修改刷新到主存当中，保证资源变量的可见性，如果某个线程占用了该锁，其他线程就必须在锁池中等待锁的释放。

而volatile的实现类似，被volatile修饰的变量，每当值需要修改时都会立即更新主存，主存是共享的，所有线程可见，所以确保了其他线程读取到的变量永远是最新值，保证可见性。

**1.3 有序性**

**有序性值程序执行的顺序按照代码先后执行。**

（有效解决了指令重排序的问题）

synchronized和volatile都具有有序性，Java允许编译器和处理器对指令进行重排，但是指令重排并不会影响单线程的顺序，它影响的是多线程并发执行的顺序性。synchronized保证了每个时刻都只有一个线程访问同步代码块，也就确定了线程执行同步代码块是分先后顺序的，保证了有序性。

**1.4 可重入性**

synchronized和ReentrantLock都是可重入锁。当一个线程试图操作一个由其他线程持有的对象锁的临界资源时，将会处于阻塞状态，但当一个线程再次请求自己持有对象锁的临界资源时，这种情况属于重入锁。通俗一点讲就是说一个线程拥有了锁仍然还可以重复申请锁。

**1.5 不可中断性**

不可中断就是指，一个线程获取锁之后，另外一个线程处于阻塞或者等待状态，前一个不释放，后一个也一直会阻塞或者等待，不可以被中断。

值得一提的是，Lock的tryLock方法是可以被中断的。

#### 2 synchronized锁的基本使用

链接：https://www.cnblogs.com/paddix/p/5367116.html

　　（1）修饰普通方法（**普通方法锁住的是this对象，谁调用该普通方法就锁谁**）

　　（2）修饰静态方法（锁的是类对象，虽然有不同对象调用同一个被锁住的类，**但由于是修饰的静态方法，调用该类的对象都会被锁住！**）

　　（3）修饰代码块（**锁住的是代码块中包含的参数，如果是this就是，谁调用该代码块就锁谁**）

![image-20210803103653835](D:\1书本笔记\java实战项目\image-20210803103653835.png)

![image-20210803103707411](D:\1书本笔记\java实战项目\image-20210803103707411.png)

![image-20210803103724034](D:\1书本笔记\java实战项目\image-20210803103724034.png)

![image-20210803103736230](D:\1书本笔记\java实战项目\image-20210803103736230.png)

![image-20210803103751053](D:\1书本笔记\java实战项目\image-20210803103751053.png)

![image-20210803103804937](D:\1书本笔记\java实战项目\image-20210803103804937.png)



#### 3 synchronized的内部实现以及优化？

前提条件：

每个对象头都有一个Monitor对象

![image-20210803105216505](D:\1书本笔记\java实战项目\image-20210803105216505.png)

1 monitorenter

每个对象有一个监视器锁（monitor）。当monitor被占用时就会处于锁定状态，线程执行monitorenter指令时尝试获取monitor的所有权，过程如下：

1、如果monitor的进入数为0，则该线程进入monitor，然后将进入数设置为1，该线程即为monitor的所有者。

2、如果线程已经占有该monitor，只是重新进入，则进入monitor的进入数加1.

3.如果其他线程已经占用了monitor，则该线程进入阻塞状态，直到 monitor的进入数为0，再重新尝试获取monitor的所有权

2 monitorexit

执行monitorexit的线程必须是objectref所对应的monitor的所有者。

指令执行时，monitor的进入数减1，如果减1后进入数为0，那线程退出monitor，不再是这个monitor的所有者。其他被这个monitor阻塞的线程可以尝试去获取这个 monitor 的所有权。 

##### 1 **Synchronized 原理：**

- 同步代码块是通过 monitorenter 和 monitorexit 指令获取线程的执行权
- 同步方法通过加 ACC_SYNCHRONIZED 标识实现线程的执行权的控制

​		Synchronized的实现原理，**Synchronized的语义底层是通过一个monitor的对象来完成**，其实wait/notify等方法也依赖于monitor对象，这就是为什么只有在同步的块或者方法中才能调用wait/notify等方法，否则会抛出java.lang.IllegalMonitorStateException的异常的原因。

　　有了对Synchronized原理的认识，再来看上面的程序就可以迎刃而解了。

1、代码段2结果：

　　虽然method1和method2是不同的方法，但是这两个方法都进行了同步，并且是通过同一个对象去调用的，所以调用之前都需要先去竞争同一个对象上的锁（monitor），也就只能互斥的获取到锁，因此，method1和method2只能顺序的执行。

2、代码段3结果：

　　虽然test和test2属于不同对象，但是test和test2属于同一个类的不同实例，由于method1和method2都属于静态同步方法，所以调用的时候需要获取同一个类上monitor（每个类只对应一个class对象），所以也只能顺序的执行。

3、代码段4结果：

　　对于代码块的同步实质上需要获取Synchronized关键字后面括号中对象的monitor，由于这段代码中括号的内容都是this，而method1和method2又是通过同一的对象去调用的，所以进入同步块之前需要去竞争同一个对象上的锁，因此只能顺序执行同步

##### 2 优化

就是锁升级的过程，见“锁”章节的细讲！

![image-20210803105256585](D:\1书本笔记\java实战项目\image-20210803105256585.png)

#### 4 synchronized的优点和缺点

**synchronized的缺点 :**

**（1）效率低**

synchronized关键字是不可中断的，这也就意味着一个等待的线程如果不能获取到锁将会一直等待，而不能再去做其他的事了。

对synchronized关键字的一个改进措施，那就是设置超时时间，如果一个线程长时间拿不到锁，就可以去做其他事情了。

**（2）不够灵活**

加锁和解锁的时候，每个锁只能有一个对象处理，这对于目前分布式等思想格格不入。

**（3）无法知道是否成功获取到锁**

锁如果获取到了，我们无法得知。既然无法得知也就很不容易进行改进。

既然synchronized有这么多缺陷。所以才出现了各种各样的锁。

**synchronized的优点 ：**



### 2 java对象头的结构

在HotSpot虚拟机中，对象在内存中存储的布局可以分为3块区域：对象头（Header）、实例数据（Instance Data）和对齐填充（Padding）。下图是普通对象实例与数组对象实例的数据结构：

![image-20210803112554673](D:\1书本笔记\java实战项目\image-20210803112554673.png)

**对象头**
HotSpot虚拟机的对象头包括两部分信息：

1. markword
   第一部分markword,用于存储对象自身的运行时数据，**如哈希码（HashCode）、GC分代年龄、锁状态标志、线程持有的锁、偏向线程ID、偏向时间戳**等，这部分数据的长度在32位和64位的虚拟机（未开启压缩指针）中分别为32bit和64bit，官方称它为“MarkWord”。
2. klass
   对象头的另外一部分是klass类型指针，即对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例.
   数组长度（只有数组对象有）
3. 如果对象是一个数组, 那在对象头中还必须有一块数据用于记录数组长度.

**实例数据**
实例数据部分是对象真正存储的有效信息，也是在程序代码中所定义的各种类型的字段内容。无论是从父类继承下来的，还是在子类中定义的，都需要记录起来。

**对齐填充**
第三部分对齐填充并不是必然存在的，也没有特别的含义，它仅仅起着占位符的作用。由于HotSpot VM的自动内存管理系统要求对象起始地址必须是8字节的整数倍，换句话说，就是对象的大小必须是8字节的整数倍。而对象头部分正好是8字节的倍数（1倍或者2倍），因此，当对象实例数据部分没有对齐时，就需要通过对齐填充来补全。

**对象大小计算**
要点

1. 在32位系统下，存放Class指针的空间大小是4字节，MarkWord是4字节，对象头为8字节。
2. 在64位系统下，存放Class指针的空间大小是8字节，MarkWord是8字节，对象头为16字节。
3. 64位开启指针压缩的情况下，存放Class指针的空间大小是4字节，MarkWord是8字节，对象头为12字节。 数组长度4字节+数组对象头8字节(对象引用4字节（未开启指针压缩的64位为8字节）+数组markword为4字节（64位未开启指针压缩的为8字节）)+对齐4=16字节。
4. 静态属性不算在对象大小内。

### 3 简述对Monitor的认识

链接：https://www.php.cn/java-article-410323.html

​		管程，英文是 Monitor，也常被翻译为“监视器”，monitor 不管是翻译为“管程”还是“监视器”，都是比较晦涩的，通过翻译后的中文，并无法对 monitor 达到一个直观的描述。
在《操作系统同步原语》 这篇文章中，介绍了操作系统在面对 进程/线程 间同步的时候，所支持的一些同步原语，其中 semaphore 信号量 和 mutex 互斥量是最重要的同步原语。
​		在使用基本的 mutex 进行并发控制时，需要程序员非常小心地控制 mutex 的 down 和 up 操作，否则很容易引起死锁等问题。为了更容易地编写出正确的并发程序，所以在 mutex 和 semaphore 的基础上，提出了更高层次的同步原语 monitor，不过需要注意的是，操作系统本身并不支持 monitor 机制，实际上，monitor 是属于编程语言的范畴，当你想要使用 monitor 时，先了解一下语言本身是否支持 monitor 原语，例如 C 语言它就不支持 monitor，Java 语言支持 monitor。
一般的 monitor 实现模式是编程语言在语法上提供语法糖，而如何实现 monitor 机制，则属于编译器的工作，Java 就是这么干的。

​		**monitor 的重要特点是，同一个时刻，只有一个 进程/线程 能进入 monitor 中定义的临界区，这使得 monitor 能够达到互斥的效果。**但仅仅有互斥的作用是不够的，无法进入 monitor 临界区的 进程/线程，它们应该被阻塞，并且在必要的时候会被唤醒。显然，monitor 作为一个同步工具，也应该提供这样的管理 进程/线程 状态的机制。想想我们为什么觉得 semaphore 和 mutex 在编程上容易出错，**因为我们需要去亲自操作变量以及对 进程/线程 进行阻塞和唤醒**。**monitor** 这个机制之所以被称为“更高级的原语”，那么它就不可避免地需要对外屏蔽掉这些机制，并且**在内部实现这些机制**，使得使用 monitor 的人看到的是一个简洁易用的接口。

**monitor 基本元素**
monitor 机制需要几个元素来配合，分别是：

1. 临界区
2. monitor 对象及锁
3. 条件变量以及定义在 monitor 对象上的 wait，signal 操作。

使用 monitor 机制的**目的主要是为了互斥进入临界区**，为了做到能够阻塞无法进入临界区的 进程/线程，还需要一个 monitor object 来协助，这个 monitor object 内部会有相应的数据结构，例如列表，来保存被阻塞的线程；同时由于 monitor 机制本质上是基于 mutex 这种基本原语的，所以 monitor object 还必须维护一个基于 mutex 的锁。
此外，**为了在适当的时候能够阻塞和唤醒 进程/线程，还需要引入一个条件变量**，这个条件变量用来决定什么时候是“适当的时候”，这个条件可以来自程序代码的逻辑，也可以是在 monitor object 的内部，总而言之，程序员对条件变量的定义有很大的自主性。不过，由于 monitor object 内部采用了数据结构来保存被阻塞的队列，**因此它也必须对外提供两个 API 来让线程进入阻塞状态以及之后被唤醒，分别是 wait 和 notify**。

### 4 Synchronized和Lock的区别？

* **Lock可以提高多个线程进行读操作的效率**。（可以通过readwritelock实现读写分离）
* 在**资源竞争不是很激烈的情况下**，Synchronized的性能要优于ReetrantLock，但是**在资源竞争很激烈的情况下**，Synchronized的性能会下降几十倍，但是ReetrantLock的性能能维持常态；
* ReentrantLock提供了多样化的同步，比如有时间限制的同步，可以被Interrupt的同步（synchronized的同步是不能Interrupt的）等。在资源竞争不激烈的情形下，性能稍微比synchronized差点点。但是当同步非常激烈的时候，synchronized的性能一下子能下降好几十倍。而**ReentrantLock**确还能维持常态。

![image-20210803135912079](D:\1书本笔记\java实战项目\image-20210803135912079.png)

#### 2 Lock是一个接口

![image-20210803140334681](D:\1书本笔记\java实战项目\image-20210803140334681.png)

　　	注意，当一个线程获取了锁之后，**是不会被interrupt()方法中断的**。**单独调用interrupt()方法不能中断正在运行过程中的线程**，只能**中断阻塞过程中的线程**。

​			ReentrantLock中的lockInterruptibly()方法使得线程可以在被阻塞时响应中断，比如一个线程t1通过lockInterruptibly()方法获取到一个可重入锁，并执行一个长时间的任务，另一个线程通过interrupt()方法就可以立刻打断t1线程的执行，来获取t1持有的那个可重入锁。

　	　因此当通过lockInterruptibly()方法获取某个锁时，是以可中断的情况下获取锁的。这个时候可以通过调用，如果不能获取到，只有进行等待的情况下，是**可以响应中断的**。

　　而用synchronized修饰的话，当一个线程处于等待某个锁的状态，是**无法被中断的**，只有一直等待下去。


### 5 对Volatile的认识

链接：https://blog.csdn.net/weixin_32265569/article/details/107425491

1. 保证可见性
2. 不保证原子性（volition不保证原子性，对于这一点可以使用 `j u c` 下的`atomic` 类来保证原子性）
3. 禁止指令重排序（volatile通过内存屏障来保证指令的有序性）

#### 1 volatile和synchronize的区别

![image-20210803141746455](D:\1书本笔记\java实战项目\image-20210803141746455.png)

#### 2 面试官：“请谈谈你对volatile的理解”

​		volatile是Java虚拟机提供的轻量级的同步机制，volatile 是一个类型修饰符。volatile 的作用是作为指令关键字，确保本条指令不会因编译器的优化而进行指令重排序，同时保证多线程环境下变量值变修改后，其他线程可见性；但volatile不保证原子性哦

#### 3 面试官接着问：“既然volatile不能保证原子性，那工作中如何解决这个问题呢？”

1. 使用JDK提供的 Atomic原子类

比如：AtomicInteger来声明变量，是采用了CAS 比较并交换 compareAndSwapInt，底层调用的是native方法，其意思是通过hotspot底层c/c++方法实现。最终实现是调用了 cmpxchg，cmpxchg指令在多线程下也是有可能被打断，所以在加入lock指令 不允许其他线程访问这块内存区域的数据。

2. 加锁，如synchronized 或 ReentrantLock

#### 4  官试官：“volatile是如何保证多线程环境下的可见性，JAVA内存模型JMM谈谈你对它的了解”

Java内存模型（Java Memory Model，简称JMM)  本身是一种抽象的概念并不真实存在，它描述的是一组规则或规范通过规范定制了程序中各个变量(包括实例字段、静态字段和构成数组对象的元素)的访问方式。

JMM关于同步规定:

1. 线程解锁前，必须把共享变量的值刷新回主内存

2. 线程加锁前，必须读取主内存的最新值到自己的工作内存

3. 加锁解锁是同一把锁

   ​			由于JVM运行程序的实体是线程，而每个线程创建时JVM都会为其创建一个工作内存(有些地方成为栈空间)，工作内存是每个线程的私有数据区域,而Java内存模型中规定所有变量都存储在**主内存**，主内存是共享内存区域,所有线程都可访问，**但线程对变量的操作 (读取赋值等) 必须在工作内存中进行，首先要将变量从主内存拷贝到自己的工作空间，然后对变量进行操作，操作完成再将变量写回主内存**，不能直接操作主内存中的变量，各个线程中的工作内存储存着主内存中的变量副本拷贝，因此不同的线程无法访问对方的工作内存，线程间的通讯(传值) 必须通过主内存来完成。
   ![image-20210803143456424](D:\1书本笔记\java实战项目\image-20210803143456424.png)

#### 5. 面试官：“为什么volatile不保证原子性呀，你知道底层原理么？”

i++在多线程下是非线程安全的。当我们添加volatile 关键字修饰时，i++ 同样被拆分成了3个指令，在多线程环境下，依然不能保证原子性。

#### 6  面试官：volatile能禁止指令重排 ，为什么有指令重排，volatile中怎样做到禁止指令重排？



#### 7  volatile的读写内存语义

* volatile写的内存语义：
   当写一个 volatile 变量时，**JMM 会把该线程对应的本地内存中的共享变量值刷新到主内存**。

* volatile读的内存语义：
   当读一个 volatile 变量时，JMM 会把该线程对应的本地内存置为无效。线程接下来将**从主内存中读取共享变量**。

**volatile的读写内存语义的实现原理：**

为了实现 volatile 的内存语义，编译器在生成字节码时，会在指令序列中插入内存屏障来禁止特定类型的处理器重排序。因为内存屏障是一组处理器指令，它并不由JVM直接暴露，因此JVM会根据不同的操作系统插入不同的指令以达成我们所要内存屏障效果。

为了保证内存可见性，java编译器在生成指令序列的适当位置会插入内存屏障指令来禁止特定类型的处理器重排序。JMM把内存屏障指令分为下列四类：

Load是将数据从内存中加载到各自的线程中去，Store是将数据由各自的线程写到内存中去。

![image-20210803152524160](D:\1书本笔记\java实战项目\image-20210803152524160.png)

下面是基于保守策略的JMM内存屏障插入策略：
 在每个volatile写操作的前面插入一个StoreStore屏障。
 在每个volatile写操作的后面插入一个StoreLoad屏障。
 在每个volatile读操作的后面插入一个LoadLoad屏障。
 在每个volatile读操作的后面插入一个LoadStore屏障。

![image-20210803152608878](D:\1书本笔记\java实战项目\image-20210803152608878.png)



![image-20210803152621744](D:\1书本笔记\java实战项目\image-20210803152621744.png)

### 6 简述对指令重排序的认识

**1、编译器重排序**

针对程序代码语而言，编译器可以在不改变单线程程序语义的情况下，可以对代码语句顺序进行调整重新排序。

**2、指令集并行的重排序**

这个是针对于CPU指令级别来说的，处理器采用了指令集并行技术来讲多条指令重叠执行，如果不存在数据依赖性，处理器可以改变主句对应的机器指令执行顺序。

**3、内存重排序**

因为CPU缓存使用 缓冲区的方式(Store Buffere )进行延迟写入，这个过程会造成多个CPU缓存可见性的问题，这种可见性的问题导致结果的对于指令的先后执行显示不一致，从表面结果上来看好像指令的顺序被改变了，内存重排序其实是造成可见性问题的主要原因所在，其原理可在上一篇可中详细了解。

![image-20210803155009703](D:\1书本笔记\java实战项目\image-20210803155009703.png)

**指令重排序的原则（as-if-serial语义）**

编译器和处理指令也并非什么场景都会进行指令重排序的优化，而是会遵循一定的原则，只有在它们认为重排序后不会对程序结果产生影响的时候才会进行重排序的优化，如果重排序会改变程序的结果，那这样的性能优化显然是没有意义的。而遵守as-if-serial 语义规则就是重排序的一个原则，as-if-serial 的意思是说，**可以允许编译器和处理器进行重排序，但是有一个条件，就是不管怎么重排序都不能改变单线程执行程序的结果**。

### 6 对final域的认识

链接 ： https://www.cnblogs.com/jojop/p/13971054.html

对于 final 域，编译器和处理器要遵守两个重排序规则。

1）在**构造函数内对一个 final 域的写入，与随后把这个被构造对象的引用赋值给一个引用变量**，这两个操作之间不能重排序；

2）**初次读一个包含 final 域的对象的引用，与随后初次读这个 final 域**，这两个操作之间不能重排序。

### 7 对原子类的理解

原子类可以弥补volatile不具有原子性的缺点。本质是通过调用CAS操作来实现的，java.util.concurrent.atomic 并发包下的所有原子类都是基于 CAS 来实现的。



### 8 顺序一致性模型和JMM模型的区别？

　**顺序一致性内存模型**是一个被计算机科学家理想化了的理论参考模型，**它为程序员提供了极强的内存可见性保证。顺序一致性内存模型有两大特性**。
1）一个线程中的所有操作必须按照程序的顺序来执行。
2）（不管程序是否同步）所有线程都只能看到一个单一的操作执行顺序。在顺序一致性内存模型中，每个操作都必须原子执行且立刻对所有线程可见。

![image-20210805211757929](D:\1书本笔记\java实战项目\image-20210805211757929.png)

![image-20210805211808583](D:\1书本笔记\java实战项目\image-20210805211808583.png)



​		**JMM中**，未同步的程序不但整体执行顺序无序，而且所有线程看到的执行顺序也可能不同。比如对一个普通变量的写操作，在当前线程未把本地缓存数据刷新到主内存当中之前，仅对当前线程可见，其他线程不可见。在这种情况下，当前线程和其他线程看到的操作执行顺序将不一致。

```java
int a = 0;
boolean flag = false;
public synchronized void writer() { // 获取锁
a = 1;
flag = true;
} // 释放锁
public synchronized void reader() { // 获取锁
if (flag) {
int i = a;
……
} // 释放锁
}
}
```

![image-20210805212309621](D:\1书本笔记\java实战项目\image-20210805212309621.png)



**对比**：

1）**顺序一致性模型保证单线程内的操作会按程序的顺序执行**，而**JMM不保证单线程内的**
**操作会按程序的顺序执行**（比如上面正确同步的多线程程序在临界区内的重排序）。
2）顺序一致性模型保证**所有线程只能看到一致的操作执行顺序**，而**JMM不保证所有线程**
**能看到一致的操作执行顺序**。
3）**JMM不保证对64位的long型和double型变量的写操作具有原子**性，而**顺序一致性模型保**
**证对所有的内存读/写操作都具有原子性**。



## 3 锁

### 1 CAS的理解

#### 1 CAS的源码

CAS是英文单词**CompareAndSwap**的缩写，中文意思是：比较并替换。CAS需要有3个操作数：内存地址V，旧的预期值A，即将要更新的目标值B。

CAS指令执行时，当且仅当内存地址V的值与预期值A相等时，将内存地址V的值修改为B，否则就什么都不做。整个比较并替换的操作是一个原子操作。

**源码分析**

![image-20210803163315871](D:\1书本笔记\java实战项目\image-20210803163315871.png)

![image-20210803163327639](D:\1书本笔记\java实战项目\image-20210803163327639.png)

**Atomic::cmpxchg方法解析**：

mp是“os::is_MP()”的返回结果，“os::is_MP()”是一个内联函数，用来判断当前系统是否为多处理器。

1. 如果当前系统是多处理器，该函数返回1。
2. 否则，返回0。

**LOCK_IF_MP(mp)**会根据mp的值来决定是否为cmpxchg指令添加lock前缀。

1. 如果通过**mp判断当前系统是多处理器（即mp值为1）**，则为**cmpxchg指令添加lock前缀**。
2. 否则，不加lock前缀。

这是一种优化手段，认**为单处理器的环境没有必要添加lock前缀**，只有在多核情况下才会添加lock前缀，因为lock会导致性能下降。**cmpxchg是汇编指令**，作用是比较并交换操作数。

**intel手册对lock前缀的说明如下：**

1. 确保对内存的读-改-写操作原子执行。在Pentium及Pentium之前的处理器中，带有lock前缀的指令在执行期间会锁住总线，使得其他处理器暂时无法通过总线访问内存。很显然，这会带来昂贵的开销。从Pentium 4，Intel Xeon及P6处理器开始，intel在原有总线锁的基础上做了一个很有意义的优化：如果要访问的内存区域（area of memory）在lock前缀指令执行期间已经在处理器内部的缓存中被锁定（即包含该内存区域的缓存行当前处于独占或以修改状态），并且该内存区域被完全包含在单个缓存行（cache line）中，那么处理器将直接执行该指令。由于在指令执行期间该缓存行会一直被锁定，其它处理器无法读/写该指令要访问的内存区域，因此能保证指令执行的原子性。**这个操作过程叫做缓存锁定**（cache locking），缓存锁定将大大降低lock前缀指令的执行开销，但是当多处理器之间的竞争程度很高或者指令访问的内存地址未对齐时，仍然会锁住总线。
2. 禁止该指令与之前和之后的读和写指令重排序。
3. 把写缓冲区中的所有数据刷新到内存中。

上面的**第1点保证了CAS操作是一个原子操作**，第2点和第3点所具有的**内存屏障效果**，保证了**CAS同时具有volatile读和volatile写的内存语义**。

**CAS为什么可以保证可见性**，是因为，在原子类里面有一个Unsafe方法用来获取内存中的值的，这个时候，由于**它的变量是声明为volatile的**，所以可以保证了可见性。

![image-20210803170622755](D:\1书本笔记\java实战项目\image-20210803170622755.png)



#### 2 CPU如何实现原子操作

CPU 处理器速度远远大于在主内存中的，为了解决速度差异，在他们之间架设了多级缓存，如 L1、L2、L3 级别的缓存，这些缓存离CPU越近就越快，将频繁操作的数据缓存到这里，加快访问速度 ，如下图所示：

![image-20210803170931864](D:\1书本笔记\java实战项目\image-20210803170931864.png)

现在都是多核 CPU 处理器，每个 CPU 处理器内维护了一块字节的内存，每个内核内部维护着一块字节的缓存，当多线程并发读写时，就会出现缓存数据不一致的情况。

此时，处理器提供：

- **总线锁定**

当一个处理器要操作共享变量时，在 BUS 总线上发出一个 Lock 信号，其他处理就无法操作这个共享变量了。

缺点很明显，总线锁定在阻塞其它处理器获取该共享变量的操作请求时，也可能会导致大量阻塞，从而增加系统的性能开销。

- **缓存锁定**

后来的处理器都提供了缓存锁定机制，也就说当某个处理器对缓存中的共享变量进行了操作，其他处理器会有个嗅探机制，将其他处理器的该共享变量的缓存失效，待其他线程读取时会重新从主内存中读取最新的数据，基于 MESI 缓存一致性协议来实现的。

现代的处理器基本都支持和使用的缓存锁定机制。

注意：

有如下两种情况处理器不会使用缓存锁定：

（1）当操作的数据跨多个缓存行，或没被缓存在处理器内部，则处理器会使用总线锁定。

（2）有些处理器不支持缓存锁定，比如：Intel 486 和 Pentium 处理器也会调用总线锁定。



#### 3 CAS的缺点

CAS虽然很高效的解决了原子操作问题，但是CAS仍然存在三大问题。

1. 循环时间长开销很大（消耗CPU）。
2. 只能保证**一个共享变量**的原子操作。（**这里需要注意的一点数CAS操作都是针对的是共享变量的操作**！！）
3. ABA问题。

**循环时间长开销很大：**我们可以看到getAndAddInt方法执行时，如果CAS失败，会一直进行尝试。如果CAS长时间一直不成功，可能会给CPU带来很大的开销。

**只能保证一个共享变量的原子操作：**当对一个共享变量执行操作时，我们可以使用循环CAS的方式来保证原子操作，但是对多个共享变量操作时，循环CAS就无法保证操作的原子性，这个时候就可以用锁来保证原子性。

**什么是ABA问题？ABA问题怎么解决？**

（ABA问题，实际上的意思是前面的A和后面的A不一样，但是计算机以为是一样的，就导致了自选成功，造成了ABA问题。）

CAS 的使用流程通常如下：1）首先从地址 V 读取值 A；2）根据 A 计算目标值 B；3）通过 CAS 以原子的方式将地址 V 中的值从 A 修改为 B。

但是在第1步中读取的值是A，并且在第3步修改成功了，我们就能说它的值在第1步和第3步之间没有被其他线程改变过了吗？

如果在这段期间它的值曾经被改成了B，后来又被改回为A，**那CAS操作就会误认为它从来没有被改变过**。这个漏洞称**为CAS操作的“ABA”问题**。Java并发包为了解决这个问题，提供了一个带有标记的原子引用类“**AtomicStampedReference**”，它可以通过**控制变量值的版本来保证CAS的正确性**，它通过包装[E,Integer]的元组来对对象标记版本戳stamp，从而避免ABA问题。因此，在使用CAS前要考虑清楚“ABA”问题是否会影响程序并发的正确性，如果需要解决ABA问题，改用传统的互斥同步可能会比原子类更高效。

#### 3 ABA问题

先来看看维基对 ABA problem 的描述：

> In multithreaded computing, the ABA problem occurs during synchronization, when a location is read twice, has the same value for both reads, and "value is the same" is used to indicate "nothing has changed".
> However, another thread can execute between the two reads and change the value, do other work, then change the value back, thus fooling the first thread into thinking "nothing has changed" even though the second thread did work that violates that assumption.

我拆分成两段来解读 wiki 的描述，第一段内容主要描述了 ABA 问题出现的场景和条件，在多线程环境中，某个location（或可以理解为某内存地址指向的变量）会被一个线程**连续重复读取**两次，那么只要第一次读取的值和第二次读取的值一样，那么这个线程就会认为这个变量在两次读取时间间隔内**没有发生任何变化**；

但是第二段告诉我们这种判定值是否发生变更的方式是**有问题**的。当然，在单线程环境下，确实可以保证说当一个线程对同一个内存地址连续读取两次，如若取值没有变化，就可以认为内存地址的值没有被修改过；而在多线程环境下，在两次读取的时间间隔内，其他线程很可能对这个值做了修改，然后又改回原值，这似乎给此时正在重复读取变量的线程造成了该内存变量**没有发生变化**的错觉。

这就是 ABA problem。

**归结起来，构成 ABA 问题有三个重要的条件**：

1. 某个线程需要重复读某个内存地址，并以内存地址的值变化作为该值是否变化的唯一判定依据；
2. 重复读取的变量会被多线程共享，且存在『值回退』的可能，即值变化后有可能因为某个操作复归原值；
3. 在多次读取间隔中，开发者没有采取有效的同步手段，比如上锁。

#### 4 CAS实现原子操作的三大问题和解决方案

1. 循环时间长开销很大（消耗CPU）。
2. 只能保证**一个共享变量**的原子操作。（**这里需要注意的一点数CAS操作都是针对的是共享变量的操作**！！）
3. ABA问题。

1. 自旋CAS如果长时间不成功，会给CPU带来非常大的执行开销。如果JVM能支持处理器提供的pause指令那么效率会有一定的提升，pause指令有两个作用，第一它可以延迟流水线执行指令（de-pipeline）,使CPU不会消耗过多的执行资源，延迟的时间取决于具体实现的版本，在一些处理器上延迟时间是零。第二它可以避免在退出循环的时候因内存顺序冲突（memory order violation）而引起CPU流水线被清空（CPU pipeline flush），从而提高CPU的执行效率。
2. 当对一个共享变量执行操作时，我们可以使用循环CAS的方式来保证原子操作，但是对多个共享变量操作时，循环CAS就无法保证操作的原子性，这个时候就可以用锁，或者有一个取巧的办法，就是把多个共享变量合并成一个共享变量来操作。比如有两个共享变量i＝2,j=a，合并一下ij=2a，然后用CAS来操作ij。从Java1.5开始JDK提供了**AtomicReference类来保证引用对象之间的原子性，你可以把多个变量放在一个对象里来进行CAS操作。**
3. Java并发包为了解决这个问题，提供了一个带有标记的原子引用类“**AtomicStampedReference**”，它可以通过**控制变量值的版本来保证CAS的正确性**，它通过包装[E,Integer]的元组来对对象标记版本戳stamp，从而避免ABA问题。因此，在使用CAS前要考虑清楚“ABA”问题是否会影响程序并发的正确性，如果需要解决ABA问题，改用传统的互斥同步可能会比原子类更高效。

#### 5 CAS与Synchronized使用场景

CAS 适用于写比较少的情况下（多读场景，冲突一般较少），synchronized 适用于写比较多的情况下（多写场景，冲突一般较多）。

1. 对于资源竞争较少（线程冲突较轻）的情况，使用 synchronized 同步锁进行线程阻塞，唤醒切换，以及用户态内核态间的切换操作，都会额外消耗 cpu 资源；而 CAS 基于硬件实现，不需要进入内核，不需要切换线程，操作自旋几率较少，因此可以获得更高的性能
2. 对于资源竞争严重（线程冲突严重）的情况，CAS 自旋的概率会比较大，从而浪费更多的 CPU 资源，效率低于 synchronized

### 2 AQS的理解

**AbstractQueuedSynchronizer，抽象队列同步器**，

​		源码里面AbstractQueuedSynchronizer类里面有一个成员内部类，和一个静态内部类。其中Node为静态内部类，ConditionObject为成员内部类。

![image-20210805195206975](D:\1书本笔记\java实战项目\image-20210805195206975.png)

![image-20210826213914431](D:\1书本笔记\java实战项目\image-20210826213914431.png)

**state状态的含义：**

1. 当state=1时，则说明有线程目前正在使用共享变量，其他线程必须加入同步队列进行等待
2. 当state=0时，则说明没有任何线程占有共享资源的锁

​		head和tail分别是AQS中的变量，其中head指向同步队列的头部，注意head为空结点，不存储信息。而tail则是同步队列的队尾，同步队列采用的是双向链表的结构这样可方便队列进行结点增删操作。state变量则是代表同步状态，执行当线程调用lock方法进行加锁后，如果此时state的值为0，则说明当前线程可以获取到锁(在本篇文章中，锁和同步状态代表同一个意思)，同时将state设置为1，表示获取成功。

![image-20210826214306486](D:\1书本笔记\java实战项目\image-20210826214306486.png)

**Node节点中各种状态的含义:**

1. CANCELLED：值为1，在同步队列中等待的线程等待超时或被中断，需要从同步队列中取消该Node的结点，其结点的waitStatus为CANCELLED，即结束状态，进入该状态后的结点将不会再变化。

2. SIGNAL：值为-1，被标识为该等待唤醒状态的后继结点，当其前继结点的线程释放了同步锁或被取消，将会通知该后继结点的线程执行。说白了，就是处于唤醒状态，只要前继结点释放锁，就会通知标识为SIGNAL状态的后继结点的线程执行。

3. CONDITION：值为-2，与Condition相关，该标识的结点处于等待队列中，结点的线程等待在Condition上，当其他线程调用了Condition的signal()方法后，CONDITION状态的结点将从等待队列转移到同步队列中，等待获取同步锁。

4. PROPAGATE：值为-3，与共享模式相关，在共享模式中，该状态标识结点的线程处于可运行状态。

5.  0状态：值为0，代表初始化状态。


#### 1 AQS的原理

链接：https://zhuanlan.zhihu.com/p/65349219，https://juejin.cn/post/6975435256111300621#heading-0，

这个链接讲得最详细：https://blog.csdn.net/javazejian/article/details/75043422

**AQS就是一个并发包的基础组件，用来实现各种锁，各种同步组件的。它包含了state变量、加锁线程、等待队列等并发中的核心组件**

关于AQS我觉得比较重要的就是获取资源和释放资源的方法，里面用到了大量的CAS操作和自旋。AQS里面维护了一个同步状态，两个队列，**State状态**，一个是**等待队列**（CHL，Craig, Landin, and Hagersten (CLH) locks，同步锁），还有一个是**条件队列**（condition）。



1. acquire()尝试获取资源，如果获取失败，将线程插入等待队列。插入后，并没有放弃获取资源，而是根据前置节点状态状态判断是否应该继续获取资源。如果前置节点是头结点，继续尝试获取资源；如果前置节点是SIGNAL状态，就中断当前线程，否则继续尝试获取资源。直到当前线程被阻塞或者获取到资源，结束。
2. release()释放资源，需要唤醒后继节点,判断后继节点是否满足情况。如果后继节点不为空且不是作废状态,则唤醒这个后继节点；否则从尾部往前面找适合的节点，找到则唤醒。
   调用await()，线程会进入条件队列，等待被唤醒，唤醒后以自旋方式获取资源或处理中断异常；调用signal()，线程会插入到等待队列，唤醒被阻塞的线程。

#### 2 CLH同步队列？

![image-20210804231614828](D:\1书本笔记\java实战项目\image-20210804231614828.png)

**入列：**

将tail（使用CAS保证原子操作）指向新节点，新节点的prev指向队列中最后一节点（旧的tail节点），原队列中最后一节点的next节点指向新节点以此来建立联系，

![image-20210804231836626](D:\1书本笔记\java实战项目\image-20210804231836626.png)

入列源码：

```java
//先通过addWaiter(Node node)方法尝试快速将该节点设置尾成尾节点，设置失败走enq(final Node node)方法
private Node addWaiter(Node mode) {
// 以给定的模式来构建节点， mode有两种模式 
//  共享式SHARED， 独占式EXCLUSIVE;
  Node node = new Node(Thread.currentThread(), mode);
    // 尝试快速将该节点加入到队列的尾部
    Node pred = tail;
     if (pred != null) {
        node.prev = pred;
            if (compareAndSetTail(pred, node)) {
                pred.next = node;
                return node;
            }
        }
        // 如果快速加入失败，则通过 anq方式入列
        enq(node);
        return node;
    }

//通过“自旋”也就是死循环的方式来保证该节点能顺利的加入到队列尾部，只有加入成功才会退出循环，否则会一直循序直到成功
private Node enq(final Node node) {
// CAS自旋，直到加入队尾成功        
for (;;) {
    Node t = tail;
        if (t == null) { // 如果队列为空，则必须先初始化CLH队列，新建一个空节点标识作为Hader节点,并将tail 指向它
            if (compareAndSetHead(new Node()))
                tail = head;
            } else {// 正常流程，加入队列尾部
                node.prev = t;
                    if (compareAndSetTail(t, node)) {
                        t.next = node;
                        return t;
                }
            }
        }
    }
```



**出列:**

同步队列（CLH）遵循FIFO，首节点是获取同步状态的节点，首节点的线程释放同步状态后，将会唤醒它的后继节点（next），而后继节点将会在获取同步状态成功时将自己设置为首节点，这个过程非常简单。如下图

![image-20210804232057892](D:\1书本笔记\java实战项目\image-20210804232057892.png)

设置首节点是通过获取同步状态成功的线程来完成的（获取同步状态是通过CAS来完成），只能有一个线程能够获取到同步状态，因此设置头节点的操作并不需要CAS来保证，只需要将首节点设置为其原首节点的后继节点并断开原首节点的next（等待GC回收）应用即可。







#### 3  AQS对资源的共享方式

**1)Exclusive**（独占）(如ReetrantLock)

**2)Share**（共享）(如Semaphore)

#### 4 AQS重要的API？

1. tryAcquire()  通过CAS自旋来尝试获取锁
2. addWaiter()  如果获取所失败，就将当前线程加入同步等待队列
3. acquireQueued()  自旋方式获取资源并判断是否需要被挂起
4. acquire(int arg) 尝试获取独占锁
5. release(int arg) 释放独占锁
6. acquireShared() 尝试获取共享锁
7. releaseShared(int arg) 释放共享锁

#### 5 CountDownLatch

###### **1 CountDownLatch的用法**

CountDownLatch典型用法：1、某一线程在开始运行前等待n个线程执行完毕。将CountDownLatch的计数器初始化为new CountDownLatch(n)，每当一个任务线程执行完毕，就将计数器减1 countdownLatch.countDown()，当计数器的值变为0时，在CountDownLatch上await()的线程就会被唤醒。一个典型应用场景就是启动一个服务时，主线程需要等待多个组件加载完毕，之后再继续执行。

CountDownLatch典型用法：2、实现多个线程开始执行任务的最大并行性。注意是并行性，不是并发，强调的是多个线程在某一时刻同时开始执行。类似于赛跑，将多个线程放到起点，等待发令枪响，然后同时开跑。做法是初始化一个共享的CountDownLatch(1)，将其计算器初始化为1，多个线程在开始执行任务前首先countdownlatch.await()，当主线程调用countDown()时，计数器变为0，多个线程同时被唤醒。

###### **2 CountDownLatch的不足**

CountDownLatch是一次性的，计算器的值只能在构造方法中初始化一次，之后没有任何机制再次对其设置值，当CountDownLatch使用完毕后，它不能再次被使用。

###### 3 CountDownLatch源码分析

链接：https://blog.csdn.net/qq_32459653/article/details/81486757?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-4.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-4.control

CountDownLatch 阻塞：

1. 首先通过调用await() 方法进行阻塞，该放法调用doAcquireSharedInterruptibly(arg);

![image-20210805132452114](D:\1书本笔记\java实战项目\image-20210805132452114.png)

1. doAcquireSharedInterruptibly（arg）方法里面首先将当前线程的节点插入链表的尾部，然后无限for循环获取当前节点的上一个节点，其中如果上一个节点**是头节点**，那么会尝试的获取当前贡献锁（当每次调用countDown()函数，arg都会减一，所以当没有达到条件也就是state不等于0将会返回-1）,如果state为0的时候，就将当前节点设置成头节点，否则就一直wait.

![image-20210805132559399](D:\1书本笔记\java实战项目\image-20210805132559399.png)

1. 其中如果上一个节点**不是头节点**，需要调用shouldParkAfterFailedAcquire（p, node）方法来判断当前的节点是不是需要被挂起（**言外之意是不是需要被阻塞**）。只要当当前节点需要被挂起的时候才会调用后面的**parkAndCheckInterrupt（）**方法来**阻塞自身，基本每个调用await函数都阻塞在这里**。

![image-20210805132739568](D:\1书本笔记\java实战项目\image-20210805132739568.png)



CountDownLatch 唤醒：

1. countDown这个函数的玄机吧，线程就是通过这个来函数来触发唤醒条件，首先或通过调用tryReleaseShared（）函数，对state经行递减操作，state减1后为0时才会返回为真 执行后面的唤醒条件。

![image-20210805132845012](D:\1书本笔记\java实战项目\image-20210805132845012.png)

1. 实际的唤醒是调用doReleaseShared()方法里面的unparkSuccessor（h）方法来进行唤醒操作：首先唤醒的时候会使用CAS将当前节点的state状态自选的置为0，当CAS成功后会调用unparkSuccessor（h）方法来唤醒节点，unparkSuccessor（h）方法是取该节点的**后节点就行唤醒**,如果后节点已被取消，则从最后一个开始往前找，找一个满足添加的节点进行唤醒，唤醒的时候调用的是**LockSupport.unpark**(s.thread)方法，将节点唤醒。

```java
private void doReleaseShared() {
    /*
     * Ensure that a release propagates, even if there are other
     * in-progress acquires/releases.  This proceeds in the usual
     * way of trying to unparkSuccessor of head if it needs
     * signal. But if it does not, status is set to PROPAGATE to
     * ensure that upon release, propagation continues.
     * Additionally, we must loop in case a new node is added
     * while we are doing this. Also, unlike other uses of
     * unparkSuccessor, we need to know if CAS to reset status
     * fails, if so rechecking.
     */
    for (;;) {
        Node h = head; //获取头节点，
        if (h != null && h != tail) {
            int ws = h.waitStatus;
            if (ws == Node.SIGNAL) { //头结点的状态为Node.SIGNAL
                if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                    continue;            // loop to recheck cases
                unparkSuccessor(h); //这里唤醒 很重要哦
            }
            else if (ws == 0 &&
                     !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                continue;                // loop on failed CAS
        }
        if (h == head)                   // loop if head changed
            break;  //这里是否有疑问明明都有这个 Node h = head为啥还要在判断一次？多次一举别着急后面有原因
    }
}
```



1. 如果有多个节点只在这进行一次唤醒工作吗？难道只唤醒一个线程就可以了？哈哈别急还记得线程是在哪阻塞的吗 让我们回来前面去看线程被阻塞的地方,(因为这里在一直不断的唤醒)，这里的自选操作是在一直获取节点的状态，当判断节点的状态为0的时候，就应该在doAcquireSharedInterruptibly（arg）方法里面被唤醒。

![image-20210805132951200](D:\1书本笔记\java实战项目\image-20210805132951200.png)

#### 6 CyclicBarrier

###### 1 CyclicBarrier的使用

“循环栅栏”。大概的意思就是一个可循环利用的屏障。，它的作用就是会让所有线程都等待完成后才会继续下一步行动。





#### 4. CyclicBarrier 与 CountDownLatch 区别

- CountDownLatch 是一次性的，CyclicBarrier 是可循环利用的
- CountDownLatch 参与的线程的职责是不一样的，有的在倒计时，有的在等待倒计时结束。CyclicBarrier 参与的线程职责是一样的。

1. CountdownLatch适用于所有线程通过某一点后通知方法,而CyclicBarrier则适合让所有线程在同一点同时执行 
2. CountdownLatch利用继承**AQS的共享锁来进行线程的通知**,利用**CAS来进行**--,而**CyclicBarrier则利用ReentrantLock的Condition来阻塞和通知线程**



#### 5 Semaphore

###### 1 Semaphore

**获得锁 Semaphore.acquire( )获取令牌**。

1、当前线程会尝试去同步队列获取一个令牌，获取令牌的过程也就是使用原子的操作去修改同步队列的state ,获取一个令牌则修改为state=state-1。

2、 当计算出来的state<0，则代表令牌数量不足，此时会创建一个Node节点加入阻塞队列，挂起当前线程。

3、当计算出来的state>=0，则代表获取令牌成功。

**释放锁Semaphore.release( )释放令牌**

1、线程会尝试释放一个令牌，释放令牌的过程也就是把同步队列的state修改为state=state+1的过程

2、释放令牌成功之后，同时会唤醒同步队列中的一个线程。

3、被唤醒的节点会重新尝试去修改state=state-1 的操作，如果state>=0则获取令牌成功，否则重新进入阻塞队列，挂起线程。

###### 2 Semaphore.CyclicBarrier.CountDownLatch三者之间的区别

1) CountDownLatch和CyclicBarrier都能够实现线程之间的等待，只不过它们侧重点不同；

CountDownLatch一般用于某个线程A等待若干个其他线程执行完任务之后，它才执行；

而CyclicBarrier一般用于一组线程互相等待至某个状态，然后这一组线程再同时执行；

另外，CountDownLatch是不能够重用的，而CyclicBarrier是可以重用的。

2) Semaphore其实和锁有点类似，它一般用于控制对某组资源的访问权限。

### 3 乐观锁和悲观锁存在的问题

悲观锁：

存在的问题：
1）多线程竞争下，加锁、解锁都会导致线程上下文切换和cpu调度延迟，性能问题存在
2）一个线程持有锁会导致其他需要此锁的线程阻塞
3）如果一个优先级高的线程等待优先级低的线程释放锁会导致优先级导致，性能问题存在

乐观锁：

1、循环时间太长。
自旋长时间不成功，CPU开销过大，所以需要限制自旋次数。

2、只能保证一个共享变量原子操作。
CAS源码中只记录了一个对象的内存位置、预期原值，所以只能保证一个共享变量原子操作。

3、ABA问题。
CAS写之前检查操作值有没有改变，如果改变则更新。

存在问题：本来是A，改成了B，又改成了A。

在多线程中，其实是已经改变过的，这是个问题。

AtomicInteger存在ABA问题，使用AtomicStampedReference解决ABA问题。


### 4 可重入锁的认识

**可重入就是说某个线程已经获得某个锁，可以再次获取锁而不会出现死锁**

可重入锁有

- synchronized
- ReentrantLock

### 5 共享锁和独占锁的理解

**共享锁：**

​		共享锁是指该锁可被多个线程所持有。如果线程T对数据A加上共享锁后，则其他线程只能对A再加共享锁，不能加排它锁。获得共享锁的线程只能读数据，不能修改数据。 独享锁与共享锁也是通过AQS来实现的，通过实现不同的方法，来实现独享或者共享。

**独占锁：**

​		独占锁也叫排他锁，是指该锁一次只能被一个线程所持有。如果线程T对数据A加上排他锁后，则其他线程不能再对A加任何类型的锁。获得排它锁的线程即能读数据又能修改数据。JDK中的synchronized和 JUC中Lock的实现类就是互斥锁。

举例：ReentrantReadWriteLock 有两把锁：ReadLock和WriteLock。

​		读锁是共享锁，写锁是独占锁。读锁的共享锁可保证并发读非常高效，而读写、写读、写写的过程互斥，因为读锁和写锁是分离的。所以ReentrantReadWriteLock的并发性相比一般的互斥锁有了很大提升

### 6 公平锁和非公平锁的认识

公平锁：多个线程按照申请锁的顺序去获得锁，线程会直接进入队列去排队，永远都是队列的第一位才能得到锁。

1. 优点：所有的线程都能得到资源，不会饿死在队列中。

2. 缺点：吞吐量会下降很多，队列里面除了第一个线程，其他的线程都会阻塞，cpu唤醒阻塞线程的开销会很大。

   

非公平锁：多个线程去获取锁的时候，会直接去尝试获取，获取不到，再去进入等待队列，如果能获取到，就直接获取到锁。

1. 优点：可以减少CPU唤醒线程的开销，整体的吞吐效率会高点，CPU也不必取唤醒所有线程，会减少唤起线程的数量。
2. 缺点：你们可能也发现了，这样可能导致队列中间的线程一直获取不到锁或者长时间获取不到锁，导致饿死。

### 7  **LockSupport**

*  LockSupport提供的是一个许可，如果存在许可。线程在调用`park`的时候，会立马返回，此时许可也会被消费掉，如果没有许可，则会阻塞。调用unpark的时候，如果许可本身不可用，则会使得许可可用。

> ​		 许可只有一个，不可累加

LockSupport可以理解为一个工具类。它的作用很简单，就是挂起和继续执行线程。它的常用的API如下：

- public static void park() : 如果没有可用许可，则挂起当前线程
- public static void unpark(Thread thread)：给thread一个可用的许可，让它得以继续执行

因为单词park的意思就是停车，因此这里park()函数就表示让线程暂停。反之，unpark()则表示让线程继续执行。

需要注意的是，LockSupport本身也是基于许可的实现，如何理解这句话呢，请看下面的代码：

```text
LockSupport.unpark(Thread.currentThread());
LockSupport.park();
```

大家可以猜一下，park()之后，当前线程是停止，还是 可以继续执行呢？

答案是：可以继续执行。那是因为在park()之前，先执行了unpark()，进而释放了一个许可，也就是说当前线程有一个可用的许可。而park()在有可用许可的情况下，是不会阻塞线程的。

综上所述，park()和unpark()的执行效果和它调用的先后顺序没有关系。这一点相当重要，因为在一个多线程的环境中，我们往往很难保证函数调用的先后顺序(都在不同的线程中并发执行)，因此，这种基于许可的做法能够最大限度保证程序不出错。

### 8 对自旋锁的认识

​		  自旋锁是SMP中经常使用到的一个锁。所谓的smp，就是对称多处理器的意思。在工业用的pcb板上面，特别是服务器上面，一个pcb板有多个cpu是很正常的事情。这些cpu相互之间是独立运行的，每一个cpu均有自己的调度队列。然而，这些cpu在内存空间上是共享的。举个例子说，假设有一个数据value = 10，那么这个数据可以被所有的cpu访问。这就是共享内存的本质意义。


static inline void __raw_spin_lock(raw_spinlock_t *lock)
{
	asm volatile("\n1:\t"
		     LOCK_PREFIX " ; decb %0\n\t"
		     "jns 3f\n"
		     "2:\t"
		     "rep;nop\n\t"
		     "cmpb $0,%0\n\t"
		     "jle 2b\n\t"
		     "jmp 1b\n"
		     "3:\n\t"
		     : "+m" (lock->slock) : : "memory");
}

上面这段代码是怎么做到自旋锁的呢？我们可以一句一句看看，

line  4: 对lock->slock自减，这个操作是互斥的，LOCK_PREFIX保证了此刻只能有一个CPU访问内存
line  5: 判断lock->slock是否为非负数，如果是跳转到3，即获得自旋锁
line  6: 位置符
line  7: lock->slock此时为负数，说明已经被其他cpu抢占了，cpu休息一会，相当于pause指令
line  8: 继续将lock->slock和0比较，
line  9: 判断lock->slock是否小于等于0，如果判断为真，跳转到2，继续休息
line 10: 此时lock->slock已经大于0，可以继续尝试抢占了，跳转到1
line 11: 位置符 

总结：
  1）在smp上自旋锁是多cpu互斥访问的基础
  2）因为自旋锁是自旋等待的，所以处于临界区的代码应尽可能短
  3）上面的LOCK_PREFIX，在x86下面其实就是“lock”，gcc下可以编过，朋友们可以自己试试

### 9 锁升级

![preview](https://pic3.zhimg.com/v2-79edfb4b2316d76ac653732fbdb72809_r.jpg?source=1940ef5c)



简单版本：





![image-20210805142936316](D:\1书本笔记\java实战项目\image-20210805142936316.png)

​		当对象状态为偏向锁时， Mark Word 存储的是**偏向的线程ID**；当状态为轻量级锁时， Mark Word 存储的是指**向线程栈中 Lock Record 的指针**；当状态为重量级锁时， Mark Word 为**指向堆中的monitor对象的指针**  

#### 1 偏向锁

**概念**

​		Hotspot的作者经过以往的研究发现⼤多数情况下锁不仅不存在多线程竞争，⽽且总是**由同⼀线程多次获得**，于是引⼊了**偏向锁**。偏向锁会偏向于第⼀个访问锁的线程，如果在接下来的运⾏过程中，该锁没有被其他的线程访问，则持有偏向锁的线程将永远不需要触发同步。也就是说，**偏向锁在资源⽆竞争情况下消除了同步语句**，**连CAS操作都不做了**，提⾼了程序的运⾏性能 。



![image-20210805143154999](D:\1书本笔记\java实战项目\image-20210805143154999.png)

**实现原理**：

⼀个线程在第⼀次进⼊同步块时，会在对象头和栈帧中的锁记录⾥存储锁的偏向的线程ID。当下次该线程进⼊这个同步块时，会去检查锁的Mark Word⾥⾯是不是放的⾃⼰的线程ID。



如果是，表明**该线程已经获得了锁**，以后该线程在**进⼊和退出同步块时不需要花费CAS操作来加锁和解锁** ；如果不是，就代表有另⼀个线程来竞争这个偏向锁。这
个时候会尝试**使⽤CAS来替换Mark Word⾥⾯的线程ID为新线程的ID**，这个时候要
分两种情况：

1. **成功**，表示之前的线程不存在了， Mark Word⾥⾯的线程ID为新线程的ID，**锁不会升级**，**仍然为偏向锁**；
2. **失败**，表示之前的**线程仍然存在**，那么暂停之前的线程，设置偏向锁标识为0，**并设置锁标志位为00，升级为轻量级锁**，会按照轻量级锁的⽅式进⾏竞争锁。  

具体流程：

![image-20210805145708510](D:\1书本笔记\java实战项目\image-20210805145708510.png)

#### 2 轻量锁

概念：

​		多个线程在不同时段获取同⼀把锁，即不存在锁竞争的情况，也就没有线程阻塞。针对这种情况，JVM采⽤轻量级锁来避免线程的阻塞与唤醒。  

JVM会为每个线程在当前线程的栈帧中创建⽤于存储锁记录的空间，**我们称为Displaced Mark Word**。如果⼀个线程获得锁的时候发现是轻量级锁，会把锁的
Mark Word复制到⾃⼰的**Displaced Mark Word⾥⾯**。

然后线程尝试⽤**CAS将锁的Mark Word替换为指向锁记录的指针**。如果成功，当前线程获得锁，如果失败，表示Mark Word已经被替换成了其他线程的锁记录，说明在与其它线程竞争锁，当前线程就尝试使⽤⾃旋来获取锁。

⾃旋：不断尝试去获取锁，⼀般⽤循环来实现。

⾃旋是需要消耗CPU的，如果⼀直获取不到锁的话，那该线程就⼀直处在⾃旋状态，⽩⽩浪费CPU资源。解决这个问题最简单的办法就是指定⾃旋的次数，例如让
其循环10次，如果还没获取到锁就进⼊阻塞状态。

但是JDK采⽤了更聪明的⽅式——适应性⾃旋，简单来说就是线程如果⾃旋成功了，则下次⾃旋的次数会更多，如果⾃旋失败了，则⾃旋的次数就会减少。

⾃旋也不是⼀直进⾏下去的，如果⾃旋到⼀定程度（和JVM、操作系统相关），依然没有获取到锁，称为⾃旋失败，那么这个线程会阻塞。同时这个锁就会升级成**重量级锁**。  



轻量级锁的释放：
在释放锁时，当前线程会使⽤CAS操作将Displaced Mark Word的内容复制回锁的Mark Word⾥⾯。**如果没有发⽣竞争，那么这个复制的操作会成功**。如果有其他线程**因为⾃旋多次导致轻量级锁升级成了重量级锁**，那么**CAS操作会失败**，此时会**释放锁并唤醒被阻塞的线程**。  

![image-20210805150721088](D:\1书本笔记\java实战项目\image-20210805150721088.png)





#### 3 重量锁

​		重量级锁依赖于操作系统的互斥量（mutex） 实现的，⽽操作系统中线程间状态的转换需要相对⽐较⻓的时间，所以重量级锁效率很低，但被阻塞的线程不会消耗CPU。  

**需要注意的是，当调⽤⼀个锁对象的 wait 或 notify ⽅法时，如当前锁的状态是偏向锁或轻量级锁则会先膨胀成重量级锁**  





#### 4 总结锁的升级流程

每⼀个线程在准备获取共享资源时：

1. 第⼀步，检查MarkWord⾥⾯是不是放的⾃⼰的ThreadId ,如果是，表示当前线程是处于 “偏向锁” 。
2. 第⼆步，如果MarkWord不是⾃⼰的ThreadId，锁升级，这时候，⽤CAS来执⾏切换，新的线程根据MarkWord⾥⾯现有的ThreadId，通知之前线程暂停，之前线程将Markword的内容置为空。
3. 第三步，两个线程都把锁对象的HashCode复制到⾃⼰新建的⽤于存储锁的记录空间，接着开始通过CAS操作， 把锁对象的MarKword的内容修改为⾃⼰新建的记录空间的地址的⽅式竞争MarkWord。
4. 第四步，第三步中成功执⾏CAS的获得资源，失败的则进⼊⾃旋 。
5. 第五步，⾃旋的线程在⾃旋过程中，成功获得资源(即之前获的资源的线程执⾏完成并释放了共享资源)，则整个状态依然处于 轻量级锁的状态，如果⾃旋失败 
6. 第六步，进⼊重量级锁的状态，这个时候，⾃旋的线程进⾏阻塞，等待之前线程执⾏完成并唤醒⾃⼰  

### 10 锁清除和锁粗化的理解

**锁粗化**

Java虚拟机中存在着一种叫做锁粗化的优化方法，即同步范围变大。

比方说StringBuffer，它是一个线程安全的类，反复append字符串意味着要进行反复的加解锁，这对性能不利，因为JVM在这条线程上要反复地在内核态和用户态之间切换，因此JVM会将多次append方法调用的代码进行一个锁粗化的操作，将多次的append的操作扩展到append方法的头尾，变成一个大的同步块，从而提升代码执行效率。
 **锁清除**

锁消除是Java虚拟机在JIT编译是，通过对运行上下文的扫描，去除不可能存在共享资源竞争的锁，通过锁消除，可以节省毫无意义的请求锁时间。

### 11 分段锁的理解

**ConcurrentHashMap**使用了分段锁来保证线程安全，效率比起使用synchronized的**HashTable**要高的很多

### 12 处理器cpu如何实现原子操作

有两种方法：1 总线锁定   2 缓存锁定

​		首先处理器会自动保证基本内存操作的原子性。处理器保证从系统内存中读取或者写 入一个字节是原子的，意思是**当一个处理器读取一个字节时，其他处理器不能访问这个字节 的内存地址**。最新的处理器能自动保证单处理器对同一个缓存行里进行16/32/64位的操作是原子的，但是复杂的内存操作处理器是不能自动保证其原子性的，比如跨总线宽度、跨多个缓存行和跨页表的访问。但是，处理器提供**总线锁定和缓存锁定两个机制**来保证复杂内存操作的原子性。

1 使用总线锁保证原子性

一句话：**总线就是连接各个部件的信息传输线，是各个部件共享的传输介质，同一时间只负责传输一个比特**。

总线说完了后，来说说总线锁。

总线锁就是使用处理器提供的一个 LOCK＃信号，当其中一个处理器在总线上输出此信号时，其它处理器的请求将被阻塞住，那么该处理器可以独占共享内存。

说到这里，大家也明白了总线锁定。但总线锁定的开销太大，因为总线锁定期间，其它处理器不能操作其它内存地址的数据。所以，下面会说到使用缓存锁来保证原子性。

2 使用缓存锁保证原子性

目前处理器在某些场合下使用缓存锁定代替总线锁定来进行优化

这里所说的缓存是指计算机CPU的高速缓存（L1,L2,L3）。在处理器准备进行处理的时候，一般会将要处理的数据预加载到高速缓存中，频繁使用的内存也会在高速缓存中。处理器处理数据会先进入高速缓存中查找，没有找到再去内存中。高速缓存的命中率很高，基本不需要到内存中查找。



所谓“**缓存锁定**”是指**内存区域如果被缓存在处理器的缓存行中**，**并且在Lock操作期间被锁定**，那么当它执行锁操作回写到内存时，处理器不会在总线上声言LOCK＃信号（总线锁定信号），而是修改内部的内存地址，并允许它的缓存一致性机制来保证操作的原子性，因**为缓存一致性机制会阻止同时修改由两个以上处理器缓存的内存区域数据，当其他处理器回写已被锁定的缓存行的数据时，会使缓存行无效**。



3 缓存锁定不能使用的特殊情况
第一种情况是：当操作的数据不能被缓存在处理器内部，或操作的数据跨多个缓存行 时，则处理器会调用总线锁定。

第二种情况是：有些处理器不支持缓存锁定。

### 13 java如何实现原子操作

在Java中可以通过**锁和循环CAS的方式**来实现原子操作。（CAS通过硬件实现，好像就是利用了处理器实现原子操作）

### 14 ReetrantLock 的内部实现

####  1 ReetrantLock 的Lock方法

 ReetrantLock 的Lock方法，调用了Sync的acquire( )方法，本质是调用了AQS的获取独占锁的方法:acquire( ), Lock方法是通过CAS和AQS进行获取锁的。

具体流程：

1. AQS中的acquire( )调用 acquireQueued ( addWaiter( Node.EXCLUSIVE), arg)  方法，其中addWaiter（）方法采用AQS的方式将当前线程加入阻塞队列。
2. 节点就会开启自旋操作，并观察前驱节点的状态，如果**前驱节点是头节点**，那么**会唤醒当前节点进行同步操作**，但是如果**前驱节点不是同步节点**，则会调用shouldParkAfterFailedAcquire（）**判断当前线程是否需要被挂起**，如果线程需要被挂起，那么就会调用parkAndCheckInterrupt（）方法，将线程挂起，至此， ReetrantLock 的Lock方法锁住了当前线程。

#### 2  ReetrantLock的可中断锁lockInterruptibly()

​		ReetrantLock 的lockInterruptibly方法,调用了sync.acquireInterruptibly(1)方法;本质是调用了AQS的获取独占锁的方法:acquireInterruptibly( ), Lock方法是通过CAS和AQS进行获取锁的.

具体流程：

​		首先会尝试拿锁，要是拿锁失败了，会调用doAcquireInterruptibly（arg）方法，**这个方法与acquireQueued方法逻辑几乎一样，而差别在于检测到线程中断后直接抛出异常**，检测到线程的中断操作后，直接抛出异常，从而中断线程的同步状态请求，移除同步队列，ok~。

#### 3  ReetrantLock的锁的释放

ReentrantLock释放锁是通过它自身的unlock方法，而在unlock方法中同样调用了AQS的release方法:

具体流程：

1. 首先调用release ( ) 方法，通过操控state，对state减去releases，如果state为0那么久释放锁，并且将排他线程设置为null,最后更新state，然后通过unparkSuccessor ( ) 方法唤醒后继节点，并且将节点的state的状态置为0，最终是调用的LockSupport.unpark(s.thread) 方法唤醒的后继节点的。



### 15 ReetrantLock相比synchronized提供的新特性

我们先看看他们的区别：

- synchronized是关键字，是JVM层面的底层啥都帮我们做了，而Lock是一个接口，是JDK层面的有丰富的API。
- synchronized会自动释放锁，而Lock必须手动释放锁。
- synchronized是不可中断的，Lock可以中断也可以不中断。
- 通过Lock可以知道线程有没有拿到锁，而synchronized不能。
- synchronized能锁住方法和代码块，而Lock只能锁住代码块。
- Lock可以使用读锁提高多线程读效率。 
- synchronized是非公平锁，ReentrantLock可以控制是否是公平锁。

在JDK 1.6之后，虚拟机对于synchronized关键字进行整体优化后，在性能上synchronized与ReentrantLock已没有明显差距，因此在使用选择上，需要根据场景而定，大部分情况下我们依然建议是synchronized关键字，原因之一是使用方便语义清晰，二是性能上虚拟机已为我们自动优化。而ReentrantLock提供了多样化的同步特性，如超时获取锁、可以被中断获取锁（synchronized的同步是不能中断的）、**等待唤醒机制的多个条件变量**(Condition)等，因此当我们确实需要使用到这些功能是，可以选择ReentrantLock
————————————————
版权声明：本文为CSDN博主「zejian_」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/javazejian/article/details/75043422

### 16 ReetrantLock如何实现公平锁和非公平锁

- NonfairSync：是ReentrantLock的内部类，继承自Sync，非公平锁的实现类。
- FairSync：是ReentrantLock的内部类，继承自Sync，公平锁的实现类。

- ReentrantLock：实现了Lock接口的，其内部类有Sync、NonfairSync、FairSync，在创建时可以根据fair参数决定创建NonfairSync(默认非公平锁)还是FairSync。

**ReetrantLock中非公平锁**

通过ReetrantLock的构造函数可以实现公平锁和非公平锁。默认false为非公平锁。

具体流程：

1. 先将调用lock( )方法里面的compareAndSetState(0, 1）方法把state方法从0设置为1，**如果返回true**则代表获取同步状态成功，也就是当前线程获取锁成，可操作临界资源，setExclusiveOwnerThread（Thread.currentThread()）方法将当前线程，设置成独占模式，**如果返回false**，则表示已有线程持有该同步状态(其值为1)，获取锁失败。执行 `acquire(1)`方法，该方法是AQS中的方法，它对中断不敏感，即使线程获取同步状态失败，进入同步队列，后续对该线程执行中断操作也不会从同步队列中移出。

2. acquire(1)方法里面会调用tryAcquire（）方法。而tryAcquire（）方法实质是调用AQS里面的nonfairTryAcquire(int acquires)方法，后面就不用说了

   ![image-20210805202901037](D:\1书本笔记\java实战项目\image-20210805202901037.png)



![image-20210805203030736](D:\1书本笔记\java实战项目\image-20210805203030736.png)

如果tryAcquire(arg)返回true，`acquireQueued`自然不会执行，这是最理想的，因为毕竟当前线程已获取到锁。



总结成逻辑流程图：



![image-20210805202508610](D:\1书本笔记\java实战项目\image-20210805202508610.png)

**ReetrantLock中公平锁**

公平锁的获取顺序是完全遵循时间上的FIFO规则，也就是说先请求的线程一定会先获取锁，后来的线程肯定需要排队，这点与前面我们分析非公平锁的`nonfairTryAcquire(int acquires)`方法实现有锁不同，两者最大的区别在于tryAcquire（）方法里，公平锁这CAS自旋之前加了一个判断条件，*先判断同步队列是否存在结点*，

![image-20210805203410413](D:\1书本笔记\java实战项目\image-20210805203410413.png)

​		该方法与nonfairTryAcquire(int acquires)方法唯一的不同是在使用CAS设置尝试设置state值前，调用了hasQueuedPredecessors()判断同步队列是否存在结点，如果存在必须先执行完同步队列中结点的线程，当前线程进入等待状态。**这就是非公平锁与公平锁最大的区别**


### 17  Condition的实现原理

​		Condition的具体实现类是AQS的内部类ConditionObject，前面我们分析过AQS中存在两种队列，一种是同步队列，一种是等待队列，而等待队列就相对于Condition而言的。**注意在使用Condition前必须获得锁**，同时在Condition的等待队列上的结点与前面同步队列的结点是同一个类即Node，**其结点的waitStatus的值为CONDITION**。

Condition的具体实现类是AQS的内部类ConditionObject，前面我们分析过AQS中存在两种队列，一种是同步队列，一种是等待队列，而等待队列就相对于Condition而言的。注意在使用Condition前必须获得锁，同时在Condition的等待队列上的结点与前面同步队列的结点是同一个类即Node，其结点的waitStatus的值为CONDITION。在实现类ConditionObject中有两个结点分别是firstWaiter和lastWaiter，firstWaiter代表等待队列第一个等待结点，lastWaiter代表等待队列最后一个等待结点，如下

 public class ConditionObject implements Condition, java.io.Serializable {
    //等待队列第一个等待结点
    private transient Node firstWaiter;
    //等待队列最后一个等待结点
    private transient Node lastWaiter;
    //省略其他代码.......
}

每个Condition都对应着一个等待队列，也就是说如果一个锁上创建了多个Condition对象，那么也就存在多个等待队列。等待队列是一个FIFO的队列，在队列中每一个节点都包含了一个线程的引用，而该线程就是Condition对象上等待的线程。当一个线程调用了await()相关的方法，那么该线程将会释放锁，并构建一个Node节点封装当前线程的相关信息加入到等待队列中进行等待，直到被唤醒、中断、超时才从队列中移出。Condition中的等待队列模型如下

![image-20210805210823416](D:\1书本笔记\java实战项目\image-20210805210823416.png)

正如图所示，Node节点的数据结构，在等待队列中使用的变量与同步队列是不同的，Condtion中等待队列的结点只有直接指向的后继结点并没有指明前驱结点，而且使用的变量是nextWaiter而不是next，这点我们在前面分析结点Node的数据结构时讲过。firstWaiter指向等待队列的头结点，lastWaiter指向等待队列的尾结点，等待队列中结点的状态只有两种即CANCELLED和CONDITION，前者表示线程已结束需要从等待队列中移除，后者表示条件结点等待被唤醒。再次强调每个Codition对象对于一个等待队列，也就是说AQS中只能存在一个同步队列，但可拥有多个等待队列。下面从代码层面看看被调用await()方法(其他await()实现原理类似)的线程是如何加入等待队列的，而又是如何从等待队列中被唤醒的

public final void await() throws InterruptedException {
      //判断线程是否被中断
      if (Thread.interrupted())
          throw new InterruptedException();
      //创建新结点加入等待队列并返回
      Node node = addConditionWaiter();
      //释放当前线程锁即释放同步状态
      int savedState = fullyRelease(node);
      int interruptMode = 0;
      //判断结点是否同步队列(SyncQueue)中,即是否被唤醒
      while (!isOnSyncQueue(node)) {
          //挂起线程
          LockSupport.park(this);
          //判断是否被中断唤醒，如果是退出循环。
          if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
              break;
      }
      //被唤醒后执行自旋操作争取获得锁，同时判断线程是否被中断
      if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
          interruptMode = REINTERRUPT;
       // clean up if cancelled
      if (node.nextWaiter != null) 
          //清理等待队列中不为CONDITION状态的结点
          unlinkCancelledWaiters();
      if (interruptMode != 0)
          reportInterruptAfterWait(interruptMode);
  }
执行addConditionWaiter()添加到等待队列。

 private Node addConditionWaiter() {
    Node t = lastWaiter;
      // 判断是否为结束状态的结点并移除
      if (t != null && t.waitStatus != Node.CONDITION) {
          unlinkCancelledWaiters();
          t = lastWaiter;
      }
      //创建新结点状态为CONDITION
      Node node = new Node(Thread.currentThread(), Node.CONDITION);
      //加入等待队列
      if (t == null)
          firstWaiter = node;
      else
          t.nextWaiter = node;
      lastWaiter = node;
      return node;
        }
**await()方法主要做了3件事，一是调用addConditionWaiter()方法将当前线程封装成node结点加入等待队列，二是调用fullyRelease(node)方法释放同步状态并唤醒后继结点的线程。三是调用isOnSyncQueue(node)方法判断结点是否在同步队列中，注意是个while循环，如果同步队列中没有该结点就直接挂起该线程，需要明白的是如果线程被唤醒后就调用acquireQueued(node, savedState)执行自旋操作争取锁，即当前线程结点从等待队列转移到同步队列并开始努力获取锁。**

2 接着看看唤醒操作**singal()方法**

 public final void signal() {
     //判断是否持有独占锁，如果不是抛出异常
   if (!isHeldExclusively())
          throw new IllegalMonitorStateException();
      Node first = firstWaiter;
      //唤醒等待队列第一个结点的线程
      if (first != null)
          doSignal(first);
 }
这里signal()方法做了两件事，**一是判断当前线程是否持有独占锁，没有就抛出异常**，从这点也可以看出只有独占模式先采用等待队列，而共享模式下是没有等待队列的，也就没法使用Condition。二是**唤醒等待队列的第一个结点，即执行doSignal(first)**

 private void doSignal(Node first) {
     do {
             //移除条件等待队列中的第一个结点，
             //如果后继结点为null，那么说没有其他结点将尾结点也设置为null
            if ( (firstWaiter = first.nextWaiter) == null)
                 lastWaiter = null;
             first.nextWaiter = null;
          //如果被通知节点没有进入到同步队列并且条件等待队列还有不为空的节点，则继续循环通知后续结点
         } while (!transferForSignal(first) &&
                  (first = firstWaiter) != null);
        }

//transferForSignal方法
final boolean transferForSignal(Node node) {
    //尝试设置唤醒结点的waitStatus为0，即初始化状态
    //如果设置失败，说明当期结点node的waitStatus已不为
    //CONDITION状态，那么只能是结束状态了，因此返回false
    //返回doSignal()方法中继续唤醒其他结点的线程，注意这里并
    //不涉及并发问题，所以CAS操作失败只可能是预期值不为CONDITION，
    //而不是多线程设置导致预期值变化，毕竟操作该方法的线程是持有锁的。
    if (!compareAndSetWaitStatus(node, Node.CONDITION, 0))
         return false;

        //加入同步队列并返回前驱结点p
        Node p = enq(node);
        int ws = p.waitStatus;
        //判断前驱结点是否为结束结点(CANCELLED=1)或者在设置
        //前驱节点状态为Node.SIGNAL状态失败时，唤醒被通知节点代表的线程
        if (ws > 0 || !compareAndSetWaitStatus(p, ws, Node.SIGNAL))
            //唤醒node结点的线程
            LockSupport.unpark(node.thread);
        return true;
    }

注释说得很明白了，这里我们简单整体说明一下，doSignal(first)方法中做了两件事，从条件等待队列移除被唤醒的节点，然后重新维护条件等待队列的firstWaiter和lastWaiter的指向。二是将从等待队列移除的结点加入同步队列(在transferForSignal()方法中完成的)，如果进入到同步队列失败并且条件等待队列还有不为空的节点，则继续循环唤醒后续其他结点的线程。到此整个signal()的唤醒过程就很清晰了，即signal()被调用后，先判断当前线程是否持有独占锁，如果有，那么唤醒当前Condition对象中等待队列的第一个结点的线程，并从等待队列中移除该结点，移动到同步队列中，如果加入同步队列失败，那么继续循环唤醒等待队列中的其他结点的线程，如果成功加入同步队列，那么如果其前驱结点是否已结束或者设置前驱节点状态为Node.SIGNAL状态失败，则通过LockSupport.unpark()唤醒被通知节点代表的线程，到此signal()任务完成，注意被唤醒后的线程，将从前面的await()方法中的while循环中退出，因为此时该线程的结点已在同步队列中，那么while (!isOnSyncQueue(node))将不在符合循环条件，进而调用AQS的acquireQueued()方法加入获取同步状态的竞争中，这就是等待唤醒机制的整个流程实现原理，流程如下图所示（注意无论是同步队列还是等待队列使用的Node数据结构都是同一个，不过是使用的内部变量不同罢了）

![image-20210805210736541](D:\1书本笔记\java实战项目\image-20210805210736541.png)

