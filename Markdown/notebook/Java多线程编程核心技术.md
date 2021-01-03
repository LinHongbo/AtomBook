# Java多线程编程核心技术

## 第一章 Java多线程技能
### 1.1 进程和多线程的概念及线程的优点
### 1.2 使用多线程
#### 1.2.1 继承Thread类
#### 1.2.2 实现Runnable接口
#### 1.2.3 实例变量与线程安全
#### 1.2.4 留意i--与System.out.println()的异常
### 1.3 currentThread()方法
### 1.4 isAlive()方法
&emsp;&emsp;线程处于正在运行或准备开始运行的状态，就认为线程是存活的。
### 1.5 sleep()方法
### 1.6 getId()方法
### 1.7 停止线程
* 使用退出标志，使线程正常退出，也就是当run方法完成后线程终止。
* 使用stop()方法强行终止线程，但是不推荐使用这个方法，因为stop和suspend以及resume一样，都是过期作废的方法，使用它们可能产生不可预料的结果。
* 使用interrupt()方法中断线程。
#### 1.7.1 停止不了的线程
#### 1.7.2 判断线程是否是停止状态
* Thread.interrupted();
* thread.isInterrupted();
#### 1.7.3 能停止的线程----异常法
#### 1.7.4 在沉睡中停止
* 在sleep(),再interrupt(),thread.isInterrupted() = false,且线程继续跑
* 先interrupt(),在sleep(),thread.isInterrupted() = true,线程中断
#### 1.7.5 能停止的线程----暴力停止
* 释放了锁
* 无法回收资源
#### 1.7.6 方法stop()与java.lang.ThreadDeath异常
#### 1.7.7 释放锁的不良后果
* 数据不同步
#### 1.7.8 使用return停止线程
### 1.8 暂停线程
#### 1.8.1 suspend()与resume()方法的使用
#### 1.8.2 suspend()与resume()方法的缺点----独占
&emsp;&emsp;我的理解是阻塞的，与锁之类的操作组合，那么会导致其他对象拿不到锁。
#### 1.8.3 suspend()与resume()方法的缺点----不同步
### 1.9 yield方法
&emsp;&emsp;放弃当前线程CPU资源。
### 1.10 线程的优先级
&emsp;&emsp;有点傻逼的章节，简单讲就是，设置了优先级，不一定按照优先级的顺序分配资源，但总体具备统计规律。
#### 1.10.1 线程优先级的继承特性
#### 1.10.2 优先级具有规则性
#### 1.10.3 优先级具有随机性
#### 1.10.4 看谁运行得快
### 1.11 守护线程
&emsp;&emsp;用户线程不存在，守护线程会自动销毁。

