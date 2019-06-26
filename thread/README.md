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

              虚拟机位数 | 头对象结构 |  说明  
              -         |-          |   -
              32/64bit  | Mark Word |默认存储对象的hashCode，分代年龄，锁类型，锁标志位等信息  |
              32/64bit  | Class Metadata Address | 类型指针指向对象的类元数据，JVM通过这个指针确定该对象是哪个类的数据 |
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
