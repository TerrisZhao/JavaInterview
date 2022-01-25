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

### 哪些场景用到了反射 - 函数表达式解析执行uppercase(concat(md5(abc), base64(username)))

```
查看候选人是否能想到反射的实现思路，如果能，继续问实现方式
函数签名匹配，反射执行。反射时注意方法重载和可变参数。返回值字符串处理

```



### hashmap 降低冲突概率的hash算法是什么

```
hashmap  hash算法怎么做的 key.hashCode() --- 很多初级会这么说




public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}


static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}


高16位和低16位进行一个异或运算


后面在用这个hash值定位到数组的index的时候，也有一个位运算,一般都是用低16位在进行运算，
如果不把hash值的高16位和低16位进行运算的话，那么就会导致后面在通过hash值找到数组index的时候，只有hash值的低16位参与了运算


在hash值的低16位里面，可以同时保留他的高16位和低16位的特征,可以降低hash冲突的概率

```



### arraylist 数组扩容原理

场景：

默认数组大小10，已经添加了10个元素

add() 添加一个元素

```
int newCapacity = oldCapacity + (oldCapacity >> 1);
 
oldCapacity = 10
oldCapacity >> 1 = oldCapacity / 2 = 5
newCapacity = 15
 
elementData = Arrays.copyOf(elementData, newCapacity);
 
数组扩容的时候，老的大小 + 老的大小 >> 1（相当于除以2），实现了数组的拷贝

```



### hashmap 扩容机制

```
回答出一种就行


JDK7中的扩容机制
JDK7的扩容机制相对简单，有以下特性：
空参数的构造函数：以默认容量、默认负载因子、默认阈值初始化数组。内部数组是空数组。
有参构造函数：根据参数确定容量、负载因子、阈值等。
第一次put时会初始化数组，其容量变为不小于指定容量的2的幂数。然后根据负载因子确定阈值。
如果不是第一次扩容，则 新容量=就容量X2,新阈值=新容量X负载因子


JDK8的扩容机制
JDK8的扩容做了许多调整。
HashMap的容量变化通常存在以下几种情况：
1.空参数的构造函数：实例化的HashMap默认内部数组是null，即没有实例化。第一次调用put方法时，则会开始第一次初始化扩容，长度为16。
2.有参构造函数：用于指定容量。会根据指定的正整数找到不小于指定容量的2的幂数，将这个数设置赋值给阈值（threshold）。第一次调用put方法时，会将阈值赋值给容量，然后让 阈值=容量X负载因子
（因此并不是我们手动指定了容量就一定不会触发扩容，超过阈值后一样会扩容！！）
3.如果不是第一次扩容，则容量变为原来的2倍，阈值也变为原来的2倍。（容量和阈值都变为原来的2倍时，负载因子还是不变）
此外还有几个细节需要注意：
首次put时，先会触发扩容（算是初始化），然后存入数据，然后判断是否需要扩容；
不是首次put，则不再初始化，直接存入数据，然后判断是否需要扩容；

```



### 为什么gson结合typetoken可以获取到泛型信息

```
gson解析一段JSON数组时，需要借助TypeToken将期望解析成的数据类型传入到fromJson()方法中


List<Person> people = gson.fromJson(jsonData, new TypeToken<List<Person>>(){}.getType());


[{"name":"张三","age":"10"},
{"name":"李四","age":"20"},
{"name":"王五","age":"30"}]


new TypeToken<List<Person>>(){} 是一个匿名内部类


等价于  MyTypeToken<List<Person>> extends TypeToken(){}


JDK里面提供了方法去读取这些泛型信息的方法


Type mySuperClass = xx.getClass().getGenericSuperclass();
 Type type = ((ParameterizedType)mySuperClass).getActualTypeArguments()[0];
System.out.println(type);

```



### 编译器优化的代码例子

