# Java

### 结合JMM内存模型聊volatile关键字

```
内存模型



使用场景

多线程读取共享变量

双重校验锁



Java内存模型，规定了所有变量的值都在主存中，每个线程都有自己的工作内存，线程对变量的操作都必须在工作内存中完成，不能直接对主存进行操作，

而且每个线程不能访问其他线程的工作内存



线程对工作内存和主存的操作模型 

read（从主存读取）

load（将主存读取到的值写入工作内存）

use（从工作内存读取数据来计算）

assign（将计算好的值重新赋值到工作内存中）

store（将工作内存数据写入主存）

write（将store过去的变量值赋值给主存中的变量）



为什么可以保证可见性，场景 多线程共享变量 flag



volatile flag = false;



// 线程1，先执行

flag=true // 已经变成true了 



// 线程2，后执行

// 读取到的还是false

while(!flag) {

// 无限循环

}



加了volatile关键词，保证了每个线程对一个变量的修改，立即都是对其他线程是可见，

只要线程1一旦修改了flag = true，其他线程会看到最新的值，while循环就会立即跳出



为什么保证不了原子性，场景 i++



如果某一时刻，两个线程都先后将这个i的值，i = 0加载到工作内存里了，可能线程1执行到use步骤了，

此时准备加1；同时线程2瞬间就完成了use、assign、store、write等步骤，i = 1，同时写回了主存，并且让线程1的工作内存缓存行失效

线程1此时不需要工作内存了，之前use已经从工作内存读取了数据，所以也同时将i加1变成了1，然后接着assign、store、write回了主存，相当于写了两次1回主存。



操作系统内存模型 缓存一致性协议（MSI）的理解， 作为高级考察点



某个cpu写数据的时候，发现写的是共享变量，会通知其他cpu这个数据在他们内部的高速缓存是无效的。
```



### java对象从创建到销毁的流程

```
举个例子，Object o= new Object() 的创建过程是怎么样的呢？


初级：
对象的内存怎么过程：
先判断栈空间能不能放入对象，如果可以直接放栈空间，用完直接出栈结束。
栈空间不够大的情况，再判Eden空间够不够大，如不够，直接放老年代，最后通过FGC 垃圾回收。


高级
Eden如果够大，再判断TLAB（线程本地缓存）是否够大，够大放TLAB，通过YGC垃圾回收。
TLAB不够大，那就放Eden区，通过YGC垃圾回收。


高级
实际TLAB也是放在Eden区,只不过TLAB是线程专属的，性能更好，优先使用。

```



### string 内存分配

```
定义String的方法：


　　1,String str1 = "hello";


　　2,String str2 = new String("hello");


　　第一种方法：引用str1被存放在栈区，字符串常量"hello"被存放在常量池，引用str1指向了常量池中的"hello"(str1中的存放了常量池中"hello"的地址)。


　　第二种方法：引用str2被存放在栈区，同时在堆区开辟一块内存用于存放一个新的String类型对象。（同上，str2指向了堆区新开辟的String类型的对象）

```



### 字符串常用的拼接方法有哪些

```
1. 使用 +，编译时会转换成StringBuilder
2. 使用StringBuilder, for循环拼接时建议使用，因为+不能复用StringBuilder对象
3. StringBuffer，比StringBuilder多加了synchronized来保证线程安全
4. StringJoiner，底层复用StringBuilder，优势在于支持前后缀及分隔符，且没有多次array copy

```



### number常量池

```
举个例子，Long a = 100L;
        Long b = 100L;
        System.out.println(a == b);
        Long c = 1000L;
        Long d = 1000L;
        System.out.println(c == d);
答案：true,false; reason:-128~127

```



### Long  a = null ， if (a == 0L) 是否有问题

```
会发生NPE
```



### Java堆内堆外内存区域划分

