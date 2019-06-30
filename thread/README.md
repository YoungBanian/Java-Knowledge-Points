# Java知识点记录

## 1. 进程和线程的区别
  - 进程是资源分配的最小单位，线程是CPU调度的最小单位
    - 所有与进程相关的资源，都被记录在PCB中
    - 进程是抢占处理机的调度单位，而线程属于某个进程,并且线程之间可以共享资源(也就是说进程可以包括多个线程)
        - 线程只由堆栈寄存器、程序计数器和TCB组成
  - 总结:
    1. 线程不能看作独立应用，而进程可看作独立应用
    2. 进程有独立的地址空间，相互不影响，线程只是进程的不同执行路径
    3. 线程没有独立的地址空间，多进程的程序比多线程程序健壮
    4. 进程的切换比线程的切换开销大

## 2. Java进程和线程的关系
  - Java对操作系统提供的功能进行封装，包括进程和线程
  - 运行一个程序会产生一个进程，进程包括至少一个线程
  - 每个进程对应一个JVM实例，多个线程共享JVM里的堆
  - Java采用单线程编程模型，程序会自动创建主线程
  - 主线程可以创建子线程，原则上要后于子线程完成执行

![进程](https://github.com/YoungBanian/Resources/blob/master/jc.png?raw=true)
![线程](https://github.com/YoungBanian/Resources/blob/master/thread.png?raw=true)

## 3. Thread中的start和run方法的区别
  - 调用start()方法会创建一个新的子线程并启动
  - run()方法只是Thread的一个普通方法的调用
  - 参考jdk源码:
    - `执行流程: Thread#start() -> JVM_StartThread -> thread_entry ->Thread#run()`
    - http://hg.openjdk.java.net/jdk8u/jdk8u/jdk/file/6bd23f3a0296/src/share/native/java/lang/Thread.c
    - http://hg.openjdk.java.net/jdk8u/jdk8u/hotspot/file/75f4e02f1113/src/share/vm/prims/jvm.cpp (native_thread = new JavaThread(&thread_entry, sz);)

## 4. Thread和Runnable是什么关系
  - Thread是实现了Runnable接口的类，使得run支持多线程
  - 因类的单一继承原则，推荐多使用Runable接口

## 5. 如何给run()方法传参:
  1. 构造函数传参
  2. 成员变量传参
  3. 回调函数传参

## 6. 如何实现处理线程的返回值
  1. 主线程等待法(例如循环sleep阻塞主线程等待子线程完成并返回值，缺点臃肿且根据返回值循环判断不够灵活,同时sleep时间不够精准)
  2. 使用Thread类的join()阻塞当前线程以等待子线程处理完毕(比①更精准，缺点粒度不够细)
  3. 通过Callable接口实现：通过FutureTask Or 线程池获取

## 7. 线程的六个状态
  1. 新建(New): 创建后尚未启动的线程的状态
  2. 运行(Runnable): 包含Running和Ready
  3. 无限期等待(Waiting): 不会被分配CPU执行时间,需要显示被唤醒
      - 没有设置Timeout参数的Object.wait()方法
      - 没有设置Timeout参数的Thread.join()方法
      - LockSupport.park()方法
  4. 限期等待(Timed Waiting): 在一定时间后会由系统自动唤醒
      - Thread.sleep()方法
      - 设置了Timeout参数的Object.wait()方法
      - 设置了Timeout参数的Thread.join()方法
      - LockSupport.parkNanos()方法
      - LockSupport.parkUntil()方法
  5. 阻塞(Blocked): 等待获取排它锁
  6. 结束(Terminated): 已终止线程的状态，线程已经结束执行

## 8. sleep和wait的区别
  - 基本的差别:
      1. sleep是Thread类的方法，wait是Object类中定义的方法
      2. sleep()方法可以在任何地方使用
      3. wait()方法只能在synchronized方法或synchronized块中使用
  - 最主要的本质区别
      1. Thread.sleep只会让出CPU，不会导致锁行为的改变
      2. Object.wait不仅让出CPU，还会释放已经占有的同步资源锁

## 9. notify和notifyAll的区别
  - 扩展知识：
    1. 锁池EntryList
        - 假设线程A已经拥有了某个对象(不是类)的锁，而其他线程B、C想要调用这个对象的某个synchronized方法(或者块)，由于B、C线程在进入对象的synchronized方法(或者块)，由于B、C线程在进入对象的synchronized方法(或者块)之前必须先获得该对象锁的拥有权，而恰巧该对象的锁目前正在被线程A所占用，此时B、C线程就会被阻塞，进入一个地方去等待锁的释放，这个地方便是该对象的锁池
    2. 等待池WaitSet
        - 假设线程A调用了某个对象的wait()方法，线程A就会释放该对象的锁，同时线程A就进入到了该对象的等待池中，进入到等待池中的线程不会去竞争该对象的锁.
  - 区别：
    1. notifyAll会让所有处于等待池的线程全部进入锁池去竞争获取锁的机会
    2. notify只会`随机`选取一个处于等待池中的线程进入锁池去竞争获取锁的机会

## 10. yield函数
  - 概念:
      - 当调用Thread.yield()函数时，会给线程调度器一个当前线程愿意让出CPU使用的暗示，但是线程调度器可能会忽略这个暗示
      - 注意对锁是没有任何影响的

## 11. synchronized
  - ### 线程安全问题的主要诱因
    - 存在共享数据（也称临界资源）
    - 存在多条线程共同操作这些共享数据
  - ### 解决问题根本方法：
    - 同一时刻有且只有一个线程在操作共享数据，其他线程必须等到该线程处理完数据后再对共享数据进行操作
  - ### 互斥锁的特性
    - 互斥性：即在同一时间只允许一个线程持有某个对象锁，通过这种特性来实现多线程的协调机制，这样在同一时间只有一个线程对需要同步的代码块(复合操作)进行访问。互斥性也称为操作的原子性。
    - 可见性：必须确保在锁被释放之前，对共享变量所做的修改，对于随后获得该锁的另一个线程是可见的（即在获得锁时应获得最新共享变量的值），否则另一个线可能是在本地缓存的某个副本上继续操作，从而引起不一致。
  - ### 根据获取的锁的分类： 获取对象锁和获取类锁
    - 获取对象锁的两种用法：
      1. 同步代码块(synchronized(this)，synchronized(类实例对象))，锁是小括号()中的实例对象
      2. 同步非静态方法（synchronized method），锁是当前对象的实例对象
    - 获取类锁的两种用法
      1. 同步代码块(synchronized(类.class))，锁是小括号()中的类对象(Class对象)
      2. 同步静态方法(synchronized static method)，锁是当前对象的类对象（Class对象）
  - ### 对象锁和类锁的总结
      1. 有线程访问对象的同步代码块时，另外的线程可以访问该对象的非同步代码块
      2. 若锁住的是同一个对象，一个线程在访问对象的同步代码块时，另外一个访问对象的同步代码块的线程会被阻塞
      3. 若锁住的是同一个对象，一个线程在访问对象的同步方法时，另一个访问对象同步方法的线程会被阻塞
      4. 若锁住的是同一个对象，一个线程在访问对象的同步代码块时，另一个访问对象同步方法的线程会被阻塞，反之亦然
      5. 同一个类的不同对象的对象锁互不干扰
      6. 类锁由于也是一种特殊的对象锁，因此表现和上述1，2，3，4一致，而由于一个类只有一把对象锁，所以同一个类的不同对象使用类锁将会是同步的
      7. 类锁和对象锁互不干扰
  - ### synchronized底层实现原理
    - 实现synchronized(Java对象头、实例数据、对齐填充)的基础
      1. Java对象头
          - 对象在内存中的布局
            - 对象头:
              <table>
                <tr>
                  <td>虚拟机位数</td>
                  <td>头对象结构</td>
                  <td>说明</td>
                </tr>
                <tr>
                  <td>32/64bit</td>
                  <td>Mark Word</td>
                  <td>默认存储对象的hashCode，分代年龄，锁类型，锁标志位等信息</td>
                </tr>
                <tr>
                  <td>32/64bit</td>
                  <td>Class Metadata Address</td>
                  <td>类型指针指向对象的类元数据，JVM通过这个指针确定该对象是哪个类的数据</td>
                </tr>
              </table>
            - Mark Word
              <table>
                <tr>
                  <td rowspan="2">锁状态</td>
                  <td colspan="2" align="center">25bit</td>
                  <td rowspan="2">4bit</td>
                  <td align="center">1bit</td>
                  <td align="center">2bit</td>
                </tr>
                <tr>
                  <td>23bit</td>
                  <td>2bit</td>
                  <td>是否是偏向锁</td>
                  <td>锁标志位</td>
                </tr>
                <tr>
                  <td>无锁状态</td>
                  <td colspan="4" align="center">对象hashcode、对象分代年龄</td>
                  <td align="center">01</td>
                </tr>
                <tr>
                  <td>轻量级锁</td>
                  <td colspan="4" align="center">指向锁记录的指针</td>
                  <td align="center">00</td>
                </tr>
                <tr>
                  <td>重量级锁</td>
                  <td colspan="4" align="center">指向重量级锁的指针</td>
                  <td align="center">10</td>
                </tr>
                <tr>
                  <td>GC标记</td>
                  <td colspan="4" align="center">空，不需要记录信息</td>
                  <td align="center">11</td>
                </tr>
                <tr>
                  <td>偏向锁</td>
                  <td align="center">线程ID</td>
                  <td align="center">Epoch</td>
                  <td align="center">对象分代年龄</td>
                  <td align="center">1</td>
                  <td align="center">01</td>
                </tr>
              </table>
      2. Monitor: 每个Java对象天生自带了一把看不见的锁
          - Monitor锁的竞争、获取与释放
            - ![Monitor竞争锁](https://github.com/YoungBanian/Resources/blob/master/monitor.png?raw=true)
      3. 什么是重入
          - 从互斥锁的设计上来说，当一个线程试图操作一个由其他线程持有的对象锁的临界资源时，将会处于阻塞状态，但当一个线程再次请求自己持有对象锁的临界资源时，这种情况属于重入(例如synchronized代码块里再次调用synchronized代码块)
      4. 自旋锁
          - 许多情况下，共享数据的锁定状态持续时间较短，切换线程不值得
          - 通过让线程执行忙循环等待锁的释放，不让出CPU
          - 缺点：若锁被其他线程长时间占用，会带来许多性能上的开销
      5. 自适应自旋锁
          - 自旋的次数不再固定
          - 由前一次在同一个锁上的自旋时间及锁的拥有者的状态来决定
      6. 锁消除(例如StringBuilder)
        - 更彻底的优化
          - JIT编译时，对运行上下文进行扫描，去除不可能存在竞争的锁
      7. synchronized的四种状态
          - 锁的内存语义
            - 当线程释放锁时,Java内存模型会把该线程对应的本地内存中的共享变量刷新到主内存中
            - 而当线程获取锁时，Java内存模型会把该线程对应的本地内存置为无效，从而使得被监视器保护的临界区代码必须从主内存中读取共享变量
            - ![锁获取与释放执行过程](https://github.com/YoungBanian/Resources/blob/master/lock.png?raw=true)
          - 无锁、偏向锁、轻量级锁、重量级锁
          - 锁膨胀方向：无锁 -> 偏向锁 -> 轻量级锁 -> 重量级锁
            - 偏向锁： 减少同一线程获取锁的代价(CAS[Compare And Swap])
              - 大多数情况下，锁不存在多线程竞争，总是由同一线程多次获得
              - 核心思想：
                - 如果一个线程获得了锁，那么锁就进入偏向模式，此时Mark Word的结构也变为偏向锁结构，当该线程再次请求锁时，无需再做任何同步操作，即获取锁的过程只需要检查Mark Word的锁标记位为偏向锁以及当前线程Id等于Mark Word的ThreadID即可，这样就省去了大量有关锁申请的操作。
                - 不适用于锁竞争比较激烈的多线程场合
            - 轻量级锁
              - 轻量级锁是由偏向锁升级来的，偏向锁运行一个线程进入同步块的情况下，当第二个线程加入锁争用的时候，偏向锁就会升级为轻量级锁。
              - 适应的场景：线程交替执行同步块
              - 若存在同一时间访问同一锁的情况，就会导致轻量级锁膨胀为重量级锁
          - 偏向锁、轻量级锁、重量级锁的汇总
            <table>
              <tr>
                <td>锁</td>
                <td>优点</td>
                <td>缺点</td>
                <td>使用场景</td>
              </tr>
              <tr>
                <td>偏向锁</td>
                <td>加锁和解锁不需要CAS操作，没有额外的性能消耗，和执行非同步方法相比仅存在纳秒的差距</td>
                <td>如果线程存在锁竞争，会带来额外的锁撤销的消耗</td>
                <td>只有一个线程访问同步块或者同步方法的场景</td>
              </tr>
              <tr>
                <td>轻量级锁</td>
                <td>竞争的线程不会阻塞，提高了响应速度</td>
                <td>若线程长时间抢不到锁，自旋会消耗CPU性能</td>
                <td>线程交替执行同步块或者同步方法的场景</td>
              </tr>
              <tr>
                <td>重量级锁</td>
                <td>线程竞争不使用自旋，不会消耗CPU</td>
                <td>线程阻塞，响应时间缓慢，在多线程下，频繁的获取释放锁，会带来巨大的性能消耗</td>
                <td>追求吞吐量，同步块或者同步方法执行时间较长的场景</td>
              </tr>
            </table>

## 12. synchronized和ReentrantLock的区别
  - ### ReentrantLock(再入锁)
    - 位于java.util.concurrent.locks包
    - 和CountDownLatch、FutureTask、Semaphore一样基于AQS实现
    - 能够实现比synchronized更细粒度的控制，如控制fairness
    - 调用lock()之后，必须调用unlock()释放锁
    - 性能未必比synchronized高，并且也是可重入的
    - ReentrantLock公平性的设置
      - ReentrantLock fairLock = new ReentrantLock(true);
      - 参数为true时，倾向于将锁富裕等待时间最久的线程
      - 公平锁: 获取锁的顺序按先后调用lock方法的顺序(慎用)
      - 非公平锁: 抢占的顺序不一定，看运气
      - synchronized是非公平锁
    - ReentrantLock将锁对象化
      - 判断是否有线程，或者某个特定线程，在排队等待获取锁
      - 带超时的获取锁的尝试
      - 感知有没有成功获取锁
    - 总结:
      - synchonized是关键字，ReentrantLock是类
      - ReentrantLock可以怼获取锁的等待时间进行设置，避免死锁
      - ReentrantLock可以获取各种锁的信息
      - ReentrantLock可以灵活地实现多路通知
      - 机制: sync操作Mark Word, lock调用Unsafe类的park()方法

## 13. Java内存模型JMM
  - Java内存模型(即Java Memory Model, 简称JMM)本身是一种抽象的概念，并不真实存在，它描述的是一组规则或规范，通过这组规范定义了程序中各个变量(包括实例字段，静态字段和构成数组的对象元素)的访问方式
    - ![JMM](https://github.com/YoungBanian/Resources/blob/master/jmm.png?raw=true)
  - JMM中的主内存和工作内存
    - JMM中的主内存
      - 存储Java实例对象
      - 包括成员变量、类信息、常量、静态变量等
      - 属于数据共享的区域，多线程并发操作时会引发线程安全问题
    - JMM中的工作内存
      - 存储当前方法的所有本地变量信息，本地变量对其他线程不可见
      - 字节码行号指示器、Native方法信息
      - 属于线程私有数据区域，不存在线程安全问题
    - JMM与Java内存区域划分是不同的概念层次
      - JMM描述的是一组规则，围绕原子性，有序性、可见性展开
      - 相似点：存在共享区域和私有区域
    - 主内存与工作内存的数据存储类型以及操作方式归纳
      - 方法里的基本数据类型本地变量将直接存储在工作内存的栈帧结构中
      - 引用类型的本地变量：引用存储在工作内存中，实例存储在主内存中
      - 成员变量、static变量、类信息均会被存储在主内存中
      - 主内存共享的方式是线程各拷贝一份数据到工作内存，操作完成后刷新回主内存
    - JMM解决可见性问题
      - JMM在多线程出现数据不一致性是由于读取数据阶段,如图:
        - ![JMM可见性分析](https://github.com/YoungBanian/Resources/blob/master/jmm_see.png?raw=true)
      - 指令重排序需要满足的条件:
        - 在单线程环境下不能改变程序运行的结果
        - 存在数据依赖关系的不允许重排序
        - 以上两个归结于：无法通过happens-before原则推导出来的，才能进行指令的重排序
          - happens-before的八大原则
            1. 程序次序规则: 一个线程内，按照代码顺序，书写在前面的操作先行于书写在后面的操作
            2. 锁定规则: 一个unLock操作先行发生于后面对同一个锁的lock操作
            3. volatile变量规则: 对一个变量的写操作先行发生于后面对这个变量的读操作
            4. 传递原则: 如果操作A先行发生于操作B，而操作B又先行发生于操作C，则可以得出操作A先行发生于操作C
            5. 线程中断规则: Thread对象的start()方法先行发生于此线程的每一个动作
            6. 线程中断规则: 对线程interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件的发生
            7. 线程终结规则: 线程中所有的操作都先行发生于线程的终止检测，我们可以通过Thread.join()方法结束、Thread.isAlive()的返回值手段检测到线程已经终止执行
            8. 对象终结规则: 一个对象的初始化完成先行发生于他的finalize()方法的开始
      - volatitle的可见性(当写一个volatitle变量时，JMM会把该线程对应的工作内存中的共享变量值刷新到主内存中;当读取一个volatititle变量时，JMM会把该线程对应的工作内存置为无效)
        - 保证被volatitle修饰的共享变量对所有线程总是可见的
        - 禁止指令重排序优化
      - volatitle如何禁止重排优化
        - 内存屏障(Memory Barrier)(通过插入内存屏障指令禁止在内存屏障前后的指令执行重排序优化,强制刷出各种CPU的缓存数据，因此任何CPU上的线程都能读取到这些数据的最新版本)
            - 保证特定操作的执行顺序
            - 保证某些变量的内存可见性
      - volatitle和synchronized的区别
        1. volatitle本质是在告诉JVM当前变量在寄存器（工作内存）中的值是不确定的，需要从主存中读取；synchronized则是锁定当前变量，只有当前线程可以访问该变量，其他线程被阻塞住直到该线程完成变量操作为止
        2. volatitle仅能使用在变量级别；synchronized则可以使用在变量、方法和类级别
        3. volatitle仅能实现变量的修改可见性，不能保证原子性；而synchronized则可以保证变量修改的可见性和原子性
        4. volatitle不会造成线程的阻塞；synchronized可能会造成线程阻塞
        5. volatitle标记的变量不会被编译器优化；synchronized标记的变量可以被编译器优化