```
每个点可以单独问


1、泛型擦除：编译的时候会进行泛型擦除,使用的时候再进行类型转换ArrayList<Integer> 
2、自动装箱与拆箱：Integer a = 128（可问缓存）Double i1 = 100.0 可问区别，比较时候
3、foreach循环遍历：for (int i: ls)编译过后 会变成while(var2.hasNext()) 这也是循环遍历的容器必须实现iterator接口的原因
4、变长参数：method(param...)编译后可看到，内部会有一个数组建立的过程,所以效率会降低
5、int 优化：int a = 10编译后：byte var1 = 10；int b = 128编译后short var2 = 128
6、默认构造器：new 对象时候自动创建
7、栈上分配：对象不会逃逸出方法之外，在栈上分配内存，对象所占用的内存空间就可以随栈帧出栈而销毁
8、同步消除：逃逸分析能够确定一个变量不会逃逸出线程，无法被其他线程访问，变量的读写就不会有竞争，同步可以消除掉
9、标量替换：逃逸分析一个对象不会被外部访问，并且这个对象可以被拆散，执行的时候可能不创建这个对象，而改为直接创建变量

```



### 高阶函数 lamda 表达式 过滤 转换 排序 场景

```
要求：
        1、过滤掉list中的null元素和birthYear为空的元素
        2、根据age进行升序排序
        3、取出user的name属性，并根据name去重
        4、将去重后的name转成一个list返回




UserVO userVO = new UserVO("n1", 1, null);
        UserVO userVO1 = new UserVO("n2", 3, 1999);
        UserVO userVO2 = new UserVO("n3", 2, 2000);
        UserVO userVO3 = new UserVO("n4", 10, 1990);
        UserVO userVO4 = new UserVO("n5", 11, 1992);
        UserVO userVO5 = new UserVO("n6", 10, 1993);
        UserVO userVO6 = new UserVO("n6", 10, 1993);


        List<UserVO> userVOList = Lists.newArrayList(null, userVO, userVO1, userVO2, userVO3, userVO4, userVO5, userVO6);


      
        List<String> nameList = userVOList.stream()
                .filter(user -> user != null && user.getBirthYear() != null)
                .sorted(Comparator.comparing(UserVO::getAge))
                .map(UserVO::getName)
                .distinct()
                .collect(Collectors.toList());

```



### io模式 

```
1.同步阻塞IO Java中提供的标准IO输入输出, 同步IO, 一个数据流即需要一个线程去读取, 此时, 线程将会等待数据流准备时间, 如果未准备好, 则需要一直阻塞, 可以采取线程池的方式对该读取方式进行优化
2.同步非阻塞IO
示例代码如下, 与普通IO最大的区别在于, 当去读取IO数据时, 无论数据有没有准备好, 会立即返回, 线程需要不断的轮询去查看socket是否准备好, 相对于同步阻塞IO, 线程不用等在那里等待IO就绪, 但同时浪费了大量cpu的轮询时间
3.IO多路复用
I/O多路复用是通过一种机制，可以监视多个描述符，一旦某个描述符就绪（一般是读就绪或者写就绪），能够通知程序进行相应的读写操作
Java中NIO中的几个部件: Channel(通道), Buffer(缓冲区), Selector(监视器)
client与服务端通过Channel.Socket 进行连接, 数据传送至 Buffer区域, 当该区域可读时, 监视器捕获该事件, 读取并处理数据(当selector注册为 OP_READ)
与同步非阻塞IO的最大区别在于, 线程不用不断的轮询去查看io是否就绪, 而是改为等待在那里, 当io就绪后, 主动通知线程去处理, 节省了cpu轮询时间; 代码上主要引入了Selector(监视器)
4.AIO(Asynchronous IO) 异步非阻塞IO
主要融合了Java的Future模式, 提供了回调的方式, 当调用阻塞操作时, 启用子线程去处理, 处理完成后, 回调后续的业务操作, 注意, 此时的线程不再是之前主线程, 如果想保持在主线程中执行后续的业务逻辑, 则可以使用 future.get() 方式, 使得主线程阻塞直到任务完成

```



###  扩展-netty 基础