```
场景


根据候选人基本经验


问下每个区域大概的参数配置
调优过的参数 如果有过调优经验（可以问下原理）


JVM内存主要分为堆、程序计数器、方法区、虚拟机栈和本地方法栈等
1.虚拟机栈：是线程运行Java所需的数据、指令、返回地址
2.本地方法栈：本地方法栈跟Java虚拟机栈的功能类似，Java虚拟机栈用于管理Java函数的调用，而本地方法栈则用于管理本地方法的调用。但本地方法并不是用Java实现的，而是由C语言实现的(比如Object.hashcode方法)。
3.程序计数器 较小的内存空间，当前线程执行的字节码的行号指示器；各线程之间独立存储，互不影响，主要用来记录各个线程执行的字节码的地址例如，分支、循环、跳转、异常、线程恢复等都依赖于计数器。由于Java是多线程语言，当执行的线程数量超过CPU核数时，线程之间会根据时间片轮询争夺CPU资源。如果一个线程的时间片用完了，或者是其它原因导致这个线程的CPU资源被提前抢夺，那么这个退出的线程就需要单独的一个程序计数器，来记录下一条运行的指令。
4.方法区（MethodArea）是可供各条线程共享的运行时内存区域。它存储了每一个类的结构信息，例如运行时常量池（RuntimeConstantPool）字段和方法数据、构造函数和普通方法的字节码内容、还包括一些在类、实例、接口初始化时用到的特殊方法
5.元空间 方法区与堆空间类似，也是一个共享内存区，所以方法区是线程共享的
堆外内存：没有被虚拟机化的操作系统上的其他内存，也就是没有被jvm虚拟机直接使用的内存

```



### java内存各区域调优

```
在内存紧张的机器上


栈
-Xss 栈大小，默认1M。一般256k足够。


问题：
1、新创建的线程从哪里申请内存？堆吗？
答：线程从物理机申请内存，是非堆
2、怎么可以创建更多的线程？
答：减小Xss、减小Xmx或者增加机器内存（还有其他，例如减小堆外、code cache metaspace等）


堆 
-Xms
初始堆内存大小，默认物理内存64/1
-Xmx
最大堆内存，默认物理内存4/1
-Xmn
新生代大小
-XX:NewRatio=2 
老年代的占比，2代表老年代：新生代=2:1
-XX:SurvivorRatio=8
新生代中Eden区域和Survivor区域（From幸存区或To幸存区）的比例，默认为8，代表eden：survivor1:Survivor2 = 8:1:1
-XX:MaxTenuringThreshold
晋升老年代的年龄阈值，默认15
-XX:PretenureSizeThreshold
大对象进入老年代的阈值
-XX:CMSInitiatingOccupancyFraction
老年代占用的阈值，触发并发收集


问题：
1、为什么一般将Xmx和Xms设置成一样的值？
答：heap在扩容和缩容时，会触发gc。例如：在项目启动时可能会频繁gc
2、什么情况下对象会晋升到老年代
答：大对象、年龄限制、1～n年龄的对象达到50%，n以上的晋升老年代
3、为什么要将大对象直接放入老年代？
答：避免大对象在新生代，屡次躲过GC，还得把他们来复制来复制去的，最后才进入老年代。minor gc会STW，复制算法，大对象耗时
4、什么场景需要调整SurvivorRatio？
答：有些对象在短时间内会继续存活，但是也不会太久，一般是增大survivor区域，避免对象由于（1～n年龄的对象达到50%），被晋升老年代，引起频繁的fullgc
5、如果98%的时间发生在STW上，且回收对象不到堆的2%，会怎么样？
答：OOM，可以-XX:-UseGCOverheadLimit禁用此功能
6、Concurrent Mode Failure 的原因是什么？ 后果是什么？怎么优化？


答：原因 老年代正在清理的过程中，又有新对象进入老年代，此时老年代没有空间容纳，
实操中，-XX:CMSInitiatingOccupancyFraction 比例参数设置过大，或者没有开启full gc压缩 或者年轻代对象晋升老年代过快。
触发serial collector，串行收集。STW。
调整垃圾回收时机（例如：CMSInitiatingOccupancyFraction），增加老年代（没有内存泄漏的话）


7、修改CMSInitiatingOccupancyFraction的原因是什么？
答：调整过小会导致频繁fullgc，太大会导致serial collector




问： yonggc stw是否是stw，对系统影响为什么比 fullgc 影响小
答：todo




元空间（存放类和方法的元数据、字节码）
-XX:MetaspaceSize
-XX:MaxMetaspaceSize
元数据空间大小
-XX:CompressedClassSpaceSize
压缩对象指针空间


问题：
1、-XX:MetaspaceSize、-XX:MaxMetaspaceSize为什么一般要设置同样大小？
答：扩容会触发fullgc
2、怎么设置一个相对合理的值？
答：通过项目运行稳定后，查看真实占用（此时：-XX:MetaspaceSize、-XX:MaxMetaspaceSize设置是不一样的）
3、CompressedClassSpaceSize的作用？
答：设置压缩对象指针的空间的大小，这块默认1G，一般占用较小，可以优化
4、怎么触发metaspace的OOM？
答：Cglib动态代理，动态产生新的类




code cache（jit 编译的机器码）
-XX:ReservedCodeCacheSize
-XX:InitialCodeCacheSize
如果code cache满的话，会导致jit关闭，系统性能急剧下降






堆外
-XX:MaxDirectMemorySize
可结合netty说下


malloc
64M内存问题 附链接 todo

```



