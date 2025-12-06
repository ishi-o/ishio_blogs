---
title: "Java: 多线程编程"
date: 2025-05-17
categories: [Programming, Java, Java SE]
tags: [Java, multithreading]
mathjax: true
---
<!-- placeholder -->
<!-- more -->
## `Java`多线程编程

### `Runnable`接口

- 所有类，如果希望让它单独作为一个线程运行，可以**继承**`Thread`类并重写`void run()`方法
- 所有类，如果希望让它单独作为一个线程运行，更常用的方法是**实现**`Runnable`**接口**以便用多态特性被调用
- `Runnable`接口要求实现`void run()`方法，当这个类作为线程运行时，运行的就是这个`run()`方法
  使用这种方法，实现了`Runnable`接口的类称为线程任务对象，传递给`Thread`运行，创建的`Thread`对象称为线程对象

### `Thread`类

- `Thread`的常用构造器：
  - `Thread()`：创建空线程
  - `Thread(Runnable task, String name)`：将实现了`Runnable`接口的方法引用传递给它，并命名为`name`
- `java`线程包括七个状态：
  - `New`：新建的线程，尚未执行
  - `Runnable`：正在执行`run()`的线程
  - `Blocked`：因阻塞而被挂起的线程
  - `Waiting`：因某些原因在等待的线程
  - `Timed Waiting`：因主动调用`Thread.sleep()`而计时等待的线程
  - `Terminated`：终止的线程，`run()`因各种原因而结束
- `start()`：启动一个线程，使其开始以另一个线程的形式执行`run()`
  在主线程直接调用某个线程类的`run()`无法达到多线程的效果，必须通过`start()`在`JVM`中登记
- `join()`：使当前线程等待调用该方法的线程实例结束，再继续运行
  `join(long)`：使当前线程仅等待有限时间，其它同上
- `interrupt()`：使该线程的中断标志位置`1`，可以通过`isInterrupted()`循环检查，实现中断线程的效果
  但`interrupt()`并不立刻生效，仅仅是发出一个中断请求
  而且当外部线程调用该线程的`interrupt()`方法时，若该线程处于等待状态(例如调用`join()`或`sleep()`)，则`join()`或`sleep()`等会抛出`InterruptedException`异常
- `volatile`关键字：由于`JVM`的内存模型，在线程修改共享变量时不会立刻写回主内存
  而`volatile`修饰的变量则会使`JVM`在读取时总是读取最新值、写入时总是立刻写入
  但`volatile`并不保证原子性，在读写含有多个字段的`volatile`变量时可能会有问题
- `setDaemon(true)`：一个线程默认是非守护线程，`JVM`进程会等待所有的非守护线程结束后再结束
  但一些线程是无限循环的，可以调用`setDaemon(true)`将它们设置为守护线程，`JVM`进程结束时不会关心它们是否结束
  守护线程不能占有任何需要显式关闭的资源，守护线程本身无法保证这些资源能在`JVM`进程结束时关闭

### 传统线程同步

- 不同线程在读写同一份资源、或需要相互协作时，就需要考虑线程同步问题
  除非资源是只读的，例如不可变类型，则不需要考虑线程同步
  大部分标准库中的类为了提高性能，都是非线程安全的，涉及到非线程安全的读写操作时，必须手动添加线程同步代码以保证线程安全
- `synchronized`关键字：
  - 作用于对象时，会对该对象加锁
  - 修饰实例方法时，等价于作用于`this`
  - 修饰静态方法时，等价于作用于所在类的`class`实例
  - 一般不用该关键字修饰方法，因为会导致加锁混乱、不明确且在很多情况会使两个本不冲突的方法变为冲突
  使用`synchronized`的代码块无法并发执行，且加锁解锁有额外开销
  对某对象加锁，不代表其它线程就无法访问该对象，如果一个线程对该对象加锁而另一个线程并不这样做，则仍存在线程同步问题
- `JVM`的基础原子操作：除`long`、`double`以外的任意变量的赋值操作
  涉及多行的赋值操作时，仍需要用`synchronized`修饰代码块
- 可重入锁：`JVM`允许**同一个线程**重复获取同一个锁，其本质是一个信号量，进入/退出`synchronized`代码块使信号量加/减`1`
- 死锁的必要条件：互斥、不可抢占、占有并等待、循环等待，只要破坏其中一种条件即可避免死锁
  - 破坏循环等待：所有线程获取同一组锁的顺序保持一致，这是最简单的一种方案
  - 互斥不可破坏，破坏不可抢占可能导致混乱，破坏占有并等待必须一次性分配需要的资源、资源利用率低且可能导致饥饿