```
1.异步非阻塞通信
在IO编程过程中，当需要同时处理多个客户端接入请求时，可以利用多线程或者IO多路复用技术进行处理。IO多路复用技术通过把多个IO的阻塞复用到同一个select的阻塞上，从而使得系统在单线程的情况下可以同时处理多个客户端请求。与传统的多线程/多进程模型比，I/O多路复用的最大优势是系统开销小，系统不需要创建新的额外进程或者线程，也不需要维护这些进程和线程的运行，降低了系统的维护工作量，节省了系统资源。
Netty的IO线程NioEventLoop由于聚合了多路复用器Selector，可以同时并发处理成百上千个客户端Channel，由于读写操作都是非阻塞的，这就可以充分提升IO线程的运行效率，避免由于频繁IO阻塞导致的线程挂起。另外，由于Netty采用了异步通信模式，一个IO线程可以并发处理N个客户端连接和读写操作，这从根本上解决了传统同步阻塞IO一连接一线程模型，架构的性能、弹性伸缩能力和可靠性都得到了极大的提升。
2.零拷贝
Netty的“零拷贝”主要体现在如下三个方面：
1) Netty的接收和发送ByteBuffer采用DIRECT BUFFERS，使用堆外直接内存进行Socket读写，不需要进行字节缓冲区的二次拷贝。如果使用传统的堆内存（HEAP BUFFERS）进行Socket读写，JVM会将堆内存Buffer拷贝一份到直接内存中，然后才写入Socket中。相比于堆外直接内存，消息在发送过程中多了一次缓冲区的内存拷贝。
2) Netty提供了组合Buffer对象，可以聚合多个ByteBuffer对象，用户可以像操作一个Buffer那样方便的对组合Buffer进行操作，避免了传统通过内存拷贝的方式将几个小Buffer合并成一个大的Buffer。
3) Netty的文件传输采用了transferTo方法，它可以直接将文件缓冲区的数据发送到目标Channel，避免了传统通过循环write方式导致的内存拷贝问题。
3.无锁化的串行设计理念
在大多数场景下，并行多线程处理可以提升系统的并发性能。但是，如果对于共享资源的并发访问处理不当，会带来严重的锁竞争，这最终会导致性能的下降。为了尽可能的避免锁竞争带来的性能损耗，可以通过串行化设计，即消息的处理尽可能在同一个线程内完成，期间不进行线程切换，这样就避免了多线程竞争和同步锁。
为了尽可能提升性能，Netty采用了串行无锁化设计，在IO线程内部进行串行操作，避免多线程竞争导致的性能下降。表面上看，串行化设计似乎CPU利用率不高，并发程度不够。但是，通过调整NIO线程池的线程参数，可以同时启动多个串行化的线程并行运行，这种局部无锁化的串行线程设计相比一个队列-多个工作线程模型性能更优。
Netty的NioEventLoop读取到消息之后，直接调用ChannelPipeline的fireChannelRead(Object msg)，只要用户不主动切换线程，一直会由NioEventLoop调用到用户的Handler，期间不进行线程切换，这种串行化处理方式避免了多线程操作导致的锁的竞争，从性能角度看是最优的。
4.高效的并发编程
Netty的高效并发编程主要体现在如下几点：
1) volatile的大量、正确使用;
2) CAS和原子类的广泛使用；
3) 线程安全容器的使用；
4) 通过读写锁提升并发性能。
5.高效的Reactor线程模型

```



### 懒汉单例模式