## 第二章 对象及变量的并发访问
### 2.1 synchronized同步方法
#### 2.1.1 方法内的变量为线程安全
#### 2.1.2 实例变量非线程安全
#### 2.1.3 多个对象多个锁
&emsp;&emsp;强调的是锁的对象。如果锁的对象不一致，那么还是异步操作。
#### 2.1.4 synchronized方法与锁对象
#### 2.1.5 脏读
#### 2.1.6 synchronized锁重入
&emsp;&emsp;关键字synchronized拥有锁重入功能，也就是在使用synchronized时，当一个线程得到一个对象锁后，再次请求此对象锁时是可以再次得到该对象的锁的。这也证明了在一个synchronized方法/块的内部调用奔雷的其他synchronized方法/块时，是永远可以得到锁的。
#### 2.1.7 出现异常，锁自动释放
#### 2.1.8 同步不具有继承性
&emsp;&emsp;举了个例子，子类**重写**了父类某个同步的方法，但是子类在该方法上并没有进行同步操作，那么子类该方法就不是线程安全的，但如果调用了super.x(),则父类该方法仍然是线程安全的。
### 2.2 synchronized同步语句块
#### 2.2.1 synchronized方法的弊端
#### 2.2.2 synchronized同步代码块的使用
#### 2.2.3 用同步代码块解决同步方法的弊端
&emsp;&emsp;垃圾篇章。其实还是因为同步代码块的范围与同步方法锁住的范围相比，同步代码块比较小，具备更高灵活性。
#### 2.2.4 一半异步，一半同步
#### 2.2.5 synchronized代码块间的同步性
#### 2.2.6 验证同步synchronized（this）代码块是锁定当前对象的
#### 2.2.7 将任意对象作为对象检测器
> 这个例子可以看下
#### 2.2.8 细化验证3个结论
> synchronized(非this对象x)格式的写法是将x对象本身作为一个“对象监视器”，这样就可以得出以下3个结论。
>* 当多个线程同时执行synchronized(x){}同步代码块时呈同步效果
>* 当其他线程执行x对象中synchronized同步方法时呈同步效果
>* 当其他线程执行x对象方法中的synchronized(this)代码块时也呈现同步效果
#### 2.2.9 静态同步synchronized方法与synchronized(class)代码块
#### 2.2.10 数据类型String的常量池特性
#### 2.2.11 同步synchronized方法无线等待与解决
#### 2.2.12 多线程的死锁
#### 2.2.13 内置类与静态内置类
#### 2.2.14 内置类与同步：实验1
#### 2.2.15 内置类与同步：实验2
#### 2.2.16 锁对象的改变
> 跟2.2.10小结类似
### 2.3 volatile关键字
#### 2.3.1 关键字volatile与死循环
> 该小节内容与标题没有一毛钱关系，垃圾。
#### 2.3.2 解决同步死循环
> 该小节内容同样跟标题没有奶子关系，垃圾。
#### 2.3.3 解决异步死循环
&emsp;&emsp;使用volatile关键字增加了实例变量在多个线程之间的可见性，但volatile关键字最致命的缺点就是不支持原子性。
下面将关键字synchronized和volatile进行比较
* 关键字volatile是线程同步的轻量级实现，所以volatile性能肯定比synchronized要好，并且volatile只能修饰变量，而synchronized可以修饰方法，以及代码块。  
* 多线程访问volatile变量不会发生阻塞，而synchronized会出现阻塞。
* volatile能保证数据可见性，但不能保证原子性；而synchronized可以保证原子性，也可以间接保证可见性，因为它会将私有内存和公共内存的数据做同步
> 线程安全包括原子性和可见性两个方面(**可能还有有序性**)
#### 2.3.4 volatile非原子特性
#### 2.3.5 使用原子类进行i++操作
#### 2.3.6 原子类也不完全安全
> 指外层没作同步处理的话，那么也会出现非线程安全的情况。

## 第三章 线程间通信
### 3.1 等待/通知机制
#### 3.1.1 不使用等待/通知机制实现进程间的通信
> 轮询
#### 3.1.2 什么是等待/通知机制
#### 3.1.3 等待/通知机制的实现
> 线程的状态  

> &emsp;&emsp;新建一个新的线程对象后，再调用其start()方法，系统会为此线程分配CPU资源，使其处于Runnable状态，这是一个准备运行的状态。如果线程抢占到CPU资源，此线程就处于Running状态。  

> &emsp;&emsp;Runnable状态和Running状态是可相互切换的，因为有可能线程运行一段时间后，有其他高优先级的线程抢占了CPU资源，此线程就会从Running状态切换到Runnable状态。  
&emsp;&emsp;线程进入Runnable状态大体分为如下几种情况：  
>* 调用sleep()方法后经过的时间超过指定的睡眠时间
>* 线程调用阻塞IO已经返回，阻塞方法执行完毕
>* 线程成功地获得了试图同步的监视器
>* 线程正在等待某个通知，其他线程发出了通知
>* 处于挂起状态的线程调用了resume()恢复方法  

> &emsp;&emsp;Blocked是阻塞的意思，例如遇到了一个IO操作，此时CPU处于空闲状态，可能会转而把CPU时间片分配给其他线程，这时也可以成为“暂停”状态。Blocked状态结束后，会进入Runnable状态，等待系统重新分配资源。
&emsp;&emsp;出现阻塞的情况大体分为如下几种情况：
>* 线程调用sleep()，主动放弃占用的CPU资源
>* 线程调用了阻塞式IO方法，在该方法返回前，该线程被阻塞
>* 线程试图获得一个同步监视器，但该同步监视器正在被其他线程持有
>* 线程等待某个通知
>* 程序调用了suspend()将该线程挂起