- 继承自`Object`的`wait()`：可使该线程暂时放弃调用`wait()`的对象锁，进入等待状态直至被唤醒，唤醒后立刻尝试重新请求这个对象锁
  需要注意这不是作用于线程类对象的，而是**作用于被加锁的资源**
- 继承自`Object`的`notify()`和`notifyAll()`：可随机唤醒某一个等待该资源的线程/唤醒全部等待该资源的线程
  同上，这两者作用于被加锁的资源，通常后者更安全
  - `notify()`可能会唤醒同类线程，导致活锁或死锁，例如生产者消费者问题：
    两个消费者阻塞$\rightarrow$生产者`P1`唤醒消费者`C1`$\rightarrow$两个生产者阻塞$\rightarrow$消费者`C1`消耗资源，但唤醒同类消费者`C2`$\rightarrow$两个消费者阻塞，至此所有线程均阻塞，造成死锁
  - 使用`notifyAll()`，至少能唤醒一个非同类线程，而其它同类线程应该继续等待，所以**`wait()`应该在循环**里而非`if`语句块中
- `JVM`线程同步原理是对象监视器，`Object`的`wait()`和`notify()`相当于每个变量都可以作为信号量的封装，由于**`wait()`使线程可以短暂释放已获得的锁**，使其不需要像操作系统课程上讲的那般麻烦(需要互斥信号量与同步信号量且互斥信号量的`PV`操作紧贴临界区)，而是使同步信号量围绕着互斥资源通过`wait()`和`notify()`进行同步
  例如在生产者消费者问题中，互斥资源的空/满可以化作`while`中的条件，充当同步信号量

  ```java
  // 一个线程同步的生产者消费者队列类
  private Queue<Integer> q;
  private int size;
  private static final MAX_SIZE = 10;
  // Consumer
  synchronized (this) {
      while (size == 0) { // Condition: Empty
          wait();
      }
      e = q.poll(); // consume
      --size;
      notifyAll();
  }
  // Provider
  synchronized (this) {
      while (size == MAX_SIZE) { // Condition: Full
          wait();
      }
      q.offer(e); // provide
      ++size;
      notifyAll();
  }
  ```

### `java.util.concurrent.locks`与`Semaphore`

- `synchronized`的加锁机制是**悲观锁**、**重量级锁**、**阻塞的**，这种锁适合竞争激烈的多线程同步场景，线程先必须获得锁(进入或退出`Monitor`)才能读写对象
- `java.util.concurrent`是`Java 5`开始提供的高级并发包，以下是需要提前了解的一些概念
  - 乐观锁：线程无需获取锁地尝试修改，在修改时检查是否冲突
    理念是估计在读过程中不会有其它线程在写
    乐观读锁在读多写少的场景中好处显而易见：一是乐观读锁不排它，减少写锁饥饿的情况；二是因为写少，乐观读大概率成功而减少了悲观锁的阻塞开销
  - `CAS`(`Compare And Swap`或`Compare And Set`)机制是实现乐观锁的一种方式：线程查询内存中的值和此前读取的值是否相同，若相同则更新，否则失败
    若失败则读取内存中的值，循环地进行`CAS`直至更新成功
  - `CAS`机制存在`ABA`问题，即线程`T1`尝试通过`CAS`读写时，虽然内存值和此前读取的值一致，但这个内存值`A`可能被另一个线程先改为`B`再改为`A`，在`T1`看来没有改变过的资源实际上被其它线程更改过
    因此实现上对资源的更改会添加时间戳/版本号
  - `CAS`机制有时会配合自旋的机制：
    **自旋锁**优点在于可避免不必要的上下文切换开销，缺点在于循环导致的`CPU`忙等
    自旋锁仅在`CPU`**多核**的并行处理场景中，线程能在忙等中获取其它线程释放的资源时才有效
  接下来从完全悲观到完全乐观地介绍`java.util.concurrent`提供的线程同步机制