```
考察点：
每一步的意义


1、单例模式-懒汉式
2、指令重排序
3、并发


class Singleton{
    private String str;
    private static volatile Singleton singleton;//第二层锁，volatile关键字禁止指令重排
    private Singleton(){
        str="hello";
    }
    public String getStr() {
        return str;
    }
    public static Singleton getInstance(){
        if(singleton==null){//第一层检查，检查是否有引用指向对象，高并发情况下会有多个线程同时进入
            synchronized (Singleton.class){//第一层锁，保证只有一个线程进入
                //双重检查，防止多个线程同时进入第一层检查(因单例模式只允许存在一个对象，故在创建对象之前无引用指向对象，所有线程均可进入第一层检查)
                //当某一线程获得锁创建一个Singleton对象时,即已有引用指向对象，singleton不为空，从而保证只会创建一个对象
                //假设没有第二层检查，那么第一个线程创建完对象释放锁后，后面进入对象也会创建对象，会产生多个对象
                if(singleton==null){//第二层检查
                    //volatile关键字作用为禁止指令重排，保证返回Singleton对象一定在创建对象后
                    singleton=new Singleton();
                    //singleton=new Singleton语句为非原子性，实际上会执行以下内容：
                    //(1)在堆上开辟空间；(2)属性初始化;(3)引用指向对象
                    //假设以上三个内容为三条单独指令，因指令重排可能会导致执行顺序为1->3->2(正常为1->2->3),当单例模式中存在普通变量需要在构造方法中进行初始化操作时，单线程情况下，顺序重排没有影响；但在多线程情况下，假如线程1执行singleton=new Singleton()语句时先1再3，由于系统调度线程2的原因没来得及执行步骤2，但此时已有引用指向对象也就是singleton!=null，故线程2在第一次检查时不满足条件直接返回singleton，此时singleton为null(即str值为null)
                    //volatile关键字可保证singleton=new Singleton()语句执行顺序为123，因其为非原子性依旧可能存在系统调度问题(即执行步骤时被打断)，但能确保的是只要singleton!=0，就表明一定执行了属性初始化操作；而若在步骤3之前被打断，此时singleton依旧为null，其他线程可进入第一层检查向下执行创建对象
                }
            }
        }
        return singleton;
    }
}

```



### 线程池 - 常用的线程池有哪些，FixedThreadPool，CachedThreadPool 使用场景 

```
 1 SingleThreadExecutor 
 2 FixedThreadExecutor 
 3 CachedThreadExecutor 
 4 ScheduledThreadExecutor


说下优缺点 


FixedThreadPool 负载比较重，而且负载比较稳定 
CachedThreadPool 每天大部分时间负载比较低，少量线程可以处理低负载，每天有少量的高峰期，一次创建几百个线程， 高峰期过去后，空闲线程被自动回收，避免给系统带来过大负载

```



### 线程池原理

具体场景  corePoolSize：2个 mamximumPoolSize：4个 keepAliveTime：30s workQueue：ArrayBlockingQueue，有界阻塞队列，队列大小是5 handler：默认的策略，抛出来一个ThreadPoolRejectException

```
1 开始线程池里的线程空的，有一个变量维护线程数量，poolSize 为 0， 如果当前线程数量小于 corePoolSize（2） 来一个任务优先创建线程，直到线程池里的线程数量跟 corePoolSize数量一样  
2.如果当前线程的线程数量poolSize=2 大于等于corePoolSize（2），而且任务队列没满（最大5，当前元素数量为0），放到任务队列中 
3.如果当前线程池的线程数量大于等于 corePoolSize，而且任务队列满了（最大5，当前放了5个元素，满了），如果当前线程数量小于最大线程数 corePoolSize=2  mamximumPoolSize 4 ，poolSize < maximumPoolSize， 继续创建线程， 提交一个任务 poolSize >= corePoolSize ，任务队列满， poolSize < maximumPoolSize, 再次创建一个任务 
4.如果poolSize >= corePoolSize, 任务队列满，poolSize == maximumPoolSize，当前线程达到了最大线程数，使用handler处理， 默认抛出异常 ThreadPoolRejectExeception 
5 任务执行完，线程池里4个线程，都处于空闲状态，此时超过了corePoolSize的2个线程，如果超出的2个线程空闲时间超过了30s，线程池会回收掉超出的2个线程


maximumPoolSize 回收到 poolSize 原理


take 超时时间 设置 idle线程释放值同一个


高级题 ---- 
如何复用线程
核心原理在于线程池对 Thread 进行了封装，并不是每次执行任务都会调用 Thread.start() 来创建新线程，而是让每个线程去执行一个“循环任务”，在这个“循环任务”中，不停地检查是否还有任务等待被执行，如果有则直接去执行这个任务，也就是调用任务的 run 方法，把 run 方法当作和普通方法一样的地位去调用，相当于把每个任务的 run() 方法串联了起来，所以线程数量并不增加


超时退出机制 
线程池状态的维护
```



### 线程池的拒绝策略场景

