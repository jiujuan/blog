> 对锁的一些要素简单罗列，每一项深入研究都有浩繁的知识。不断完善ing...

#### 标题：锁到底是什么

目录：

1. 进程间通信

2. 锁是什么

2. 硬件

3. 软件

4. 锁分类 锁粒度

5. CPU cache

6. 编译器 和 内存

   

标题： **锁到底是什么**

由于有时钟中断，以及出现多个进程，多个进程操作同一个数据，就有可能使数据变成脏数据。为了解决脏数据的问题，就出现了锁这种机制，保护数据。
后来出现的多核cpu，进一步给操作共享数据增加难度。



## 进程间通信（inter Process Communication，IPC）


### 竞争条件（竞态条件）

多个进程（线程）通过共享内存（或共享文件）的方式进行通信就会出现竞争条件。
为了避免这种`竞态条件`的出现，就需要找出存在这种`竞态条件`的程序片段，通过`互斥`的手段来阻止多个进程同时读写共享的数据。

实现进程间通信手段：
一般会想到的手段，

- **屏蔽中断**

- **锁变量**

1. 互斥锁

2. 忙等待互斥（自旋等待）

3. 读写锁

4. 信号量

   

### 临界区

从代码角度来说：把代码分为导致竞争的条件程序和不会导致竞争的程序片段。
会导致竞争的程序片段叫临界区。避免竞争条件只需要组织多个进程同时读写共享的数据就可以了，也就是保证同时只有一个进程处于临界区。



## 锁是什么

锁就是保证只有一个进程处于临界区的一种机制。



## 硬件

### 单处理器：

临界区问题好解决，修改共享变量时禁止中断出现。但是屏蔽中断后，时钟中断也会被屏蔽。相当于这个时候我们把CPU交给了这个进程，他不会因为CPU时钟走完而切换，如果其不再打开中断，后果将是非常可怕的。总之，把屏蔽中断的权力交给用户级进程是很不明智的。


### 多处理器：

屏蔽中断只会对执行了中断指令的那个CPU有效，但是其他CPU还是有可能会改变共享内存中的数据的。


### 硬件原子操作：

许多现代计算机系统都提供了特殊硬件指令以允许能原子地（不可中断地）检查和修改字的内容或交换两个字的内容（比如CAS，compare and swap）。其实很多锁的软件方案都是通过调用原子操作来实现的。

基于硬件指令一般是基于冲突检测的乐观并发策略：先进行操作，如果没有其他进程争用共享数据，操作就成功了，如果产生了冲突，就进行补偿，不断重试。

乐观并发策略需要硬件指令集的发展才能进行，需要硬件指令实现：操作+冲突检的原子性。

这类指令有：

- 测试并设置锁 Test and Set Lock  (TSL)
- 获取并增加 Fetch-and-Incrementsuo
- 交换 Swap<br />
- 比较并交换 Compare-and-Swap (CAS)
- 加载链接/条件存储 Load-linked / Store-Conditional  LL/SC