- `Lock`接口：可以替代`synchronized`，是一种显式锁，由代码层面而非语法层面实现加锁和解锁，其实现类是对`synchronized`的封装
  `Lock`是悲观锁
  - `lock()`：显式加锁
  - `unlock()`：需要在`finally`块中解锁
  - `tryLock()`和`tryLock(long, TimeUnit)`：`Lock`支持非阻塞获取锁或有限忙等地获取锁，前者仅尝试一次、后者在有限时间内循环尝试
    返回`true`表示获取成功
  - `newCondition()`：返回一个`Condition`对象，`Lock`支持多条件锁，与`synchronized`单锁相区别
  `ReentrantLock`是`Lock`的实现类之一，译为“可重入锁”
  - 支持公平锁，在构造时传递`true`即可
    公平锁即每次向等待队列加入新的线程时，它无法插队，保证每次获取锁的线程是队列中等待时间最长的线程
- `Condition`接口：表示某条件的等待队列，如上文传统的线程同步代码中，由于空/满这两种条件使不同类线程在同一等待队列中，因此会出现死锁现象而必须用`notifyAll()`来唤醒非同类线程
  而`Condition`和`Lock`允许在一个锁对象上分配多个不同条件的等待队列，能保证每次唤醒都能唤醒非同类线程
  - `await()`：使线程在该条件上短暂地释放锁，进入等待状态，被唤醒后重新尝试获取
  - `await(long, TimeUnit)`：可以在有限时间内地等待
  - `signal()`：唤醒在该条件上等待的某个线程

  ```java
  private Queue<Integer> q;
  private int size;
  private static int MAX_SIZE = 10;
  private Lock lock = new ReentrantLock();
  private final Condition notempty = lock.newCondition(), notfull = lock.newCondition();
  // Consumer
  lock.lock();
  try {
      while (size == 0) {
          notempty.await();
      }
      e = q.poll();
      --size;
      notfull.signal();
  } finally {
      lock.unlock();
  }
  // Provider
  lock.lock();
  try {
      while (size == MAX_SIZE) {
          notfull.await();
      }
      q.offer(e);
      ++size;
      notempty.signal();
  } finally {
      lock.unlock();
  }
  ```

- `ReadWriteLock`接口：有时排它锁过于重量，在多读少写的场景下，希望允许多个线程同时读，这个接口就用于这种场景
  - `readLock()`：获取读锁
  - `writeLock()`：获取读锁
  类的内部会维护，若有线程占有读锁，则不会有线程能占有写锁，反之亦然；若没有线程占有写锁，则允许多个线程占有读锁
  虽然它实现了读写分离，但它仍是**悲观锁**，因此也可能导致需要**写锁的线程饥饿**
  `ReentrantReadWriteLock`是`ReadWriteLock`的实现类之一，是可重入锁
- `AQS`框架：
- `StampedLock`类：`Java 8`开始提供的**乐观读锁**和**读写分离的悲观锁**两者的封装锁，在互斥上可替代`ReadWriteLock`，但要注意它不是可重入锁、不支持公平锁
  - `StampedLock`和`ReadWriteLock`一样有`readLock()`和`writeLock()`方法，用于获取读锁和写锁
    但它们的实现有所区别：它们不返回`Lock`对象而是在内部就调用了锁的`lock()`方法，然后返回`long`类型的版本号`stamp`
  - 因此需要使用`unlockRead(stamp)`和`unlockWrite(stamp)`释放掉这个版本号的读写锁
  - `StampedLock`还支持**乐观读锁**，通过`tryOptimisticRead()`尝试获取乐观读锁的版本号`stamp`，然后通过`validate(stamp)`检验，**乐观锁不需要解锁，因为本质上并没有加锁操作**
    如前文所说，乐观锁不排它，乐观读过程中允许其它线程写，如果检验成功，说明加乐观读锁途中没有线程占有死锁
    如果检验不成功则有两种选择：自旋地重复乐观读、悲观读
  - 同`Lock`一样支持非阻塞或有限阻塞地获取悲观读锁和悲观写锁
  - 支持锁转换：
    `tryConvertToReadLock(stamp)`：若`stamp`有效，原子地，悲观读锁则返回其本身、悲观写锁则释放它并返回悲观读锁、乐观读锁则非阻塞地请求一个悲观读锁
    `tryConvertToWriteLock(stamp)`：若`stamp`有效，原子地，悲观读锁且写锁空闲则返回写锁、悲观写锁则返回其本身、乐观读锁则非阻塞地请求一个悲观写锁
    `tryConvertToOptimisticRead(stamp)`：若`stamp`有效，原子地，悲观锁则释放它们并返回乐观读锁、有效的乐观锁则返回其本身
  - 但`StampedLock`不支持基于条件的线程间协作，因此只适用于单个资源的互斥读写场景