### 优化young gc/old gc次数

```
优化young gc/old gc次数
代码层面：减少创建对象数量，减少日志输出，使用StringBuilder或StringBuffer来代替String
将进入老年代的对象数量降到最低：老年代GC相对来说会比新生代GC更耗时，因此，减少进入老年代的对象数量可以显著降低Full GC的频率
减少Full GC的执行时间：Full GC的执行时间比Minor GC要长很多，因此，如果在Full GC上花费过多的时间(超过1s)，将可能出现超时错误
GC优化的过程
1.监控GC状态
2.分析监控结果后决定是否需要优化GC如果分析结果显示运行GC的时间只有0.1-0.3秒，那么就不需要把时间浪费在GC优化上，但如果运行GC的时间达到1-3秒，甚至大于10秒，那么GC优化将是很有必要的。
3.设置GC类型/内存大小

```



### 优化young gc 实际场景

```
职位列表在有redis缓存的情况下，依然young gc频繁
```



### 优化old gc 实际场景

```
以下示例存在的两处问题：
1、出生年的计算，如果age是null会发生NPE
2、循环查询新注册用户信息后，追加到userVOList，如果新用户是持续增加或现有已经注册新用户足够多，会导致OOM
3、可以再问下怎么优化这块代码

public List<UserVO> test1() {
        List<UserVO> userVOList = Lists.newArrayList();


        while(true) {


            // 从db中读取新注册的用户信息
            List<UserVO> newUserList = userMapper.selectNewUser();


            if (newUserList.size() == 0) {
                break;
            }


            for(UserVO userVO : newUserList) {
                userVO.setBirthYear(new Date().getYear() - userVO.getAge());
            }
            
            userVOList.addAll(newUserList);
        }


        return userVOList;


    }


    @Data
    public static class UserVO {
        private String name;
        private Integer age;
        private Integer birthYear;
    }

```



### 后端java开发中，常见old 区持续增长场景

```
让候选人举例


持续升高的原因是：存在大量回收不掉的对象。如：静态变量类型为集合，不断往集合中添加元素，这种对象又回收不了。再比如：一次性从DB中取出大批量的对象，再逐个处理，短时间处理不完，导致这批对象一直滞留在内存中，无法回收。
解决方案：减少对象的生命周期，减小一次性从DB中查出记录的条数。慎重使用全局静态变量，使其占用空间控制在可以容忍的范围内。

```



### 栈中存放哪些变量，栈可以存储对象吗

```
存放基本类型的变量数据和对象的引用，但对象本身不存放在栈中，而是存放在堆（new 出来的对象）或者常量池中(对象可能在常量池里)（字符串常量对象存放在常量池中。）


栈中不可以存放对象
```



### 垃圾回收算法以及垃圾回收器有哪些？应用场景？