链接：[https://juejin.im/post/6860125321149022221/](https://juejin.im/post/6860125321149022221/)

对上面一个发展历程解释：
原先计算机的原子操作都是通过硬件实现的 , X86架构中的指令 LOCK，这个是锁住总线，从而阻止其他CPU访问。可想而知，这种实现方式是非常低效的。从PentinumPro开始，LOCK 只会阻塞其他cpu对相关内存缓存块的访问。

现在，大多数X86处理器都支持CAS的硬件实现，保证了多处理器多核系统下的原子操作的正确性。CAS的实现同样无需锁住总线，只会阻塞其他cpu核对相关内存的缓存块的访问。

同样的，在MIPS和ARM架构下，还支持了LL/SC的实现[[3]](https://link.zhihu.com/?target=https%3A//en.wikipedia.org/wiki/Load-link/store-conditional)。LL/SC不会出现CAS中的ABA问题，而且基于LL/SC可以实现CAS FAA等原子操作。




## 软件

### 信号量：

信号量是E. W. Dijkstra在1965年提出的一种方法，它使用一个整型变量来累计唤醒次数。

一个信号量的取值可以为0（表示没有保存下来的唤醒次数）或者为正值（表示有一个或者多个唤醒操作）。信号量除了初始化外只能通过两个标准原子操作：wait()和signal()来访问。这些操作原来被称为P（荷兰语Proberen，测试）和V（荷兰语Verhogen，增加）。wait()和signal()的定义可表示如下：

```c
wait(S) {
    while(S<=0)
        ;   //no-op
    S--}

signal(S) {
    S++}
```


### 互斥量：

互斥量可以认为是取值只有0和1的信号量。

我们经常使用就是这个。使用其来同步的代码如下：

```c
do {
    waiting(mutex);
        //critical section
    signal(mutex);
        //remainder section
}while(TRUE)
```

实现。每个信号量关联一个等待进程链表。进程wait()的时候发现信号量不为正时，可以选择忙等待（信号量那儿的例子），也可以选择阻塞自己（下面的实现），进程加入等待链表。signal()时从等待链表中取出进程唤醒。

```c
typedef struct {
    int value;
    struct process *list;} semaphore;

wait(semaphore *S) {
    S->value--;
    if (S->value < 0) {
        add this process to S->list;
        block()
    }}

signal(semphore *S) {
    S->value++;
    if (S->value <=0 ) {
        remove a process P from S->list;
        wakeup(P)
    }}
```



## 锁种类

常见锁：互斥锁、自旋锁、读写锁



## 锁为什么会慢

关于锁竞争慢，可以参考这篇文章:[Locks Aren't Slow; Lock Contention Is](http://preshing.com/20111118/locks-arent-slow-lock-contention-is/)

简短来说，频繁的锁竞争会导致CPU core中的进程上下文切换，同时还有缓存的失效。



## 锁粒度

在各种大会上，我们经常会听到演讲者说锁应该是关联数据的，粒度要尽可能的小。但是实际上粒度过小对于代码的效率真的很有影响。



## cpu cache

为什么需要cpucache：因为cpu的速度太快了，快到主存跟不上，这样在处理器时钟周期内，CPU常常等待主存，浪费资源。所以cache出现了，是为了缓解cpu和内存之间速度不匹配问题（结构：cpu->cache->memory）

CPU cache有什么意义？cache的容量远远小于主存，因此出现cache miss在所难免，既然cache不能包含CPU所需要的所有数据，那么cache的存在真的有意义吗？当然是有意义的——局部性原理。

 A. 时间局部性：如果某个数据被访问，那么在不久的将来它很可能被再次访问；
 B. 空间局部性：如果某个数据被访问，那么与它相邻的数据很快也可能被访问；

单核cpu cache结构：

![image.png](https://cdn.nlark.com/yuque/0/2020/png/127300/1594900482944-da238b0e-f476-4953-854b-a21caa4495cf.png#align=left&display=inline&height=281&margin=%5Bobject%20Object%5D&name=image.png&originHeight=281&originWidth=623&size=13136&status=done&style=none&width=623)


多核cpu cache结构：

![image.png](https://cdn.nlark.com/yuque/0/2020/png/127300/1594900504897-b01c4bac-fe13-4b4a-ad3a-77216f56d49c.png#align=left&display=inline&height=281&margin=%5Bobject%20Object%5D&name=image.png&originHeight=281&originWidth=760&size=15948&status=done&style=none&width=760)

MESI(缓存一致性）： 缓存一致性：在多核CPU中，内存中的数据会在多个核心中存在数据副本，某一个核心发生修改操作，就产生了数据不一致的问题。而一致性协议正是用于保证多个CPU cache之间缓存共享数据的一致。

至于MESI，则是缓存一致性协议中的一个，到底怎么实现，还是得看具体的处理器指令集。

memory barriers 内存屏蔽：性能，因为对内存访问顺序的重排可以获得更好的性能。

cpu结构：
![image.png](https://cdn.nlark.com/yuque/0/2020/png/127300/1594900152494-f905ccab-9a9d-4c32-8275-08abf135ad2c.png#align=left&display=inline&height=461&margin=%5Bobject%20Object%5D&name=image.png&originHeight=461&originWidth=690&size=31360&status=done&style=none&width=690)

![image.png](https://cdn.nlark.com/yuque/0/2020/png/127300/1594900235792-4ae5fe0f-60df-44b0-bfc2-5a08e2af3f88.png#align=left&display=inline&height=570&margin=%5Bobject%20Object%5D&name=image.png&originHeight=570&originWidth=517&size=49520&status=done&style=none&width=517)


# 编译器


> [http://0xffffff.org/2017/02/21/40-atomic-variable-mutex-and-memory-barrier/](http://0xffffff.org/2017/02/21/40-atomic-variable-mutex-and-memory-barrier/)


现代编译器对代码的优化和编译器指令重排可能会影响到代码的执行顺序。

为什么要重排？
编译器指令重排是通过调整代码中的指令顺序，在不改变代码语义前提下，对变量访问进行优化。从而尽可能的减少对寄存器的读取和存储，并充分复用寄存器。

那禁止编译器该类变量优化行不行？解决编译器的重排序不就行了吗， 答案是：不行， cpu还有乱序执行（out-of-order Exection）

流水线（Pipeline）和乱序执行是现代CPU基本都具有的特性。<br />机器指令在流水线中经历取指、译码、执行、访存、写回等操作。为了CPU的执行效率，流水线都是并行处理的，在不影响语义的情况下。

**处理器次序（Process Ordering，机器指令在CPU实际执行时的顺序）** 和 **程序次序（Program Ordering，程序代码的逻辑执行顺序）** 是允许不一致的，即满足As-if-Serial特性。

从此单核时代CPU的Self-Consistent特性在多核时代已不存在，多核CPU作为一个整体看，不再满足Self-Consistent特性。

> 简单总结一下，如果不做多余的防护措施，单核时代的无锁环形队列在多核CPU中，一个CPU核心上的Writer写入数据，更新index后。另一个CPU核心上的Reader依靠这个index来判断数据是否写入的方式不一定可靠。index有可能先于数据被写入，从而导致Reader读到脏数据。


所有的麻烦到这里就结束了吗？当然不，还有Cache的问题。前文提到的都是 **顺序一致性（Sequential Consistency）** 的问题，没有涉及 **Cache一致性（Cache Coherence）** 的问题。虽然说一般情况下程序员只需要关注顺序一致性即可，但是区分清楚这两个概念也能更好的解释内存屏障（Memory Barrier）。

开始提到Cache一致性协议之前，先介绍两个名词：

- Load/Read CPU读操作，是指将内存数据加载到寄存器的过程
- Store/Write CPU写操作，是指将寄存器数据写回主存的过程

现代处理器缓存一般分为三季，有每一个核心独享的L1、L2 Cache，以及所有核心共享的L3 Cache 组成：
![image.png](https://cdn.nlark.com/yuque/0/2020/png/127300/1595148852412-fc7d7963-8233-4928-b178-146ac058ea8f.png#align=left&display=inline&height=584&margin=%5Bobject%20Object%5D&name=image.png&originHeight=584&originWidth=595&size=29424&status=done&style=none&width=595)

由于Cache的容量很小，一般都是充分的利用局部性原理，按行/块来和主存进行批量数据交换，以提升数据的访问效率。

既然各个核心之间有独立的Cache存储器，那么这些存储之间的数据同步就是一个比较复杂的事情。缓存的数据一致性由缓存一致性协议保证。

传统的MESI协议中有两个行为的执行成本比较大。一个是将某个Cache Line标记为Invalid状态，另一个是当某Cache Line当前状态为Invalid时写入新的数据。所以CPU通过Store Buffer和Invalidate Queue组件来降低这类操作的延时。如图：

![image.png](https://cdn.nlark.com/yuque/0/2020/png/127300/1595148864403-67219c4b-e40e-4e41-86dd-3d86768827c2.png#align=left&display=inline&height=413&margin=%5Bobject%20Object%5D&name=image.png&originHeight=413&originWidth=448&size=22124&status=done&style=none&width=448)

**编译器优化乱序** 和 **CPU执行乱序** 的问题可以分别使用 **优化屏障 (Optimization Barrier)** 和 **内存屏障 (Memory Barrier)** 这两个机制来解决：

**优化屏障 (Optimization Barrier)** ：
避免编译器的重排序优化操作，保证编译程序时在优化屏障之前的指令不会在优化屏障之后执行。这就保证了编译时期的优化不会影响到实际代码逻辑顺序。

IA-32/AMD64架构上，在Linux下常用的GCC编译器上，优化屏障定义为（linux kernel, include/linux/compiler-gcc.h）：

```c
/* The "volatile" is due to gcc bugs */
#define barrier() __asm__  __volatile__("":  :  :"memory")
```

优化屏障告知编译器：

- 内存信息已经修改，屏障后的寄存器的值必须从内存中重新获取
- 必须按照代码顺序产生汇编代码，不得越过屏障

> C/C的volatile关键字也能起到优化限制的作用，但是和Java中的volatile（Java 5之后）不同，C/C中的volatile不提供任何防止乱序的功能，也并不保证访存的原子性。


**内存屏障 (Memory Barrier)** 分为 **写屏障（Store Barrier）**、**读屏障（Load Barrier）**和**全屏障（Full Barrier）**，其作用有两个：

- 防止指令之间的重排序
- 保证数据的可见性


关于第一点，关于指令重排，这里不考虑架构的话，Load和Store两种操作会有Load-Store、Store-Load、Load-Load、Store-Store这四种可能的乱序结果。 上文提到的三种屏障则是限制这些不同乱序的机制。

关于第二点。写屏障会阻塞直到把Store Buffer中的数据刷到Cache中；读屏障会阻塞直到Invalid Queue中的消息执行完毕。以此来保证核间各级数据的一致性。

这里要强调，**内存屏障**解决的只是**顺序一致性**的问题，**不解决Cache一致性**的问题（这是Cache一致性协议的责任，也不需要程序员关注）。

Store Buffer和Load Buffer等组件是属于流水线的一部分，和Cache无关。这里一定要区分清楚这两点，Cache一致性协议只是保证了Cache一致性（Cache Coherence），但是不关注顺序一致性（Sequential Consistency）的问题。比如，一个处理器对某变量A的写入操作仅比另一个处理器对A的读取操作提前很短的一点时间，那就不一定能确保该读取操作会返回新写入的值。这个新写入的值多久之后能确保被读取操作读取到，这是内存一致性模型（Memory Consistency Models）要讨论的问题。

**完全的确保顺序一致性**需要很大的代价，不仅限制编译器的优化，也限制了CPU的执行效率。为了更好地挖掘硬件的并行能力，现代的CPU多半都是介于两者之间，即所谓的**宽松的内存一致性模型（Relaxed Memory Consistency Models）**。不同的架构在重排上有各自的尺度，在严格排序和自由排序之间会有各自的偏向。偏向严格排序的一边，称之为**强模型（Strong Model）**，而偏向于自由排序的一边，称之为**弱模型（Weak Model）**