> 每个锁对象都有两个队列，一个是就绪队列，一个是阻塞队列。就绪队列存储了将要获得锁的线程，阻塞队列存储了被阻塞的线程。一个线程被唤醒后，才会进入就绪队列，等待CPU的调度；反之，一个线程被wait()后，就会进入阻塞队列，等待下一次被唤醒。
#### 3.1.4 方法wait()释放锁与notify()不释放锁
* sleep()也不释放锁
#### 3.1.5 当interrupt()遇到wait()
#### 3.1.6 只通知一个线程
#### 3.1.7 唤醒所有线程
#### 3.1.8 方法wait(long)的使用
#### 3.1.9 通知过早
#### 3.1.10 等待wait()的条件发生变化
> 暂时觉得这个例子不错，可以一看。
#### 3.1.11 生产者/消费者模式的实现
> 1、一生产与一消费：操作值  
> 2、多生产与多消费：操作值---假死  
&emsp;&emsp;在代码中确实已经通过wait/notify()进行通信了，但不保证notify唤醒的是异类，也许是同类，比如生产者唤醒生产者，或消费者唤醒消费者这种情况。（**这个例子不错**）  
> 3、多生产与多消费：操作值  
&emsp;&emsp;notifyAll()  
> 4、一生产与一消费：操作栈  
> 5、一生产与多消费：操作栈（解决wait条件改变与假死）  
> 6、多生产与一消费：操作栈  
> 7、多生产与多消费：操作栈
#### 3.1.12 通过管道进行线程间的通信：字节流
#### 3.1.13 通过管道进行线程间的通信：字符流
#### 3.1.14 实战：等待/通知之交叉备份
### 3.2 方法join()的使用
&emsp;&emsp;等待线程对象销毁
#### 3.2.1 学习join()前的铺垫
&emsp;&emsp;方法join()具有使线程排队运行的作用，有些类似同步的运行效果。join()与synchronized的区别是：join()内部使用了wait()方法进行等待，而synchronized关键字使用的是“对象监视器”原理作为同步。
#### 3.2.2 用join()方法来解决
#### 3.2.3 方法join()与异常
#### 3.2.4 方法join(long)的使用
#### 3.2.5 方法join(long)与sleep(long)的区别
&emsp;&emsp;方法join()的核心是wait，所以看wait()与sleep()区别即可
#### 3.2.6 方法join()后面的代码提前运行：出现意外
#### 3.2.7 方法join()后面的代码提前运行：解释意外
### 3.3 类ThreadLocal的使用
#### 3.3.1 方法get()与null
#### 3.3.2 验证线程变量的隔离性
#### 3.3.3 解决get()返回null问题
#### 3.3.4 再次验证线程变量的隔离性
### 3.4 类InheritableThreadLocal的使用
#### 3.4.1 值继承
&emsp;&emsp;使用类InheritableThreadLocal可以再子线程中取得父线程继承下来的值。
#### 3.4.2 继承再修改


## 第四章 Lock的使用
> 垃圾章节，都在讲API
### 4.1 使用ReentrantLock类
&emsp;&emsp;在Java多线程中，可以使用synchronized关键字来实现线程之间同步互斥，但在JDK 1.5中心增加了ReentrantLock类也能达到同样的效果，并且在拓展功能上也更加强大，比如具有嗅探锁定、多路分支通知等功能，而且在使用上也更加灵活。
#### 4.1.1 使用ReentrantLock实现同步:测试1
#### 4.1.2 使用ReentrantLock实现同步:测试2
#### 4.1.3 使用Condition实现等待等待/通知:错误用法与解决
#### 4.1.4 正确使用Condition实现等待/通知
#### 4.1.5 使用多个Condition实现通知部分线程：错误用法
#### 4.1.6 使用多个Condition实现通知部分线程：正确用法
#### 4.1.7 实现生产者/消费者模式：一对一交替打印
#### 4.1.8 实现生产者/消费者模式：多对多交替打印
&emsp;&emsp;没看懂
#### 4.1.9 公平锁与非公平锁
#### 4.1.10 方法getHoldCount()、getQueueLength()和getWaitQueueLength()的测试
#### 4.1.11 方法hasQueuedThread(Thread)、hasQueuedThreads()和hasWaiters(Cindition)的测试
#### 4.1.12 方法isFair()、isHeldByCurrentThread()和isLocked()的测试
#### 4.1.13 方法lockInterruptibly()、tryLock()和tryLock(long, TimeUnit)的测试
#### 4.1.14 方法awaitUninterrupted()的使用
#### 4.1.15 方法awaitUntil()的使用
#### 4.1.16 使用Condition实现顺序执行
### 4.2 使用ReentrantReadWriteLock类
#### 4.2.1 类ReentrantReadWriteLock的使用：读读共享
#### 4.2.2 类ReentrantReadWriteLock的使用：写写互斥
#### 4.2.3 类ReentrantReadWriteLock的使用：读写互斥
#### 4.2.4 类ReentrantReadWriteLock的使用：写读互斥