- `Semaphore`：信号量可以被最多`N`个线程获取，但不支持基于条件的线程间协作，用于复数个资源的互斥获取，它也支持公平锁
  - `acquire()`：阻塞地获取该信号量
  - `release()`：需要在`finally`块中释放
  - `tryAcquire()`和`tryAcquire(long, TimeUnit)`：非阻塞地或有限阻塞地获取信号量

### `Callable<T>`接口与异步线程池

- `Callable<T>`接口：由于`Runnable`的`run()`没有返回值且不允许抛出异常，因此`Callable<T>`诞生了，是单方法接口，包含方法`T call() throws Exception`
- `java`线程池提供了**多线程异步**的功能，能更好地利用多核资源
  所谓同步，即各个任务有一定的执行顺序，一些任务必须等待其它任务完成后，利用其计算结果才能继续运行；异步是各个任务没有执行顺序，任务通过回调函数或其它手段获取其它任务的计算结果，在这段时间内可以执行其它计算
  同步异步与线程个数没有必然联系，单线程异步可以通过检测事件循环实现
- `Future<T>`接口是实现多线程异步的核心，表示“能在未来得到计算结果的对象”，包括以下核心方法：
  - `get()`：获取结果，如果其对应的计算任务尚未完成则会使调用`get()`的线程进入阻塞
  - `get(long, TimeUnit)`：仅等待有限时间地获取结果
  - `cancel(true)`：`true`表示通过发出中断请求来取消任务，若任务尚未开始或线程响应中断请求后则成功取消，返回`true`，否则返回`false`
    `cancel(false)`：若任务尚未完成则一定能取消成功，但任务可以继续执行直至结束
    成功调用`cancel()`后，`isCancelled()`返回`true`，此后`get()`会抛出`CancellationException`异常
- `ExecutorService`接口：
  - 任务提交：把任务提交给线程池，异步地执行
    `Future<T> submit(Callable<T> task)`：提交一个计算任务给线程池执行，**立刻**获得一个`Future<T>`对象
  - 任务执行：把多个任务提交给线程池，当前线程阻塞，即同步地执行
    `List<Future<T>> invokeAll(List<Callable<T>>)`：执行多个计算任务，当前线程阻塞直至它们全部执行完成，返回值顺序和提交的任务顺序一致
    `invokeAll()`还允许传递时间参数，有限地阻塞
  - `shutdown()`：停止接受所有新线程，执行完已提交的线程，然后关闭线程池
  - `shutdownNow()`：停止接受所有新线程，尝试中断地取消所有已提交线程，返回所有未开始的任务
- 创建`ExecutorService`对象：
  - `Executors`
  - `ThreadPoolExecutor`
- `Fork/Join`线程池：

### 其它`API`

- `java.util.concurrent.atomic`中的类基于`CAS`机制实现无锁的线程同步，使用**乐观锁**、**非阻塞**地封装资源
  例如`AtomicReferencte<T>、AtomicReferenceArray<T>`，除此之外还对三个基本数据类型有更全面的封装：`AtomicInteger、AtomicLong、AtomicBoolean`
  如果`ABA`问题会影响业务逻辑，则应使用`AtomicStampedReference`或`AtomicMarkableReference`
  以下是`AtomicReference`一系列常用的**原子方法**：
  - `get()`：获取当前值
  - `set(T)`：`volatile`地写入新值
  - `compareAndSet(T ept, T upd)`：`CAS`地更新值，可能失败
  - `getAndSet(T)`：获取旧值，写入新值
  - `getAndUpdate()`或`updateAndGet()`：提供单元运算函数更新值，返回更新前/更新后的值
  - `getAndAccumulate()`或`accumulateAndGet()`：提供二元运算函数和加值，返回更新前/更新后的值
- 线程安全的集合：
  - `List`的实现类`CopyOnWriteArrayList`
  - `Map`的实现类`ConcurrentHashMap`
  - `Set`的实现类`CopyOnWriteArraySet`
  - `Queue`的实现类`ArrayBlockingQueue`与`LinkedBlockingQueue`
  - `Deque`的实现类`LinkedBlockingDeque`
- `CompletableFuture`
- `ThreadLocal`
- 虚拟线程：