```
问下候选人实际使用的垃圾回收器
跟其他垃圾回收器的区别


垃圾收集器
1.Serial收集器：是单线程的收集器。它在进行垃圾收集时，必须暂停其他所有的工作线程，直到它收集完成。Serial收集器依然是虚拟机运行在Client模式下默认新生代收集器，对于运行在Client模式下的虚拟机来说是一个很好的选择
2.ParNew收集器：ParNew收集器其实就是Serial收集器的多线程版本，ParNew收集器是许多运行在Server模式下的虚拟机中首选新生代收集器，其中有一个与性能无关但很重要的原因是，除Serial收集器之外，目前只有ParNew它能与CMS收集器配合工作
3.Parallel Scavenge（并行回收）收集器：Parallel Scavenge收集器是一个新生代收集器，它也是使用复制算法的收集器，又是并行的多线程收集器该收集器的目标是达到一个可控制的吞吐量，高吞吐量则可用高效率地利用CPU时间，尽快完成程序的运算任务，主要适合在后台运算而不需要太多交互的任务。
4.Serial Old 收集器 Serial Old是Serial收集器的老年代版本，它同样是一个单线程收集器，使用标记整理算法。这个收集器的主要意义也是在于给Client模式下的虚拟机使用。
5.Parallel Old 收集器Parallel Old 是Parallel Scavenge收集器的老年代版本，使用多线程和“标记-整理”算法。
6.CMS收集器 CMS(Concurrent Mark Sweep)收集器是一种以获取最短回收停顿时间为目标的收集器。目前很大一部分的Java应用集中在互联网站或者B/S系统的服务端上，这类应用尤其重视服务器的响应速度，希望系统停顿时间最短，以给用户带来较好的体验。CMS收集器就非常符合这类应用的需求。 CMS收集器是基于“标记-清除”算法实现的
CMS收集器主要优点：并发收集，低停顿。
7. G1收集器：G1收集器的优势： （1）并行与并发 （2）分代收集 （3）空间整理 （标记整理算法，复制算法）
（4）可预测的停顿（G1处处理追求低停顿外，还能建立可预测的停顿时间模型，能让使用者明确指定在一个长度为M毫秒的时间片段内，消耗在垃圾收集上的时间不得超过N毫秒


垃圾回收算法
1）复制算法
将内存分为（大小相等）两部分，每次只使用其中一块进行内存分配，当内存使用完后，就出发GC，将存活的对象直接复制到另一块空闲的内存中，然后对当前使用的内存块一次性清除所有，然后转到另一块内存进行使用。
优点：简单，高效。
缺点：浪费内存，因为每次都有另一块内存空闲着。
2）标记-清除算法
分两步进行，第一步标记出可以回收的对象，第二步统一清理可以回收的对象内存。
缺点：首先标记和清除步骤效率都不高，其次会产生内存碎片。
3）标记-整理算法
类似于标记-清除算法，但是第二步进行内存回收时，将存活的对象向内存一端移动，达到消除内存碎片问题。
4）分代收集算法
java sun hotspot虚拟机将内存分为新生代（堆）、老年代（堆）、永久代（方法区、常量池、即时编译代码）几个区域，新生代主要使用基于复制算法的垃圾回收，老年代和永久代主要使用标记-整理算法进行垃圾回收。具体每个区域使用哪种垃圾回收算法还要视收集器的实现制约。

```



### 垃圾对象识别算法有几种？

```
引用计数，可达性分析
```



### Java垃圾对象识别算法为什么没有用引用计数？

```
循环引用问题，对象A引用了对象B，B也引用了A，但是除此之外，再没有其他的地方引用这两个对象了，这两个对象的引用计数就都是1。这种情况下，这两个对象是不能被回收的。
```



### GC ROOT 有哪些？

```
运行时数据区的GC ROOT 
1.虚拟机栈（栈帧中的本地变量表）中引用的对象；各个线程调用方法堆栈中使用到的参数、局部变量等
2.方法区中类静态属性引用的对象，java 类的引用类型静态变量
3.方法区中常量引用的对象，例如：字符串常量池里的引用
4.本地方法栈中 JNI引用的对象。
另外，还有JVM 的内部引用（class 对象、异常对象 NullPointException、OutofMemoryError，系统类加载器），所有被同步锁(synchronized )持有的对象等

```