## 第五章 定时器Timer
> 主要讲API，需要注意一点，启动一个timer,需要及时调用cancel()，否则会造成该线程一直在跑。
## 第六章 单例模式与多线程
### 6.1 立即加载/饿汉模式
### 6.2 延迟加载/懒汉模式
### 6.3 使用静态内置类实现单例模式
### 6.4 序列化和反序列化的单例模式的实现
* 使用readResolve()方法解决
### 6.5 使用static代码块实现单例模式
### 6.6 使用枚举数据类型实现单例模式
### 6.7 完善使用枚举实现单例模式
> 总结：  
> 6.1、6.3、6.5：实际上是一样的，因为都不能带参数。  
> 6.6、6.7：使用enum，实际上利用了枚举类会自动调用构造方法的特性

## 第七章 拾遗补漏
### 7.1 线程的状态
> android线程有6种状态，分别是  
> Thread.State.NEW:至今未启动的线程处于该状态  
> Thread.State.RUNNABLE:正在Java虚拟机种执行的线程处于该状态  
> Thread.State.BLOCKED:受阻塞并等待某个监听器锁的线程处于该状态  
> Thread.State.WAITING:无限期等待另一个线程处于这种状态  
> Thread.State.TIMED_WAITTING:等待另一个线程来执行取决于制定等待时间的操作的线程处于该状态  
> Thread.State.TERMINATED:已退出的线程处于该状态  
![image](https://raw.githubusercontent.com/LinHongbo/pic/master/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E7%BC%96%E7%A8%8B%E6%A0%B8%E5%BF%83%E6%8A%80%E6%9C%AF/Java%E7%BA%BF%E7%A8%8B%E7%8A%B6%E6%80%81%E6%9C%BA.png)

> ***todo:wait跟sleep的区别***  
> 1. sleep()是阻塞的，wait()是非阻塞的；
> 2. sleep()不会释放锁，wait()放弃持有的对象锁
### 7.2 线程组
> 1. 线程组自动归属特性：新实例化的线程/线程组，如果不指定所属线程组，那么该线程/线程组会自动添加到当前线程所属的线程组  
> 2. 组内的线程可批量停止  
> 3. 可以递归/非递归取得组内对象  
> * Thread.currentThread().getThreadGroup().enumerate(new Array[], true/false)  
### 7.3 使线程具有有序性(*)
> 1. 同步；
> 2. 序数化；
>3. wait()/notify()
### 7.4 SimpleDateFormat非线程安全
> 解决方式：
> 1. 每个线程使用不同的SimpleDateFormat对象。
> 2. 使用ThreadLocal类处理
### 7.5 线程中出现异常的处理
> 1. thread.setUncaughtExceptionHandler();
>2. Thread.setDefaultUncaughtExceptionHandler();
### 7.6 线程组内的异常处理
> 本小结旨在线程组内如果有某一线程出现异常，那么直接停止掉所有线程。
>* 重写线程组的uncaughtException()方法。
>* 需要注意：如果线程内有捕捉异常，那么线程组的uncaughtException()方法不执行。
### 7.7 线程异常处理的传递
> 线程对象的异常处理器->线程组对象的异常处理器->线程类的全局处理器