```
1.AbortPolicy中止策略：丢弃任务并抛出RejectedExecutionException异常。


这是线程池默认的拒绝策略，在任务不能再提交的时候，抛出异常，及时反馈程序运行状态。如果是比较关键的业务，推荐使用此拒绝策略，这样子在系统不能承载更大的并发量的时候，能够及时的通过异常发现。


功能：当触发拒绝策略时，直接抛出拒绝执行的异常，中止策略的意思也就是打断当前执行流程.
使用场景：这个就没有特殊的场景了，但是有一点要正确处理抛出的异常。ThreadPoolExecutor中默认的策略就是AbortPolicy，ExecutorService接口的系列ThreadPoolExecutor因为都没有显示的设置拒绝策略，所以默认的都是这个。但是请注意，ExecutorService中的线程池实例队列都是无界的，也就是说把内存撑爆了都不会触发拒绝策略。当自己自定义线程池实例时，使用这个策略一定要处理好触发策略时抛的异常，因为他会打断当前的执行流程。


2.DiscardPolicy丢弃策略：ThreadPoolExecutor.DiscardPolicy：丢弃任务，但是不抛出异常。如果线程队列已满，则后续提交的任务都会被丢弃，且是静默丢弃。


使用此策略，可能会使我们无法发现系统的异常状态。建议是一些无关紧要的业务采用此策略。例如，本人的博客网站统计阅读量就是采用的这种拒绝策略。


功能：直接静悄悄的丢弃这个任务，不触发任何动作。
使用场景：如果你提交的任务无关紧要，你就可以使用它 。因为它就是个空实现，会悄无声息的吞噬你的的任务。所以这个策略基本上不用了。


3.DiscardOldestPolicy弃老策略：丢弃队列最前面的任务，然后重新提交被拒绝的任务。
此拒绝策略，是一种喜新厌旧的拒绝策略。是否要采用此种拒绝策略，还得根据实际业务是否允许丢弃老任务来认真衡量。


功能：如果线程池未关闭，就弹出队列头部的元素，然后尝试执行
使用场景：这个策略还是会丢弃任务，丢弃时也是毫无声息，但是特点是丢弃的是老的未执行的任务，而且是待执行优先级较高的任务。基于这个特性，想到的场景就是，发布消息和修改消息，当消息发布出去后，还未执行，此时更新的消息又来了，这个时候未执行的消息的版本比现在提交的消息版本要低就可以被丢弃了。因为队列中还有可能存在消息版本更低的消息会排队执行，所以在真正处理消息的时候一定要做好消息的版本比较。


4.CallerRunsPolicy调用者运行策略：由调用线程处理该任务。
功能：当触发拒绝策略时，只要线程池没有关闭，就由提交任务的当前线程处理。
使用场景：一般在不允许失败的、对性能要求不高、并发量较小的场景下使用，因为线程池一般情况下不会关闭，也就是提交的任务一定会被运行，但是由于是调用者线程自己执行的，当多次提交任务时，就会阻塞后续任务执行，性能和效率自然就慢了。

```



### 线程同步

一个服务启动的过程 a（异步） b（异步） c（同步）， c需要等待a b执行完成，如何实现

```
CompletableFuture


考察候选人是否了解 CountDownLatch的使用场景

```



### AQS使用场景

```
ReentrantLock、Semaphore、CountDownLatch等
```



### AQS基本原理

```
AQS AbstractQueuedSynchronizer 抽象队列同步器


内部有一个FIFO队列，如果某个线程获取同步状态失败了，AQS会把这个线程构造成一个node，
放入FIFO队列的尾部，如果同步状态释放的时候，会从FIFO队列的头部唤醒一个node。
线程获取一个共享资源的时候，自动进行排队。


AQS里有一个FIFO队列，用来排队 ，和一个volatile修饰的state的，代表了某个共享资源的同步状态，或者锁的状态


获取一把锁的时候，尝试去获取AQS里的state状态


刚开始state的值是0（没有人获取这把锁），线程1是可以获取到锁，然后会将state设置为1， 
线程2来尝试获取state的值，是此时state = 1，被别人锁了，线程2的信息构造成一个node放入FIFO队列中来排队，同时阻塞住线程2
 
线程1释放锁的时候，会将state设置为0，此时会唤醒FIFO队列中的排在队头的线程2，
线程2发现state = 0，就会获取锁，将state设置为1，同时线程2的node从FIFO队列里面出队。

```