### 内存泄露场景？

```
threadlocal导致内存泄漏
```



### 类加载过程

```
加载：把class字节码文件从各个来源通过类加载器装载入内存中。
	类加载器（启动类加载器，扩展类加载器，应用类加载器，以及用户的自定义类加载器。）
链接：验证 准备 解析
验证：保证加载进来的字节流符合虚拟机规范，不会造成安全错误。（文件格式，元数据，字节码，符号引用）
准备：为类变量（注意，不是实例变量）分配内存，并且赋予初值。（比如8种基本类型的初值，默认为0；引用类型的初值则为null；常量的初值即为代码中设置的值）
解析：将常量池内的符号引用替换为直接引用的过程。


初始化：对类变量初始化，是执行类构造器的过程。（对static修饰的变量或语句进行初始化。）
```



### STW触发时机

```
1 2 4 6 属于常用场景
2 6 中级


1.定时进入 SafePoint：每经过-XX:GuaranteedSafepointInterval 配置的时间，都会让所有线程进入 Safepoint，一旦所有线程都进入，立刻从 Safepoint 恢复。这个定时主要是为了一些没必要立刻 Stop the world 的任务执行，可以设置-XX:GuaranteedSafepointInterval=0关闭这个定时，我推荐是关闭。
2.由于 jstack，jmap 和 jstat 等命令，也就是 Signal Dispatcher 线程要处理的大部分命令，都会导致 Stop the world：这种命令都需要采集堆栈信息，所以需要所有线程进入 Safepoint 并暂停。
3.偏向锁取消（这个不一定会引发整体的 Stop the world，参考JEP 312: Thread-Local Handshakes）：Java 认为，锁大部分情况是没有竞争的（某个同步块大多数情况都不会出现多线程同时竞争锁），所以可以通过偏向来提高性能。即在无竞争时，之前获得锁的线程再次获得锁时，会判断是否偏向锁指向我，那么该线程将不用再次获得锁，直接就可以进入同步块。但是高并发的情况下，偏向锁会经常失效，导致需要取消偏向锁，取消偏向锁的时候，需要 Stop the world，因为要获取每个线程使用锁的状态以及运行状态。
4.Java Instrument 导致的 Agent 加载以及类的重定义：由于涉及到类重定义，需要修改栈上和这个类相关的信息，所以需要 Stop the world
5.Java Code Cache相关：当发生 JIT 编译优化或者去优化，需要 OSR 或者 Bailout 或者清理代码缓存的时候，由于需要读取线程执行的方法以及改变线程执行的方法，所以需要 Stop the world
6.GC：这个由于需要每个线程的对象使用信息，以及回收一些对象，释放某些堆内存或者直接内存，所以需要 Stop the world

```



### oom场景

```
OOM场景


分区问 


1、java.lang.OutOfMemoryError: Metaspace
Cglib 不断创建新类
场景：
a、Spring 的 GCLib 实现动态创建对象
B、代理性框架基本都有这个问题
C、大量 JSP 或动态产生 JSP  文件的应用
2、java.lang.OutOfMemoryError: Java heap space
堆内对象无法回收
场景：
A、一次性取出全部待处理的任务（任务很多）
B、类似uc项目，大量获取config配置，导致gc过高
C、垃圾产生过快，堆设置太小
3、java.lang.OutOfMemoryError: unable to create new native thread
机器内存不足
可问：
A、在硬件条件不变，怎么创建更多的线程数量？
答：两方面，第一：减小线程内存占用，第二：腾出更多的空间，基本就是调整其他参数
4、java.lang.OutOfMemoryError: GC overhead limit exceeded（与2类似）
98%的时间STW，垃圾回收不到2%
5、java.lang.OutOfMemoryError: Direct buffer memory
NIO，使用直接内存溢出。
6、java.lang.StackOverflowError
栈溢出，递归太深。
7、OOM Killer （系统内存不足，导致应用被强制kill）
升机器内存；减少应用实际占用
8、java.lang.OutOfMemoryError: Requested array size exceeds VM limit（创建超长数组，超出jvm限制）
一般没这么干的

```