### CAS工作原理

```
工作原理


CAS，Compare and Swap，比较和交换， 实现了乐观锁的思想


CAS会操作3个数字，当前内存中的值，旧的预期值，新的修改值，只有当旧的预期值跟内存中的值一样的时候，才会将内存中的值修改为新的修改值。
eg： int a = 1，内存中的当前值，CAS（1, 5），第一个是旧的预期值，如果1和a是一样的，就将a修改为5。
 
CAS存在的问题


1 ABA问题：如果某个值开始是A，后来变成了B，然后又变成了A，期望的是值如果是第一个A才会设置新值，跟第二个A比较也一样，设置了新值，跟期望是不符合。
AtomicStampedReference类，就是会比较两个值的引用是否一致，如果一致，才会设置新值
 
2 无限循环问题：Atomic类设置值的时候会进入一个无限循环，只要不成功，就不断循环再次尝试，高并发下，多线程频繁修改，可能会导致这个compareAndSet()里要循环N次才设置成功
 
3 多变量原子问题：一般的AtomicInteger，只能保证一个变量的原子性，多个变量可以用AtomicReference，可以封装自定义对象，多个变量可以放一个自定义对象里，会检查这个对象的引用是不是一个。
 
```



### 锁升级过程

```
当线程1访问代码块并获取锁对象时，会在java对象头和栈帧中记录偏向的锁的threadID，因为偏向锁不会主动释放锁，因此以后线程1再次获取锁的时候，需要比较当前线程的threadID和Java对象头中的threadID是否一致，如果一致（还是线程1获取锁对象），则无需使用CAS来加锁、解锁；如果不一致（其他线程，如线程2要竞争锁对象，而偏向锁不会主动释放因此还是存储的线程1的threadID），那么需要查看Java对象头中记录的线程1是否存活，如果没有存活，那么锁对象被重置为无锁状态，其它线程（线程2）可以竞争将其设置为偏向锁；如果存活，那么立刻查找该线程（线程1）的栈帧信息，如果还是需要继续持有这个锁对象，那么暂停当前线程1，撤销偏向锁，升级为轻量级锁，如果线程1 不再使用该锁对象，那么将锁对象状态设为无锁状态，重新偏向新的线程 ，线程1获取轻量级锁时会先把锁对象的对象头MarkWord复制一份到线程1的栈帧中创建的用于存储锁记录的空间（称为DisplacedMarkWord），然后使用CAS把对象头中的内容替换为线程1存储的锁记录（DisplacedMarkWord）的地址；  如果在线程1复制对象头的同时（在线程1CAS之前），线程2也准备获取锁，复制了对象头到线程2的锁记录空间中，但是在线程2CAS的时候，发现线程1已经把对象头换了，线程2的CAS失败，那么线程2就尝试使用自旋锁来等待线程1释放锁。 自旋锁简单来说就是让线程2在循环中不断CAS  但是如果自旋的时间太长也不行，因为自旋是要消耗CPU的，因此自旋的次数是有限制的，比如10次或者100次，如果自旋次数到了线程1还没有释放锁，或者线程1还在执行，线程2还在自旋等待，这时又有一个线程3过来竞争这个锁对象，那么这个时候轻量级锁就会膨胀为重量级锁。重量级锁把除了拥有锁的线程都阻塞，防止CPU空转 
```



### syncronized和ReentrantLock区别

```
synchronized 的实现涉及到锁的升级，具体为无锁、偏向锁、自旋锁、向系统申请重量级锁；ReentrantLock实现则是通过利用CAS自旋机制保证线程操作的原子性和volatile保证数据可见性以实现锁的功能。

场景 需求 举例 todo @刘欢

Sync 不需要手动释放，lock 需要unlock；
Sync 不能中断，lock 可以设置trylock 超时中断，也可用interrupt中断；
Sync 只有非公平锁，lock默认非公平锁，可以配置公平锁；
Sync 不能绑定条件，lock 可以绑定condition；
Sync锁的是对象，锁是保存在对象头里面的，根据对象头数据来标识是否有线程获得锁/争抢锁；lock锁的是线程，根据进入的线程和int类型的state标识锁的获得/争抢。
```

